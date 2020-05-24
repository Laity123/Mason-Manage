<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert show-icon title="注意:只允许为第三级分类设置相关参数!" type="info" :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>请选择商品分类:&nbsp;</span>
          <el-cascader
            v-model="selectedCatekeys"
            :options="cateList"
            :props="{ expandTrigger: 'hover',
            value:'cat_id',
            label:'cat_name',
            children:'children'
            }"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible=true"
          >添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染Tag标签 -->
                <el-tag
                  v-for="(item,index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="handleClose(index,scope.row)"
                >{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加的按钮 -->
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
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible=true"
          >添加属性</el-button>
          <!-- 静态属性表格 -->
          <el-table :data="onlyTableData" border>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染Tag标签 -->
                <el-tag
                  v-for="(item,index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="handleClose(index,scope.row)"
                >{{item}}</el-tag>
                <!-- 输入的文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <!-- 添加的按钮 -->
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
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 添加参数的表单 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数的对话框 -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 添加参数的表单 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      // 商品分类列表
      cateList: [],
      // 级联选择框的配置对象
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children"
      },
      // 级联选择框双向绑定到的数组
      selectedCatekeys: [],
      // 被激活的页签名称
      activeName: "many",
      // 动态参数的数据
      manyTableData: [],
      // 静态属性
      onlyTableData: [],
      // 控制添加参数对话框的显示与隐藏
      addDialogVisible: false,
      // 添加参数的表单数据
      addForm: {
        attr_name: ""
      },
      // 添加表单的验证规则
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: "请输入参数名称",
            trigger: "blur"
          }
        ]
      },
      // 控制修改参数对话框的显示与隐藏
      editDialogVisible: false,
      // 修改的表单对象
      editForm: {},
      // 修改表单的验证规则
      editFormRules: {
        attr_name: [
          {
            required: true,
            message: "请输入参数名称",
            trigger: "blur"
          }
        ]
      }
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败!");
      }
      this.cateList = res.data;
    },
    // 级联选择框选项变化触发函数
    handleChange() {
      this.getParamsData();
    },
    // Tab页签点击事件的处理函数
    handleTabClick() {
      this.getParamsData();
    },
    // 获取参数的列表数据
    async getParamsData() {
      // 选中的不是三级分类
      if (this.selectedCatekeys.length !== 3) {
        this.selectedCatekeys = [];
        this.manyTableData = [];
        this.onlyTableData = [];
        return;
      }
      // 选中的是三级分类
      // 根据所选分类的Id和当前所处的面板获取对应参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数列表失败!");
      }
      // 将attr-vals转换为数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals = item.attr_vals
          ? item.attr_vals.split(" ")
          : [];
        // 给每个数据设置独立的布尔值与value
        item.inputVisible = false;
        item.inputValue = "";
      });
      if (this.activeName === "many") {
        this.manyTableData = res.data;
      } else {
        this.onlyTableData = res.data;
      }
    },
    // 关闭对话框重置数据
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    // 点击确定按钮添加参数
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加参数失败!");
        }
        this.$message.success("添加参数成功!");
        this.getParamsData();
        this.addDialogVisible = false;
      });
    },
    // 点击按钮展示修改对话框
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        {
          params: {
            attr_sel: this.activeName
          }
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败!");
      }
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    // 关闭对话框重置数据
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },
    // 点击确定按钮添加参数
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败!");
        }
        this.$message.success("修改参数成功!");
        this.getParamsData();
        this.editDialogVisible = false;
      });
    },
    // 根据Id删除对应的参数项
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
      // 用户取消了删除操作
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }
      // 删除
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除参数失败!");
      }
      this.$message.success("删除参数成功!");
      this.getParamsData();
    },
    // 文本框失去焦点或按下回车键触发
    handleInputConfirm(row) {
      // 当内容为空
      if (row.inputValue.trim().length === 0) {
        row.inputValue = "";
        row.inputVisible = false;
        return;
      }
      // 有内容
      row.attr_vals.push(row.inputValue.trim());
      row.inputValue = "";
      row.inputVisible = true;
      // 将数据同步到数据库
      this.saveAttrVals(row);
    },
    // 点击按钮展示文本输入框
    showInput(row) {
      //   this.inputVisible = true;
      row.inputVisible = true;
      // 让文本框自动获取焦点 $nextTick:当页面重新渲染之后执行
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    // 删除 Tag
    handleClose(index, row) {
      row.attr_vals.splice(index, 1);
      this.saveAttrVals(row);
    },
    // 将数据同步到数据库
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(" ")
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("修改参数项失败!");
      }
      this.$message.success("修改参数项成功!");
    }
  },
  computed: {
    // 控制按钮是否禁用
    isBtnDisabled() {
      if (this.selectedCatekeys.length !== 3) {
        return true;
      }
      return false;
    },
    // 获取三级分类Id
    cateId() {
      if (this.selectedCatekeys.length === 3) {
        return this.selectedCatekeys[2];
      }
      return null;
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === "many") {
        return "动态参数";
      }
      return "静态属性";
    }
  }
};
</script>

<style lang='less' scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 8px;
}
.input-new-tag {
  width: 100px;
}
</style>
