<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="setAddRoleVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolesList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
              :class="['bdbottom',i1 === 0 ? 'bdtop':'','vcenter']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="deleteRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                  :class="[i2===0?'':'bdtop','vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="deleteRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <!-- 渲染三级权限 -->
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="deleteRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="getRoleById(scope.row)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="deleteRoleById(scope.row)"
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的树形对话框 -->
    <el-dialog
      @close="closeRightDialog"
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
    >
      <el-tree
        :data="rightsList"
        :props="treeProps"
        :default-checked-keys="defKeys"
        ref="treeRef"
        show-checkbox
        default-expand-all
        node-key="id"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="角色添加"
      @close="closeAddRoleDialog"
      :visible.sync="setAddRoleVisible"
      width="50%"
    >
      <el-form ref="addRoleRef" :rules="addRoleRules" :model="addRoleForm" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setAddRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 角色编辑对话框 -->
    <el-dialog
      title="修改角色信息"
      :visible.sync="modifyRoleDialogVisible"
      width="50%"
      @close="closeEditRoleDialog"
    >
      <el-form
        ref="modifyRoleFormRef"
        :rules="modifyRoleRules"
        :model="modifyRoleForm"
        label-width="80px"
      >
        <el-form-item label="角色姓名" prop="roleName">
          <el-input v-model="modifyRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="modifyRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rolesList: [],
      setRightDialogVisible: false,
      //角色添加对话框
      setAddRoleVisible: false,
      rightsList: [],
      treeProps: {
        children: "children",
        label: "authName"
      },
      //树形dialog的默认选中权限
      defKeys: [],
      roleId: "",
      addRoleRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      },
      //角色添加对象
      addRoleForm: {
        roleName: "",
        roleDesc: ""
      },
      modifyRoleForm: {},
      modifyRoleDialogVisible: false,
      modifyRoleRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
          { min: 3, max: 5, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      }
    };
  },
  created: function() {
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolesList = res.data;
      console.log(this.rolesList);
    },
    async deleteRightById(role, rightId) {
      const confirmRes = await this.$confirm(
        "此操作将永久删除该权限, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (confirmRes !== "confirm") {
        return this.$message.info("删除权限取消");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除权限失败");
      }
      role.children = res.data;
    },
    async showRightDialog(role) {
      this.roleId = role.id;
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200) {
        return this.$message.error("获取所有的权限失败");
      }
      this.rightsList = res.data;
      this.getLeafKeys(role);
      this.setRightDialogVisible = true;
    },
    getLeafKeys(node) {
      if (!node.children) {
        return this.defKeys.push(node.id);
      }
      node.children.forEach(element => {
        this.getLeafKeys(element);
      });
    },
    closeRightDialog() {
      this.defKeys = [];
    },
    async allotRights() {
      const keys = [
        ...this.$refs["treeRef"].getCheckedKeys(),
        ...this.$refs["treeRef"].getHalfCheckedKeys()
      ];
      const idStr = keys.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("分配权限失败");
      }
      this.$message.success("分配权限成功");
      this.getRolesList();
      this.setRightDialogVisible = false;
    },
    addRoles() {
      this.$refs["addRoleRef"].validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post("roles", this.addRoleForm);
        if (res.meta.status !== 201) {
          return this.$message.error("角色添加失败");
        }
        this.$message.success("角色添加成功");
        this.setAddRoleVisible = false;
        this.getRolesList();
      });
    },
    closeAddRoleDialog() {
      this.$refs["addRoleRef"].resetFields();
    },
    async deleteRoleById(role) {
      const confirmRes = await this.$confirm(
        "此操作将永久删除该角色, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (confirmRes !== "confirm") {
        return this.$message.info("删除角色操作已经取消");
      }
      const { data: res } = await this.$http.delete("roles/" + role.id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除角色失败");
      }
      this.$message.success("删除角色成功");
      this.getRolesList();
    },
    async getRoleById(role) {
      const { data: res } = await this.$http.get("roles/" + role.id);
      if (res.meta.status !== 200) {
        return this.$message.error("获取当前角色失败");
      }
      this.modifyRoleForm = res.data;
      this.modifyRoleDialogVisible = true;
    },
    closeEditRoleDialog() {
      this.$refs["modifyRoleFormRef"].resetFields();
    },
    editRole() {
      this.$refs["modifyRoleFormRef"].validate(async valid => {
        if (!valid) {
          return this.$message.error("角色存在不合法信息");
        }
        console.log(this.modifyRoleForm.id);
        const { data: res } = await this.$http.put(
          "roles/" + this.modifyRoleForm.roleId,
          {
            roleName: this.modifyRoleForm.roleName,
            roleDesc: this.modifyRoleForm.roleDesc
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("角色修改失败");
        }
        this.$message.success("角色修改成功");
        this.modifyRoleDialogVisible = false;
        this.getRolesList();
      });
    }
  }
};
</script>
<style lang="less" scope>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
</style>