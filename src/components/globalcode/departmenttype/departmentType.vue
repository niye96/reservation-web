<template>
  <div>
    <div class="top-bar">
      <el-row>
        <el-col :span="5">
          <el-input placeholder="请输入分类名称" prefix-icon="el-icon-search" v-model="filter.departmentTypeName" @keyup.enter.native="initPage">
          </el-input>
        </el-col>
        <el-col :span="2">
          <el-button style="margin-left:10px;" @click="initPage" plain>搜 索</el-button>
        </el-col>
        <el-col :span="2" :offset="13">
          <el-button style="float: right" type="primary" @click="openAddDialog">新 增</el-button>
        </el-col>
        <el-col :span="2">
          <el-button style="margin-left: 20px" type="primary" @click="deleteDepartmentType">删 除</el-button>
        </el-col>
      </el-row>
    </div>
    <el-table :data="tableData" border>
      <el-table-column
        prop="departmentTypeId"
        label="分类编码"
        width="100px">
      </el-table-column>
      <el-table-column
        prop="departmentTypeName"
        label="分类名称"
        width="200px">
      </el-table-column>
      <el-table-column
        prop="remark"
        label="说明">
      </el-table-column>
      <el-table-column
        label="操作"
        width="100px">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="openUpdateDialog(scope.row)">编辑</el-button> |
          <el-button type="text" size="small" style="margin-left: 0" @click="deleteDepartmentType(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="filter.pageNo"
      :page-sizes="[5, 10, 15]"
      :page-size="filter.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableDataLength"
      style="float:right">
    </el-pagination>
    <el-dialog title="新增科室分类" :visible.sync="addDialog.visible" width="35%" :before-close="closeAddDialog">
      <el-form :model="addDialog.form" ref="addDialog.form" label-width="100px" :rules="rules">
        <el-form-item label="分类名称" prop="departmentTypeName">
          <el-input v-model="addDialog.form.departmentTypeName"></el-input>
        </el-form-item>
        <el-form-item label="说明" prop="remark">
          <el-input type="textarea" v-model="addDialog.form.remark"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit('addDialog.form')">新增</el-button>
          <el-button>取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <el-dialog title="编辑科室分类" :visible.sync="updateDialog.visible" width="35%" :before-close="closeUpdateDialog">
      <el-form :model="updateDialog.form" ref="updateDialog.form" label-width="100px" :rules="rules">
        <el-form-item label="分类名称" prop="departmentTypeName">
          <el-input v-model="updateDialog.form.departmentTypeName"></el-input>
        </el-form-item>
        <el-form-item label="说明" prop="remark">
          <el-input type="textarea" v-model="updateDialog.form.remark"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit('updateDialog.form')">保存</el-button>
          <el-button @click="closeUpdateDialog">取消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
import BizService from '../../../services/biz-service.js'
var service = new BizService()
export default {
  data () {
    return {
      tableData: [],
      tableDataLength: null,
      filter: {
        departmentTypeName: null,
        pageSize: 10,
        pageNo: 1
      },
      addDialog: {
        form: {},
        visible: false,
      },
      updateDialog: {
        form: {},
        visible: false,
      },
      rules: {
        departmentTypeName: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.initPage()
  },
  methods: {
    initPage () {
      var params = this.filter
      service.getDepartmentTypeList(params,(isOk, data) => {
        if (isOk == true){
          this.tableData = data.data
          this.tableDataLength = data.total
        } else{
          this.$message({
            type: 'warning',
            message: data.message
          })
        }
      })
    },
    addDepartmentType () {
      var params = this.addDialog.form
      service.addDepartmentType(params,(isOk, data) => {
        if (isOk == true){
          this.$message({
            type: 'success',
            message: '新增成功'
          })
          this.closeAddDialog()
          this.initPage()
        } else{
          this.$message({
            type: 'warning',
            message: '新增失败'
          })
        }
      })
    },
    updateDepartmentType () {
      var params = this.updateDialog.form
      if (params == null || params.departmentTypeId == null) return
      service.updateDepartmentType(params,(isOk, data) => {
        if (isOk == true){
          this.$message({
            type: 'success',
            message: '保存成功'
          })
          this.updateDialog.visible = false
          this.$refs['updateDialog.form'].resetFields()
          this.initPage()
        } else{
          this.$message({
            type: 'warning',
            message: '保存失败'
          })
        }
      })
    },
    deleteDepartmentType (row) {
      console.log(row)
      this.$confirm('删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        if (row == null || row.departmentTypeId == null) return
        service.deleteDepartmentType({departmentTypeId: row.departmentTypeId},(isOk, data) => {
          if (isOk == true){
            this.$message({
              type: 'success',
              message: '删除成功'
            })
            this.initPage()
          } else{
            this.$message({
              type: 'warning',
              message: '删除失败'
            })
          }
        })
      }).catch(() => {
      });
    },
    onSubmit (form) {
      this.$refs[form].validate(valid => {
        if(valid){
          if (form === 'addDialog.form') {
            this.addDepartmentType()
          }else if (form === 'updateDialog.form') {
            this.updateDepartmentType()
          }
        }
      });
    },
    openAddDialog () {
      this.addDialog.visible = true
    },
    closeAddDialog () {
      this.addDialog.visible = false;
      // 重置表单
      this.$refs['addDialog.form'].resetFields();
    },
    openUpdateDialog (row) {
      this.updateDialog.visible = true
      this.updateDialog.form = GBFL.Util.deepCopy(row)
    },
    closeUpdateDialog () {
      this.$confirm('退出后编辑的数据将会丢失, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.updateDialog.visible = false;
        // 重置表单
        this.$refs['updateDialog.form'].resetFields();
      }).catch(() => {
      });
    },
    handleSizeChange (val) {
      this.filter.pageSize = val
      this.initPage()
    },
    handleCurrentChange (val) {
      this.filter.pageNo = val
      this.initPage()
    }
  }
}
</script>
<style>
  .top-bar {
    margin-top: 10px;
    margin-bottom: 20px;
  }
  .top-bar .title {
    float: left;
    line-height: 40px;
    margin-right: 30px;
  }
  .top-bar .title :before {
    content: '';
    border-left: 2px solid #88B7E0;
    margin-right: 10px;
  }
  .el-form-item {
    width: 90%;
  }
</style>
