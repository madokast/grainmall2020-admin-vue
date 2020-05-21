<template>
  <div>
    <el-tree
      class="menus-tree"
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      点击节点不会展开_只有点击左边的三角
      :expand-on-click-node="false"
      关键id_为了vue区分
      node-key="catId"
      要展开的key
      :default-expanded-keys="expandedKey"
    >
      <!-- 添加节点的增删功能，slot-scope 即 vue 的插槽，node 是每一个节点，data是全部数据-->
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ "["+ data.catId + "] "+ node.label }}</span>
        <span class="crud-node">
          <el-button type="text" size="mini" @click="() => edit(data)">Edit</el-button>
          <el-button v-if="node.level<=2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button
            v-if="node.childNodes===null||node.childNodes.length==0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 数据
      menus: [],
      // 要展开的key key 就是 catId
      expandedKey: [],
      // append 数据的对话框
      dialogFormVisible: false,
      defaultProps: {
        // menus 数组是一个对象数组，里面哪个属性指向子节点
        children: "children",
        // menus 数组是一个对象数组，要显示哪个属性的值到页面
        label: "name"
      }
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    // 获得所有菜单
    getMenus() {
      // 发送请求
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
        params: this.$http.adornParams({})
      }).then(({ data }) => {
        // 这里的大括号 {} 是对象解构
        // console.log(data.data);
        // 绑定
        this.menus = data.data;
      });
    },
    // 添加节点
    append(data) {
      const parentCid = data.catId;

      this.$prompt(`新增【${data.name}】分类的子分类`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(({ value }) => {
          this.$http({
            url: this.$http.adornUrl("/product/category/save"),
            method: "post",
            data: this.$http.adornData(
              {
                name: value,
                parentCid: data.catId,
                catLevel: data.catLevel + 1
              },
              false
            )
          })
            .then(({ data }) => {
              if (data && data.code === 0) {
                // 刷新
                this.getMenus();
                this.expandedKey = [parentCid];
                console.log(parentCid);

                this.$message({
                  message: "新增【${data.name}】分类的子分类【${value}】成功",
                  type: "success",
                  duration: 1500,
                  onClose: () => {
                    this.getDataList();
                  }
                });
              } else {
                this.message.error(data.msg);
              }
            })
            .catch(() => {
              this.$message({
                type: "info",
                message: "新增失败，请检查格式或重试"
              });
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消新增"
          });
        });
    },
    // 删除节点
    // 传入两个参数node和data，分别表示当前节点的 Node 对象和当前节点的数据
    remove(node, data) {
      const that = this;

      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          // 拿到要删除的节点id
          let ids = [data.catId];

          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            if (data && data.code === 0) {
              // 刷新
              this.getMenus();
              this.expandedKey = [node.parent.data.catId];

              this.$message({
                message: "删除成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.getDataList();
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    edit(data){
      const parentCid = data.parentCid;

      this.$prompt(`修改分类`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        inputValue: data.name
      })
        .then(({ value }) => {
          this.$http({
            url: this.$http.adornUrl("/product/category/update"),
            method: "post",
            data: this.$http.adornData(
              {
                name: value,
                catId:data.catId
              },
              false
            )
          })
            .then(({ data }) => {
              if (data && data.code === 0) {
                // 刷新
                this.getMenus();
                this.expandedKey = [parentCid];
                console.log(parentCid);

                this.$message({
                  message: "修改成功",
                  type: "success",
                  duration: 1500,
                  onClose: () => {
                    this.getDataList();
                  }
                });
              } else {
                this.message.error(data.msg);
              }
            })
            .catch(() => {
              this.$message({
                type: "info",
                message: "修改失败，请检查格式或重试"
              });
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消新增"
          });
        });
    }
  },
  created() {
    // 构建vue时，就发送请求
    this.getMenus();
  }
};
</script>
<style scoped>
.menus-tree .crud-node {
  visibility: hidden;
}

.menus-tree .custom-tree-node:hover .crud-node {
  visibility: visible;
}
</style>