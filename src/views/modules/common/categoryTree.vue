<template>
  <el-tree :data="category"
           :props="defaultProps"
           node-key="catId"
           @node-click="treeNodeClick"
           :highlight-current="true"
           ref="tree">
  </el-tree>
</template>

<script>
export default {
  name: "categoryTree",
  data () {
    return {
      category: [],
      defaultProps: {
        children: 'child',
        label: 'name'
      }
    }
  },
  methods: {
    getCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.category = data.listTree
      })
    },
    treeNodeClick (data, node, components) {
      if (data.catLevel === 3) {
        this.$emit('tree-node-click', data, node, components);
      }
    }
  },
  created () {
    this.getCategory()
  }
}
</script>

<style scoped>

</style>
