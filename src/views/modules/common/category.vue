<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeclick"
  ></el-tree>
</template>

<script>
export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    //这里存数据
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
  //计算属性
  computed: {},
  //监控data中数据变化
  watch: {},
  //方法
  methods: {
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
    // node节点被点击事件回调函数
    nodeclick(data, node, component) {
      console.log(data, node, component);

      // 向父组件发射事件
      this.$emit("tree-node-click", data, node, component);
    }
  },
  //声明周期 - 创建完成（可以访问当前this实例）
  created() {
    // 构建vue时，就发送请求
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //声明周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁之后
  activated() {} //缓存keep-alive
};
</script>

<style scoped>
</style>