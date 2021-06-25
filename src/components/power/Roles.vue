<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRolesDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">
                  {{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!--
            <pre
              >{{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showRolesEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoles(scope.row.id)"
              >删除</el-button
            >
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>

      <!-- 添加角色的对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addRolesDialogVisible"
        @close="addRolesDialogClosed"
      >
        <!-- 主体区域 -->
        <el-form
          :model="addRolesForm"
          :rules="addRolesRules"
          ref="addRolesFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addRolesDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRoles">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 编辑角色的对话框 -->
      <el-dialog
        title="修改角色"
        :visible.sync="editRolesDialogVisible"
        @close="editRolesDialogClosed"
      >
        <!-- 主体区域 -->
        <el-form
          :model="editRolesForm"
          :rules="editRolesRules"
          ref="editRolesFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="editRolesDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRolesInfo">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 分配权限的对话框 -->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <!-- 树形控件 -->
        <el-tree :data="rightslist" :props="treeProps" show-checkbox
        node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef">
        </el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights">确 定</el-button>
        </span>
      </el-dialog>


    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表
      rolelist: [],
      //控制添加角色对话框的显示与隐藏
      addRolesDialogVisible: false,
      //添加角色表单的数据
      addRolesForm: {
        roleName: "",
        roleDesc: "",
      },
      //添加角色表单验证规则对象
      addRolesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          {
            min: 2,
            max: 10,
            message: "角色名称长度在 2 到 10 个字符之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: false, message: "请输入角色描述", trigger: "blur" },
          {
            min: 4,
            max: 15,
            message: "角色描述长度在 4 到 15 个字符之间",
            trigger: "blur",
          },
        ],
      },
      //控制编辑角色对话框的显示与隐藏
      editRolesDialogVisible: false,
      //查询到的角色表单的数据
      editRolesForm: {},
      //编辑角色表单的验证规则对象
      editRolesRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          {
            min: 2,
            max: 10,
            message: "角色名称长度在 2 到 10 个字符之间",
            trigger: "blur",
          },
        ],
        roleDesc: [
          { required: false, message: "请输入角色描述", trigger: "blur" },
          {
            min: 4,
            max: 15,
            message: "角色描述长度在 4 到 15 个字符之间",
            trigger: "blur",
          },
        ],
      },
      // 控制分配对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: ''
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    //获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败！");
      }
      this.rolelist = res.data;
    },
    //监听添加角色对话框的关闭事件
    addRolesDialogClosed() {
      this.$refs.addRolesFormRef.resetFields();
    },
    //点击按钮，添加新角色
    addRoles() {
      this.$refs.addRolesFormRef.validate(async (valid) => {
        if (!valid) return;
        //发起添加角色的网络请求
        const { data: res } = await this.$http.post("roles", this.addRolesForm);
        if (res.meta.status !== 201) {
          this.$message.error("添加角色失败！");
        }
        this.$message.success("添加角色成功！");
        //隐藏添加角色对话框
        this.addRolesDialogVisible = false;
        //重新获取角色列表
        this.getRolesList();
      });
    },
    //展示编辑角色对话框
    async showRolesEditDialog(id) {
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询角色信息失败！");
      }
      this.editRolesForm = res.data;
      this.editRolesDialogVisible = true;
    },
    //监听编辑角色对话框关闭事件
    editRolesDialogClosed() {
      this.$refs.editRolesFormRef.resetFields();
    },
    //修改角色信息并提交
    editRolesInfo() {
      this.$refs.editRolesFormRef.validate(async (valid) => {
        if (!valid) return;
        //发起修改角色信息的请求
        const { data: res } = await this.$http.put(
          "roles/" + this.editRolesForm.roleId,
          {
            roleName: this.editRolesForm.roleName,
            roleDesc: this.editRolesForm.roleDesc,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("更新角色信息失败！");
        }
        //关闭编辑角色对话框
        this.editRolesDialogVisible = false;
        //重新渲染角色列表
        this.getRolesList();
        //提示修改角色信息成功
        this.$message.success("更新角色信息成功！");
      });
    },
    //根据id删除对应的角色信息
    async removeRoles(id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该角色, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => {
        return err;
      });
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除！");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除角色信息失败！");
      }
      this.$message.success("删除角色信息成功！");
      this.getRolesList();
    },
    //根据id删除对应的权限
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除！");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除权限失败！");
      }
      //   this.getRolesList();
      //   this.$message.success('')
      role.children = res.data;
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限列表
      const {data: res} = await this.$http.get('rights/tree')
      if (res.meta.status !==200) return this.$message.error('获取权限数据失败！')
      // 获取到的权限数据保存到data中
      this.rightslist = res.data
      console.log(this.rightslist);

      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归形式，获取角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      });
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys=[]
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')

      const { data: res } =await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  },
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>