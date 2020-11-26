<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> Account
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button
            type="primary"
            icon="el-icon-delete"
            class="handle-del mr10"
            @click="delAllSelection"
        >批量删除
        </el-button>
        <el-select v-model="query.address" placeholder="地址" class="handle-select mr10">
          <el-option key="1" label="广东省" value="广东省"></el-option>
          <el-option key="2" label="湖南省" value="湖南省"></el-option>
        </el-select>
        <el-input v-model="query.name" placeholder="username" class="handle-input mr10"></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
      </div>
      <el-table
          :data="tableData"
          border
          class="table"
          ref="multipleTable"
          header-cell-class-name="table-header"
          @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" align="center"></el-table-column>
        <el-table-column prop="userId" label="ID" width="120"></el-table-column>
        <el-table-column prop="name" label="Username" width="160"></el-table-column>
        <el-table-column prop="type" label="Role" width="150"></el-table-column>
        <!--        <el-table-column label="账户余额">-->
        <!--          <template slot-scope="scope">￥{{ scope.row.money }}</template>-->
        <!--        </el-table-column>-->
        <el-table-column prop="facultyId" label="Faculty" width="150"></el-table-column>
        <el-table-column prop="program" label="Program" width="220"></el-table-column>
        <el-table-column prop="email" label="Email" width="220"></el-table-column>
        <el-table-column label="Status" align="center" width="100">
          <template slot-scope="scope">
            <el-tag
                :type="scope.row.accountStatus==='current'?'success':scope.row.accountStatus==='unauthorized'?'primary':scope.row.accountStatus==='expelled'?'danger':scope.row.accountStatus==='sabbatical'?'info':scope.row.accountStatus==='alumni'?'default':''"
            >{{ scope.row.accountStatus }}
            </el-tag>
          </template>
        </el-table-column>

        <el-table-column prop="lastLogin" label="Last login" width="230"></el-table-column>
        <el-table-column label="Operator" width="180" align="center">
          <template slot-scope="scope">
            <el-button
                type="text"
                icon="el-icon-edit"
                @click="handleEdit(scope.$index, scope.row)"
            >Modify
            </el-button>
            <el-button
                type="text"
                icon="el-icon-delete"
                class="red"
                @click="handleDelete(scope.$index, scope.row)"
            >delete
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
            background
            layout="total, prev, pager, next"
            :current-page="query.pageIndex"
            :page-size="query.pageSize"
            :total="pageTotal"
            @current-change="handlePageChange"
        ></el-pagination>
      </div>
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
      <el-form ref="form" :model="form" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="form.address"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
    </el-dialog>
  </div>
</template>

<script>
import {fetchData} from "@/api";
import axios from "axios";

export default {
  name: "AccountManager",
  data() {
    return {
      query: {
        address: '',
        name: '',
        pageIndex: 1,
        pageSize: 10
      },
      pageSize: this.pageSize,
      pageTotal: this.pageTotal,
      tableData: [],

      multipleSelection: [],
      delList: [],
      editVisible: false,

      form: {},
      idx: -1,
      id: -1
    };
  },
  created() {
    //this.getData();
    axios.get('http://localhost:8080/admin/account/getAll/0/9').then(resp => {
      this.tableData = resp.data.content
      this.pageSize = resp.data.size
      this.pageTotal = resp.data.totalElements
    })
  },
  methods: {
    // 获取 easy-mock 的模拟数据
    getData() {
      fetchData(this.query).then(res => {
        console.log(res);
        this.tableData = res.list;
        this.pageTotal = res.pageTotal || 50;
      });
    },
    // 触发搜索按钮
    handleSearch() {
      this.$set(this.query, 'pageIndex', 1);
      this.getData();
    },
    // 删除操作
    handleDelete(index, row) {
      // 二次确认删除
      this.$confirm('确定要删除吗？', '提示', {
        type: 'warning'
      })
          .then(() => {
            this.$message.success('删除成功');
            this.tableData.splice(index, 1);
          })
          .catch(() => {
          });
    },
    // 多选操作
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    delAllSelection() {
      const length = this.multipleSelection.length;
      let str = '';
      this.delList = this.delList.concat(this.multipleSelection);
      for (let i = 0; i < length; i++) {
        str += this.multipleSelection[i].name + ' ';
      }
      this.$message.error(`删除了${str}`);
      this.multipleSelection = [];
    },
    // 编辑操作
    handleEdit(index, row) {
      this.idx = index;
      this.form = row;
      this.editVisible = true;
    },
    // 保存编辑
    saveEdit() {
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
      this.$set(this.tableData, this.idx, this.form);
    },
    // 分页导航
    handlePageChange(currentPage) {
      axios.get('http://localhost:8080/admin/account/getAll/' + (currentPage - 1) + '/6').then(resp => {
        console.log(resp)
        this.tableData = resp.data.content
        this.pageSize = resp.data.size
        this.pageTotal = resp.data.totalElements
      })
    }
  }
}
</script>

<style scoped>
.container {
  text-align: left;
}

.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}

.table {
  width: 100%;
  font-size: 14px;
}

.red {
  color: #ff0000;
}

.mr10 {
  margin-right: 10px;
}

.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>