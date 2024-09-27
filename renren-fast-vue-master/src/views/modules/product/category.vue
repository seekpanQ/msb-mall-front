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
  </div>
</template>
<script>
/* eslint-disable */
export default {
  data() {
    return {
      data: [],
      expandKeys: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
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
      console.log("添加", data);
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