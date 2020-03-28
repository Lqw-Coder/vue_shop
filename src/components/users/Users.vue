<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <!-- 搜索栏区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            class="input-with-select"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- table数据栏区域 -->
      <el-table :data="users" style="width: 100%" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="changeStatus(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="modifyDialog(scope.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteUserById(scope.row.id)"
            ></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top-start" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页数据栏 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totalNum"
      ></el-pagination>
    </el-card>
    <!-- 用户添加输入框 -->
    <el-dialog title="用户添加" @close="closeAddDialog" :visible.sync="addDialogVisible" width="50%">
      <el-form ref="addFormRef" :rules="addRules" :model="addForm" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 用户添加框 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addaddDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 用户修改对话框 -->
    <el-dialog
      title="修改用户信息"
      :visible.sync="modifyDialogVisible"
      width="50%"
      @close="closeModifyDialog"
    >
      <el-form ref="modifyFormRef" :rules="modifyRules" :model="modifyForm" label-width="80px">
        <el-form-item label="用户名">
          <el-input v-model="modifyForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="modifyForm.email" type="password"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="modifyForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    var checkMail = (rule, value, callback) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/;
      if (regEmail.test(value)) {
        return callback();
      }
      callback(new Error("请输入合法的邮箱"));
    };
    var checkMobile = (rule, value, callback) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/;

      if (regMobile.test(value)) {
        return callback();
      }

      callback(new Error("请输入合法的手机号"));
    };
    return {
      modifyDialogVisible: false,
      addDialogVisible: false,
      users: [],
      totalNum: 0,
      queryInfo: {
        query: "",
        pagenum: 1, //当前页数
        pagesize: 2 //页面数据数量
      },
      modifyForm: {},
      addForm: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      addRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ],
        email: [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            validator: checkMail,
            trigger: "blur"
          }
        ],
        mobile: [
          { required: true, message: "请输入手机号码", trigger: "blur" },
          { validator: checkMobile, trigger: "blur" }
        ]
      },
      modifyRules: {
        email: [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            validator: checkMail,
            trigger: "blur"
          }
        ],
        mobile: [
          { required: true, message: "请输入手机号码", trigger: "blur" },
          { validator: checkMobile, trigger: "blur" }
        ]
      }
    };
  },
  created: function() {
    this.getUserList();
  },
  methods: {
    //获取用户数据
    async getUserList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo
      });
      if (res.meta.status != 200) {
        this.$message.error("获取用户列表失败");
      } else {
        this.users = res.data.users;
        this.totalNum = res.data.total;
      }
    },
    handleSizeChange(newValue) {
      this.queryInfo.pagesize = newValue;
      this.getUserList();
    },
    handleCurrentChange(newValue) {
      this.queryInfo.pagenum = newValue;
      this.getUserList();
    },
    //修改用户状态
    async changeStatus(userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      );
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state;
        this.$message.error("更新用户状态失败");
      } else {
        this.$message.success("更新状态成功");
      }
    },
    //关闭添加输入框时清空校验数据
    closeAddDialog() {
      this.$refs["addFormRef"].resetFields();
    },
    //将输入框中的数据发送到后台
    addUser() {
      this.$refs["addFormRef"].validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post("users", this.addForm);
        if (res.meta.status !== 201) {
          return this.$message.error("用户添加失败");
        }
        this.$message.success("用户添加成功");
        this.addDialogVisible = false;
        this.getUserList();
      });
    },
    //获取编辑框中个人用户数据
    async modifyDialog(id) {
      const { data: res } = await this.$http.get("users/" + id);
      console.log(id);
      if (res.meta.status !== 200) {
        return this.$message.error("获取待修改用户失败");
      }
      this.modifyForm = res.data;
      this.modifyDialogVisible = true;
    },
    //关闭编辑框时清空校验数据
    closeModifyDialog() {
      this.$refs["modifyFormRef"].resetFields();
    },
    modifyUserInfo() {
      this.$refs["modifyFormRef"].validate(async valid => {
        if (!valid) {
          return this.$message.error("用户存在不合法数据");
        }
        const { data: res } = await this.$http.put(
          "users/" + this.modifyForm.id,
          {
            email: this.modifyForm.email,
            mobile: this.modifyForm.mobile
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("用户修改失败");
        }
        this.$message.success("用户修改成功");
        this.modifyDialogVisible = false;
        this.getUserList();
      });
    },
    async deleteUserById(id) {
      const confirmRes = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if(confirmRes !== 'confirm'){
        return this.$message.info('删除操作已取消')
      }
      const {data :res} = await this.$http.delete('users/'+id)
      if(res.meta.status !== 200){
        return this.$message.error('删除用户失败')
      }
      this.$message.success('用户删除成功')
      this.getUserList()
    }
  }
};
</script>
<style lang="less" scope>
</style>