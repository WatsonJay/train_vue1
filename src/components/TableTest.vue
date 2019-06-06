<template>
  <div style="width: 80%;margin-left: auto;margin-right: auto">
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.name" placeholder="姓名"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="loadData">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="success" @click="addUser">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <el-table :data="userInfoList" style="width: 100%">
      <el-table-column prop="id" label="id" width="180">
      </el-table-column>
      <el-table-column prop="name" label="名字" width="180">
      </el-table-column>
      <el-table-column prop="password" label="密码" width="180">
      </el-table-column>
      <!--第二步  开始进行修改和查询操作-->
      <el-table-column label="操作" align="center" min-width="100">
        <template slot-scope="scope">
          <el-button type="info" @click="checkDetail(scope.row)">查看详情</el-button>
          <el-button type="warning" @click="modifyUser(scope.row)">修改</el-button>
          <el-button type="danger" @click="deleteUser(scope.row)">删除</el-button>
        </template>
      </el-table-column>
      <!--编辑与增加的页面-->
    </el-table>
    <div style="width: 80%;margin-right: auto;margin-left: auto">
      <el-dialog title="记录" :visible.sync="dialogVisible" width="50%" :close-on-click-modal="false">
        <el-form :model="addFormData" :rules="rules2" ref="addFormData" label-width="120px" class="demo-ruleForm login-container" style="width: 90%;margin-right: auto;margin-left: auto">
          <el-form-item prop="name" label="用户名">
            <el-input type="text" :disabled="isEdit" v-model="addFormData.name" auto-complete="off" placeholder="账号"></el-input>
          </el-form-item>
          <el-form-item prop="password" label="密码">
            <el-input type="password" :disabled="isEdit" v-model="addFormData.password" auto-complete="off" placeholder="密码"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
         <el-button type="danger" @click.native="dialogVisible = false,addFormData={id:0  ,name:'',password:''}">取 消</el-button>
         <el-button v-if="isView" type="success" @click.native="sumbitData">确 定</el-button>
     </span>
      </el-dialog>
    </div>
    <!--新增界面-->
  </div>
</template>

<script>
  import axios from 'axios';
  import qs from 'qs';
  axios.defaults.baseURL = 'http://localhost:8081'
  export default {
      name: 'TableTest',
      data() {
        return {
          userInfoList: [],
          addFormData:{
            id: 0,
            name: '',
            password: ''
          },
          filters: {
            name:''
          },
          dialogVisible: false,
          isView: true,
          isEdit: false,
          rules2: {
            name: [{
              required: true,
              message: '用户名不能为空',
              trigger: 'blur'
            }],
            password: [{
              required: true,
              message: '密码不能为空',
              trigger: 'blur'
            }]
          },
        }
      },
      mounted: function () {
          this.loadData();
      },
      methods: {
        loadData() {
        let param = {filter:this.filters.name};
          axios.post('/user/userlist',qs.stringify(param)).then((result) => {
            var _data = result.data;
            this.userInfoList = _data;
          });
        },
        addUser(){
          this.isView = true
          this.isEdit = false
          this.dialogVisible = true
        },
        checkDetail(rowData){
          this.addFormData = Object.assign({}, rowData);
          this.isView = false
          this.isEdit = true
          this.dialogVisible = true
        },
        modifyUser(rowData){
          this.addFormData = Object.assign({}, rowData);
          this.isView = true
          this.isEdit = false
          this.dialogVisible = true
        },
        sumbitData(){
          //先判断表单是否通过了判断
          this.$refs.addFormData.validate((valid) => {
            //代表通过验证 ,将参数传回后台
            if (valid) {
              let param = Object.assign({}, this.addFormData);
              axios.post("/user/submit", qs.stringify(param)).then((result) => {
                if (result.data.success) {
                  this.$message({
                    type: 'info',
                    message: '添加成功',
                  });
                  this.loadData();
                } else {
                  this.$message({
                    type: 'info',
                    message: '添加失败',
                  });
                }
                this.dialogVisible = false;
              });
            }
          });
        },
        deleteUser(rowData){
          this.$alert('是否删除该记录','信息删除',{
            confirmButtonText: '确定',
            callback:action => {
              var params = {
                id: rowData.id
              };
              axios.post("/user/deleteUser", qs.stringify(params)).then((result) =>{
                if(result.data.success){
                  this.$message({
                    type: 'info',
                    message: '已删除'
                  });
                }else {
                  this.$message({
                    type: 'info',
                    message: '删除失败'
                  });
                }
                this.loadData();
              });
            }
          });
        }
      }
    }
</script>

<style scoped>

</style>
