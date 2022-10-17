<template>
  <div>
    <el-switch v-model="treeDraggable" active-text="开始拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="treeDraggable" @click="saveSort">保存拖拽</el-button>
    <el-tree :data="category"
             :props="defaultProps"
             :expand-on-click-node="false"
             node-key="catId"
             :default-expanded-keys="defaultExpanded"
             show-checkbox
             :draggable="treeDraggable"
             :allow-drop="allowDrop"
             @node-drop="handleDrop">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="dialogTitle" width="30%" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="categoryInfo">
        <el-form-item label="分类名称">
          <el-input v-model="categoryInfo.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="categoryInfo.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类单位">
          <el-input v-model="categoryInfo.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogClose">取 消</el-button>
        <el-button type="primary" @click="submitCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogTitle: '',
      category: [],
      categoryInfo: {
        catId: '',
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''
      },
      defaultExpanded: [],
      dialogFormVisible: false,
      defaultProps: {
        children: 'child',
        label: 'name'
      },
      treeDraggable: false,
      updateCategories: []
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
    append (data) {
      this.dialogTitle = '新增分类'
      this.dialogFormVisible = true
      this.categoryInfo.parentCid = data.catId
      this.categoryInfo.catLevel = data.catLevel * 1 + 1
      this.categoryInfo.sort = data.child.length * 1 + 1
    },
    edit (data) {
      console.log(data)
      this.dialogTitle = '编辑分类'
      // 请求后台获取最新分类数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        this.categoryInfo = data.category
      })
      this.dialogFormVisible = true
    },
    addCategory () {
      var expanded = this.categoryInfo.parentCid
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categoryInfo, false)
      }).then(({data}) => {
        this.$message({
          message: '分类新增成功',
          type: 'success'
        })
        this.dialogClose()
        this.getCategory()
        this.defaultExpanded = [expanded]
      })
    },
    editCategory () {
      var expanded = this.categoryInfo.parentCid
      var {catId, name, icon, productUnit} = this.categoryInfo
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: '分类编辑成功',
          type: 'success'
        })
        this.dialogClose()
        this.getCategory()
        this.defaultExpanded = [expanded]
      })
    },
    submitCategory () {
      if (this.dialogTitle === '新增分类') {
        this.addCategory()
      }
      if (this.dialogTitle === '编辑分类') {
        this.editCategory()
      }
    },
    remove (node, data) {
      this.$confirm(`将删除分类【${data.name}】, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var thisData = data
        var ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '分类删除成功',
            type: 'success'
          })
          this.defaultExpanded = [thisData.parentCid]
          this.getCategory()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    dialogClose () {
      this.categoryInfo = this.$options.data().categoryInfo
      this.dialogFormVisible = false
    },
    // 判断是否可以拖拽至目标层级
    allowDrop (draggingNode, dropNode, type) {
      console.log(draggingNode, dropNode, type)
      // 获取目标层级的最大层级
      var targetMaxDeep = this.getMaxDeep(draggingNode)
      console.log('targetMax:', targetMaxDeep)
      // 最终层级 = 目标最大层级 - 目标本身层级 + 1
      var finalMaxDeep = Math.abs(targetMaxDeep - draggingNode.level) + 1
      console.log('finalMax:', finalMaxDeep)
      if (type === 'inner') {
        return (finalMaxDeep + draggingNode.level) <= 3
      } else {
        return (finalMaxDeep + draggingNode.parent.level) <= 3
      }
    },
    // 获取节点的最深层级的等级
    getMaxDeep (node, max = 0) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > max) {
            max = node.childNodes[i].level
          }
          max = this.getMaxDeep(node.childNodes[i], max)
        }
      }
      return max
    },
    // 拖拽成功后触发的事件
    handleDrop (draggingNode, dropNode, dropType, ev) {
      // 被拖拽节点的最新父节点id
      let newParentCid = 0
      // 需要更新排序的分类
      let siblings = null

      if (dropType === 'inner') {
        // 拖拽至目标节点之内
        // 父节点等于目标节点
        newParentCid = dropNode.data.catId
        // 目标节点内的所有子节点全部更新排序
        siblings = dropNode.childNodes
      } else {
        // 拖拽至目标节点的前或后
        // 父节点等于目标节点的父节点
        newParentCid = dropNode.data.parentCid
        // 目标节点的父节点内的所有子节点全部更新排序
        siblings = dropNode.parent.childNodes
      }

      // 封装需要更新排序的分类对象
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level
          // 且正在拖拽的节点层级发生了变化，那么被拖拽节点的层级及所有子节点的层级都需要变化
          if (siblings[i].level !== draggingNode.level) {
            // 更新层级
            catLevel = siblings[i].level
            // 更新子节点的层级
            this.updateChildNodeLevel(siblings[i], this.updateCategories)
          }
          this.updateCategories.push({catId: siblings[i].data.catId, sort: i, parentCid: newParentCid, catLevel: catLevel})
        } else {
          this.updateCategories.push({catId: siblings[i].data.catId, sort: i})
        }
      }
    },
    // 递归更新子节点的层级
    updateChildNodeLevel (node, updateCategories) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let childData = node.childNodes[i].data
          updateCategories.push({catId: childData.catId, catLevel: node.childNodes[i].level})
          this.updateChildNodeLevel(node.childNodes[i], updateCategories)
        }
      }
    },
    // 保存拖拽
    saveSort () {
      if (this.updateCategories.length === 0) {
        this.$message({
          message: '分类顺序未改动',
          type: 'info'
        })
      } else {
        this.$http({
          url: this.$http.adornUrl('/product/category/update/sort'),
          method: 'post',
          data: this.$http.adornData(this.updateCategories, false)
        }).then(({data}) => {
          this.$message({
            message: '分类调整成功',
            type: 'success'
          })
          this.dialogClose()
          this.getCategory()
        })
        this.updateCategories = []
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
