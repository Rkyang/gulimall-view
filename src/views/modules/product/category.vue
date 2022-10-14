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
    }
  },
  created () {
    this.getCategory()
  }
}
</script>

<style scoped>

</style>
