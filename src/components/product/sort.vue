<template>
    <div class="qtt_table">
      <div class="filter-container">
        <el-button class="filter-item" type="primary" @click="handelCreate"  icon="edit">添加</el-button>
        <el-button class="filter-item" type="primary" @click="handleDelAll"  icon="delete">全部删除</el-button>
        <el-button class="filter-item" type="primary" icon="document" @click="handleDownload">导出</el-button>
      </div>

      <el-table ref="multipleTable" :data="table" border tooltip-effect="dark"  style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column label="序号" width="120">
      <template scope="scope">{{ scope.$index+1 }}</template>
      </el-table-column>
      <el-table-column prop="sor_id" label="分类编号" sortable show-overflow-tooltip width="120"></el-table-column>
      <el-table-column prop="sor_name" label="分类名称"  width="350"></el-table-column>
      <el-table-column prop="add_data" label="添加时间"  width="350" sortable show-overflow-tooltip></el-table-column>
      <el-table-column align="center" label="操作">
      <template scope="scope">
        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
        <el-button size="small" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
      </template>
      </el-table-column>
      </el-table>

      
      <!-- 添加信息功能的表单 -->
      <el-dialog title="添加分类" :visible.sync="dialogFormVisible">
        <el-form class="small-space" :model="form" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;'>
          <el-form-item label="分类名称">
            <el-input v-model="form.sor_name"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>          
          <el-button type="primary" @click="handleCreateSubmit">确 定</el-button>
          <el-button type="primary" @click="resetForm('form')">重置</el-button>
        </div>
      </el-dialog>
      <!-- 编辑信息的表单 -->
      <el-dialog title="修改分类" :visible.sync="dialogFormEditVisible">
        <el-form class="small-space" :model="form" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;'>
          <el-form-item label="分类名称">
            <el-input v-model="tableEdit.sor_name"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormEditVisible = false">取 消</el-button>          
          <el-button type="primary" @click="handleEditSubmit">确 定</el-button>
        </div>
      </el-dialog>
    
    </div>
</template>

<script type="text/ecmascript-6">
import {api} from '@/global/api.js';
import {http} from '@/global/http.js';
export default {
  data () {
    return {
      table: [],
      tableEdit: [],
      total: null,
      dialogFormVisible: false,
      dialogFormEditVisible: false,
      form: {
        sor_name: '',
        add_data: ''
      },
      multipleSelection: []
    };
  },
  methods: {
    handleSelectionChange (val) {
      this.multipleSelection = val;
    },
    // 编辑
    handleEdit (index, row) {
      this.dialogFormEditVisible = true;
      this.tableEdit = row;
    },
    // 编辑表单的提交
    handleEditSubmit (index) {
      let this_ = this;
      http.ajaxPost(this, api.sort_edit, {sor_id: this.tableEdit.sor_id, sor_name: this.tableEdit.sor_name}, function (data) {
        this_.dialogFormEditVisible = false;
        this_.table[index] = this_.tableEdit;
      });
    },
    // 选择删除
    handleDelete (index, row) {
      // 删除当前一个
      let deleteArr = [];
      if (this.multipleSelection.indexOf(row) === -1) {
        deleteArr.push(row);
      } else {
        deleteArr = this.multipleSelection;
      };
      let deleteArrId = [];
      deleteArr.forEach(function (e) {
        deleteArrId.push(e.sor_id);
      });
      let this_ = this;
      let newTable = [];
      http.ajaxPost(this, api.sort_delete, {deleteArr: deleteArrId}, function (data) {
        this_.table.forEach(function (val) {
          if (deleteArr.indexOf(val) === -1) {
            newTable.push(val);
          }
        });
        this_.table = newTable;
      });
    },
    // 全部删除
    handleDelAll () {
      let this_ = this;
      http.ajaxPost(this, api.sort_delAll, {}, function (data) {
        this_.table = [];
      });
    },
    // 弹出添加弹窗
    handelCreate () {
      this.dialogFormVisible = true;
      this.resetForm();
    },
    // 导出为表格
    handleDownload () {
      var vm = this;
      require.ensure([], () => {
        const { export_json_to_excel } = require('vendor/Export2Excel');
        const tHeader = ['分类编号', '分类名称', '添加时间'];
        const filterVal = ['sor_id', 'sor_name', 'add_data'];
        const table = vm.table;
        const data = vm.formatJson(filterVal, table);
        export_json_to_excel(tHeader, data, '商品分类列表');
      });
    },
    formatJson (filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]));
    },
    handleCreateSubmit () {
      let vm = this;
      if (!vm.form.sor_name) {
        this.$message({
          message: '请输入分类名称',
          type: 'warning'
        });
        return;
      }
      http.ajaxPost(this, api.sort_add, {sor_name: vm.form.sor_name}, function (data) {
        if (data.code === 200) {
          vm.table.push(data.data[0]);
          vm.dialogFormVisible = false;
        }
      });
    },
    // 重置添加表单
    resetForm () {
      this.form = {
      };
    }
  },
  created () {
    // 获取所有分类
    let this_ = this;
    http.ajaxGet(this, api.sort_all, {}, function (data) {
      this_.table = data.data;
    });
  }
};
</script>

<style lang="stylus-loader" rel="stylesheet/stylus" scoped>   
  .qtt_table
    padding:10 10px
    .filter-container
      margin-bottom:20px
    .pagination-container
      margin-top:50px
      margin-left:-5px
</style>

