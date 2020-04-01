<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table
        :data="cateList"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        :columns="columns"
        class="treeTable"
      >
        <!-- 是否可删除 -->
        <template slot="isOk" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen;"
          ></i>
          <i class="el-icon-error" v-else style="color:red;"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            icon="el-icon-edit"
            @click="showEditDialog(scope.row)"
          >编辑</el-button>
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
            @click="deleteCateById(scope.row)"
          >删除</el-button>
        </template>
      </tree-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类 -->
    <el-dialog
      title="提示"
      @close="addCateDialogClosed"
      :visible.sync="addCateDialogVisible"
      width="50%"
    >
      <el-form
        ref="addCateFormRef"
        label-width="100px"
        :model="addCateForm"
        :rules="addCateFormRules"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="handleCascaderChange"
            clearable
          ></el-cascader>
          <!-- change-on-select -->
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类 -->
    <el-dialog
      title="修改分类信息"
      :visible.sync="modifyCateDialogVisible"
      width="50%"
      @close="closeEditCateDialog"
    >
      <el-form
        ref="modifyRoleFormRef"
        :rules="modifyCateRules"
        :model="modifyForm"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="modifyForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [
        { label: "分类名称", prop: "cat_name" },
        {
          label: "是否有效",
          type: "template",
          template: "isOk"
        },
        {
          label: "排序",
          type: "template",
          template: "order"
        },
        {
          label: "操作",
          type: "template",
          template: "opt"
        }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        //分类名称
        cat_name: "",
        //父级分类的id
        cat_pid: 0,
        //分类的等级
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ]
      },
      //获取父级分类
      parentCateList: [],
      //获取选中的key
      selectedKeys: [],
      //级联选择器的属性配置
      cascaderProps: {
        checkStrictly: true,
        expandTrigger: "hover",
        value: "cat_id",
        label: "cat_name",
        children: "children"
      },
      //编辑窗口显示
      modifyCateDialogVisible: false,
      modifyForm: {},
      modifyCateRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取分类列表失败");
      }
      this.total = res.data.total;
      this.cateList = res.data.result;
      console.log(this.cateList);
    },
    //分页处理页面大小改变
    handleSizeChange(pageSize) {
      this.queryInfo.pagesize = pageSize;
      this.getCateList();
    },
    //分页处理页面改变
    handleCurrentChange(currentPage) {
      this.queryInfo.pagenum = currentPage;
      this.getCateList();
    },

    //获取父级分类列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: {
          type: 2
        }
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类数据失败");
      }
      console.log(res.data);
      this.parentCateList = res.data;
    },
    //添加分类对话框
    addCateDialog() {
      this.getParentCateList();
      console.log(this.parentCateList);
      this.addCateDialogVisible = true;
    },
    handleCascaderChange() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ];
        this.addCateForm.cat_level = this.selectedKeys.length;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    //添加分类
    addCate() {
      console.log(this.addCateForm);
      this.$refs["addCateFormRef"].validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          "categories",
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加分类失败");
        }
        this.$message.success("添加分类成功");
        this.getCateList();
        this.addCateDialogVisible = false;
      });
    },
    addCateDialogClosed() {
      this.$refs["addCateFormRef"].resetFields();
      this.selectedKeys = [];
      this.addCateForm.cat_pid = 0;
      this.addCateForm.cat_level = 0;
    },
    async deleteCateById(cate) {
      const confirmRes = await this.$confirm(
        "此操作将永久删除该分类, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      console.log(cate);
      if (confirmRes !== "confirm") {
        return this.$message.info("删除分类操作已经取消");
      }
      const { data: res } = await this.$http.delete(
        `categories/${cate.cat_id}`
      );
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error("删除角色失败");
      }
      this.$message.success("删除分类成功");
      this.getCateList();
    },
    editCate() {
      this.$refs["modifyRoleFormRef"].validate(async valid => {
        if (!valid) {
          return this.$message.info("分类数据不符合要求");
        }
        const { data: res } = await this.$http.put(
          `categories/${this.modifyForm.cat_id}`,
          {
            cat_name: this.modifyForm.cat_name
          }
        );
        console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error("更新分类失败");
        }
        this.$message.success("分类更新成功");
        this.getCateList();
        this.modifyCateDialogVisible = false;
      });
    },
    async showEditDialog(cate) {
      const { data: res } = await this.$http.get("categories/" + cate.cat_id);
      if (res.meta.status !== 200) {
        return this.$message.error("获取分类信息失败");
      }
      this.modifyForm = res.data;
      this.modifyCateDialogVisible = true;
    },
    closeEditCateDialog() {
      this.$refs["modifyRoleFormRef"].resetFields();
      this.modifyForm = {};
    }
  }
};
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>