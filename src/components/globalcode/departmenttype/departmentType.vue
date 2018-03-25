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
        <el-col :span="2" :offset="15">
          <el-button style="float: right;" type="primary" @click="openDialog">新 增</el-button>
        </el-col>
      </el-row>
    </div>
    <el-table :data="tableData" border>
      <el-table-column
        prop="departmentTypeId"
        label="分类编码">
      </el-table-column>
      <el-table-column
        prop="departmentTypeName"
        label="分类名称">
      </el-table-column>
      <el-table-column
        prop="remark"
        label="说明">
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="filter.currentPage"
      :page-sizes="[5, 10, 15]"
      :page-size="filter.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableDataLength"
      style="float:right">
    </el-pagination>
    <el-dialog title="新增科室分类" :visible.sync="dialog.visible" width="35%" :before-close="handleClose">
      <el-form :model="dialog.form" ref="dialog.form" label-width="100px" :rules="dialog.rules">
        <el-form-item label="分类名称" prop="departmentTypeName">
          <el-input v-model="dialog.form.departmentTypeName"></el-input>
        </el-form-item>
        <el-form-item label="说明" prop="remark">
          <el-input type="textarea" v-model="dialog.form.remark"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="">新增</el-button>
          <el-button>取消</el-button>
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
        currentPage: 1
      },
      dialog: {
        form: {},
        visible: false,
        rules: {
          departmentTypeName: [
            { required: true, message: '请输入分类名称', trigger: 'blur' },
            { min: 2, max: 8, message: '长度在 2 到 8 个字符', trigger: 'blur' }
          ],
        }
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
    openDialog () {
      this.dialog.visible = true
    },
    handleClose () {
      this.dialog.visible = false;
      // 重置表单
      this.$refs['dialog.form'].resetFields();
    },
    handleSizeChange (val) {
      this.pageSize = val
      this.initPage()
    },
    handleCurrentChange (val) {
      this.currentPage = val
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
