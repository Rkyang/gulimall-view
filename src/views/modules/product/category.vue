<template>
  <div>
    <el-tree :data="category"
             :props="defaultProps"
             :expand-on-click-node="false"
             node-key="catId"
             :default-expanded-keys="defaultExpanded"
             show-checkbox>
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
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="新增分类" width="20%" :visible.sync="dialogFormVisible">
      <el-form :model="categoryInfo">
        <el-form-item label="分类名称">
          <el-input v-model="categoryInfo.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogClose">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      category: [],
      categoryInfo: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
      },
      defaultExpanded: [],
      dialogFormVisible: false,
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
    append (data) {
      this.dialogFormVisible = true
      this.categoryInfo.parentCid = data.catId
      this.categoryInfo.catLevel = data.catLevel * 1 + 1
      this.categoryInfo.sort = data.child.length * 1 + 1
    },
    addCategory () {
      var expanded = this.categoryInfo.parentCid
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.categoryInfo, false)
      }).then(({data}) => {
        this.$message({
          message: '分类添加成功',
          type: 'success'
        })
        this.dialogClose()
        this.getCategory()
        this.defaultExpanded = [expanded]
      })
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
    }
  },
  created () {
    this.getCategory()
  }
}
</script>

<style scoped>

</style>
