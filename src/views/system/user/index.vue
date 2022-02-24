<template>
    <el-row class="container" :gutter="20">
      <el-col :span="4" :xs="24">
        <div class="grid-content">
              <el-input
                placeholder="请输入部门名称"
              prefix-icon="el-icon-search"
                v-model="filterText"
                style="margin-bottom: 20px">
              </el-input>

              <el-tree
                class="filter-tree"
                :data="data"
                :props="defaultProps"
                default-expand-all
                :filter-node-method="filterNode"
                ref="tree"
                @node-click="handleNodeClick">
              </el-tree>
        </div>
      </el-col>
      <el-col :span="20" :xs="24">
        <div class="grid-content">
          <el-form :inline="true" :model="params" class="demo-form-inline">
            <el-form-item label="用户名称">
              <el-input v-model="params.userName" placeholder="请输入用户名称" style="width:240px;"></el-input>
            </el-form-item>
            <el-form-item label="手机号码">
              <el-input v-model="params.phonenumber" placeholder="请输入手机号码" style="width:240px;"></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="params.status" placeholder="用户状态" style="width:240px;">
                <el-option label="正常" value="0"></el-option>
                <el-option label="停用" value="1"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="创建时间">
              <el-date-picker type="daterange" v-model="params.dateRange" value-format="yyyy-MM-dd" range-separator="-" start-placeholder="开始日期" end-placeholder="结束日期" style="width:240px;"></el-date-picker>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" icon="el-icon-search" @click="submitForm('ruleForm')">搜索</el-button>
              <el-button icon="el-icon-refresh" @click="resetForm">重置</el-button>
            </el-form-item>
          </el-form>
            <el-row :gutter="10" style="margin-bottom:10px;">
            <el-col :span="1.5"><el-button plain type="primary" icon="el-icon-plus
">新增</el-button></el-col>
            <el-col :span="1.5"><el-button plain type="success" icon="el-icon-edit
">修改</el-button></el-col>
            <el-col :span="1.5"><el-button plain type="danger" icon="el-icon-delete">删除</el-button></el-col>
            <el-col :span="1.5"><el-button plain type="info" icon="el-icon-upload2">导入</el-button></el-col>
            <el-col :span="1.5"><el-button plain type="warning" icon="el-icon-download">导出</el-button></el-col>
            <right-toolbar :showSearch.sync="showSearch" @queryTable="getList" :columns="columns"></right-toolbar>
          </el-row>
        </div>
        <el-table
          :data="tableData"
          style="width:100%"
          max-height="250"
          @selection-change="handleSelectionChange">
          <el-table-column
            type="selection"
            align="center"
            width="30">
          </el-table-column>
          <el-table-column
            prop="userId"
            label="用户编号"
            align="center"
            width="100">
          </el-table-column>
          <el-table-column
            prop="userName"
            label="用户名称"
            align="center"
            width="100">
          </el-table-column>
          <el-table-column
            prop="nickName"
            label="用户昵称"
            align="center"
            width="100">
          </el-table-column>
          <el-table-column
            prop="dept.deptName"
            label="部门"
            align="center"
            width="100">
          </el-table-column>
          <el-table-column
            prop="phonenumber"
            label="手机号码"
            align="center"
            width="110">
          </el-table-column>
          <el-table-column
            prop="state"
            label="状态"
            align="center"
            width="55">
            <template slot-scope="scope">
              <el-switch
                v-model="scope.row.status"
                active-value="0"
                inactive-value="1"
              ></el-switch>
            </template>
          </el-table-column>
          <el-table-column
            prop="time"
            label="创建时间"
            align="center"
            width="200">
            <template slot-scope="scope">
              <span>{{ parseTime(scope.row.createTime) }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            width="150">
            <template slot-scope="scope">
              <el-button
                @click.native.prevent="update(scope.$index, tableData)"
                type="text"
                size="small">
                修改
              </el-button>
              <el-button
                @click.native.prevent="deleteRow(scope.$index, tableData)"
                type="text"
                size="small">
                删除
              </el-button>
              <el-dropdown>
                <span class="el-dropdown-link">
                  更多<i class="el-icon-arrow-down el-icon--left"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item icon="el-icon-plus">重置密码</el-dropdown-item>
                  <el-dropdown-item icon="el-icon-circle-plus">分配角色</el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </template>
          </el-table-column>
        </el-table>
        <div class="block" style="width:100%">
          <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pageNum"
            :page-sizes="[10, 20, 30, 40,50]"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
            style="float:right;margin-top: 20px;">
          </el-pagination>
  </div>
      </el-col>
    </el-row>
</template>

<script>
import {treeselect} from '@/api/system/dept'
import {listUser} from '@/api/system/user'
  export default {
    data() {
      return {
        tableData:[],
        dateRange: [],
        filterText: '',
        data: [],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        formInline: {
          user: '',
          region: ''
        },
        form: {
          name: '',
          region: '',
          date1: '',
          date2: '',
          delivery: false,
          type: [],
          resource: '',
          desc: ''
        },
        pageNum: 1,
        pageSize: 10,
        total: 0,
        params: {
          pageNum: 1,
          pageSize: 10,
        }
      };
    },
    watch: {
      filterText(val) {
        this.$refs.tree.filter(val);
      }
    },
    created () {
      treeselect().then((result) => {
        console.log(result)
        this.data = result.data;
      }),
      this.getUserList({
          pageNum: this.pageNum, 
          pageSize: this.pageSize
      });
    },
    methods: {
      onSubmit() {
        console.log('submit!');
      },
      submitForm(formName) {
        console.log(this.params);
        const params = {};
        const newParams = {
          ...this.params,
          params: {
            ['beginTime']: this.params.dateRange && this.params.dateRange[0],
            ['endTime']: this.params.dateRange && this.params.dateRange[1],
          } 
        };
        delete newParams.dateRange;
        this.params = newParams;
        this.getUserList();
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      },
      filterNode(value, data) {
        if (!value) return true;
        return data.label.indexOf(value) !== -1;
      },
      handleNodeClick(data) {
        console.log(data)
        this.pageNum = 0;
        this.pageSize = 10;
        this.params = {
          ...this.params,
          pageNum: 0,
          pageSize: 10,
          deptId: data.id
        }
        this.getUserList();
      },
      handleSelectionChange(val){
        if (val.length > 1) {
          this.$refs.multipleTable.toggleRowSelection(val[0], false)
          val.splice(0, 1)
        }
        this.multipleSelection = val
      },
      handleSizeChange(pageSize){
        this.pageSize = pageSize;
         this.params = {
          ...this.params,
          pageSize: pageSize
        }
         this.getUserList();
      },
      handleCurrentChange(pageNum){
        this.pageNum = pageNum;
        this.params = {
          ...this.params,
          pageNum: pageNum
        }
        this.getUserList();
      },
      getUserList() {
        listUser(this.params).then((result) => {
          this.tableData = result.rows;
          this.total = result.total;
        })
      }
    }
  };
</script>
<style scoped rel="stylesheet/scss" lang="scss">
.container{
  display: flex;
  padding: 15px;
}
.el-row {
    margin-bottom: 20px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  .el-button{
    font-size: 12px;
  }
</style>