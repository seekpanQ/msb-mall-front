<template>
  <div>
    <el-tree
      :data="data"
      show-checkbox
      :props="defaultProps"
      :expand-on-click-node="false"
      node-key="catId"
      :default-expanded-keys="expandKeys"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="data.catLevel <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加
          </el-button>
          <el-button
            v-if="data.children.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <!--添加弹出框-->
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="categoryForm">
        <el-form-item label="类别名称" :label-width="formLabelWidth">
          <el-input v-model="categoryForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-input v-model="categoryForm.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input
            v-model="categoryForm.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialog">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
/* eslint-disable */
export default {
  data() {
    return {
      data: [],
      expandKeys: [],
      dialogVisible: false,
      defaultProps: {
        children: "children",
        label: "name",
      },
      categoryForm: {
        parentCid: 0,
        catLevel: 0,
        showStatus: 0,
        sort: 0,
        name: "",
        icon: "",
        productUnit: "",
      },
      formLabelWidth: "120px",
    };
  },
  methods: {
    getCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/listTree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取的类别数据：", data.data);
        this.data = data.data;
      });
    },
    append(data) {
      this.dialogVisible = true; // 打开弹出窗口
      console.log("添加", data);
      this.categoryForm.parentCid = data.catId; // 添加的类别对应的父菜单
      this.categoryForm.catLevel = data.catLevel + 1; // 设置添加类别的层级
      this.categoryForm.showStatus = 1; // 菜单的显示状态  1 显示  0 被删除
      this.categoryForm.sort = 0; // 排序 默认给 0
    },
    addDialog() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.categoryForm, false),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
          });
          this.dialogVisible = false; // 关闭弹出窗口
          // 重新加载所有的菜单数据
          this.getCategory();
          // 设置默认展开的父节点信息
          this.expandKeys = [this.categoryForm.parentCid];
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    remove(node, data) {
      this.$confirm(`是否确认删除【${data.name}】记录?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then((_) => {
          // 传递的数据
          let ids = [data.catId];
          // 把删除的请求提交到后台服务
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
              });
              // 重新加载所有的菜单数据
              this.getCategory();
              // 设置默认展开的父节点信息
              this.expandKeys = [node.parent.data.catId];
            } else {
              this.$message.error(data.msg);
            }
          });
        })
        .catch((_) => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
  created() {
    this.getCategory();
  },
};
</script>