<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数" type="warning" :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <!-- <span>选择商品分类：</span> -->
          <!-- 级联选择框 -->
          <el-cascader
            v-model="selectKeys"
            :options="cateList"
            :props="cascaderProps"
            @change="handleCascaderChange"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabsClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addParamsVisible = true"
          >添加参数</el-button>
          <!-- 动态属性表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleTagClosed(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  id="el-input"
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addParamsVisible = true"
          >添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleTagClosed(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  id="el-input"
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加动态或静态属性的对话框 -->
    <el-dialog
      @close="handleAddParamsDialogClosed"
      :title="'添加'+titleText"
      :visible.sync="addParamsVisible"
      width="50%"
    >
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamsVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleAddParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      @close="handleEditParamsDialogClosed"
      :title="'修改'+titleText"
      :visible.sync="editParamsVisible"
      width="50%"
    >
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamsVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleEditParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      // 级联选择框中的配置属性
      cascaderProps: {
        expandTrigger: "hover",
        value: "cat_id",
        label: "cat_name",
        children: "children"
      },
      //   级联选择框中选择的属性
      selectKeys: [],
      activeName: "many",
      //动态参数
      manyTableData: [],
      //静态参数
      onlyTableData: [],
      addParamsVisible: false,
      //添加动态或静态参数表单
      addForm: {
        attt_name: ""
      },
      addFormRules: {
        attr_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      },
      editParamsVisible: false,
      editFormRules: {
        attr_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 5 个字符", trigger: "blur" }
        ]
      },
      editForm: {}
    };
  },
  created() {
    this.getCateList();
  },
  computed: {
    isBtnDisabled: function() {
      console.log(this.selectKeys.length);
      if (this.selectKeys.length !== 0) {
        return false;
      }
      return true;
    },
    titleText: function() {
      if (this.activeName === "many") {
        return "动态参数";
      } else {
        return "静态属性";
      }
    },
    cateId: function() {
      return this.selectKeys.length !== 0
        ? this.selectKeys[this.selectKeys.length - 1]
        : 0;
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$messsage.error("获取分类数据失败");
      }
      this.cateList = res.data;
      console.log(this.cateList);
    },
    handleCascaderChange() {
      this.getParamsData();
    },
    handleTabsClick() {
      this.getParamsData();
      console.log(this.activeName);
    },
    //根据静态参数或动态参数获取数据
    async getParamsData() {
      if (this.selectKeys.length == 0) {
        this.manyTableData = [];
        this.onlyTableData = [];
        return;
      }
      const id = this.cateId;
      const { data: res } = await this.$http.get(
        `categories/${id}/attributes`,
        {
          params: {
            sel: this.activeName
          }
        }
      );
      if (res.meta.status !== 200) {
        return this.$messsage.error("获取参数列表失败");
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
        //为每一个元素添加各自的tag标签属性，以防止互相干扰
        item.inputVisible = false;
        item.inputValue = "";
      });
      if (this.activeName === "many") {
        this.manyTableData = res.data;
      } else {
        this.onlyTableData = res.data;
      }
    },
    handleAddParamsDialogClosed() {
      this.$refs["addFormRef"].resetFields();
    },
    handleAddParams() {
      this.$refs["addFormRef"].validate(async valid => {
        if (!valid) {
          return this.$message.error("请确定您输入的信息合法");
        }
        const id = this.cateId;
        const { data: res } = await this.$http.post(
          `categories/${id}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加参数失败");
        }
        this.$message.success("添加参数成功");
        this.addParamsVisible = false;
        this.getParamsData();
      });
    },
    handleEditParamsDialogClosed() {
      this.$refs["editFormRef"].resetFields();
    },
    handleEditParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return;
        const {
          data: res
        } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        );

        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败！");
        }

        this.$message.success("修改参数成功！");
        this.getParamsData();
        this.editParamsVisible = false;
      });
    },
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        {
          params: { attr_sel: this.activeName }
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败！");
      }

      this.editForm = res.data;
      this.editParamsVisible = true;
    },
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该参数, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);

      // 用户取消了删除的操作
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除！");
      }

      // 删除的业务逻辑
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      );

      if (res.meta.status !== 200) {
        return this.$message.error("删除参数失败！");
      }

      this.$message.success("删除参数成功！");
      this.getParamsData();
    },
    //自定义标签处理函数
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputVisible = false;
        row.inputValue = "";
        return;
      }
      //将标签添加到数组中
      row.attr_vals.push(row.inputValue.trim());
      row.inputValue = "";
      row.inputVisible = false;
      this.addAttrParams(row);
    },
    async addAttrParams(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(" ")
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("添加或删除标签失败");
      }
      this.$message.success("添加或删除标签成功");
    },
    showInput(row) {
      row.inputVisible = true;
      //$nextTick方法会在页面被渲染之后进行回调
      //input框必须等到页面渲染完毕后才能获得
      this.$nextTick(_ => {
        document.getElementById("el-input").focus();
        // this.$refs['saveTagInput'].$refs['input'].focus();
      });
    },
    handleTagClosed(index, row) {
      row.attr_vals.splice(index, 1);
      this.addAttrParams(row);
    }
  }
};
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>