import style from './user.module.css'

<template lang="">
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20"
          ><el-input
            v-model="searchModel.username"
            placeholder="username"
            clearable
          ></el-input>
          <el-input
            v-model="searchModel.phone"
            placeholder="phone number"
            clearable
          ></el-input>
          <el-button
            @click="getUserList"
            type="primary"
            round
            icon="el-icon-search"
            >Search</el-button
          ></el-col
        >
        <el-col :span="4" align="right">
          <el-button type="primary" icon="el-icon-edit" circle> </el-button>
        </el-col>
      </el-row>
    </el-card>

    <el-card>
      <el-table :data="userList" stripe style="width: 100%">
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{
              (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1
            }}
          </template>
        </el-table-column>
        <el-table-column prop="id" label="userID" width="180">
        </el-table-column>
        <el-table-column prop="username" label="username" width="180">
        </el-table-column>
        <el-table-column prop="phone" label="phone" width="180">
        </el-table-column>
        <el-table-column prop="email" label="email"> </el-table-column>
        <el-table-column label="edit" width="180"> </el-table-column>
      </el-table>
    </el-card>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="searchModel.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>
  </div>
</template>
<script>
import userApi from "@/api/userManage";
export default {
  data() {
    return {
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 10,
      },
      userList: [],
    };
  },
  methods: {
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize;
      this.getUserList();
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo;
      this.getUserList();
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then((response) => {
        this.userList = response.data.rows;
        this.total = response.data.total;
      });
    },
  },
  created() {
    this.getUserList();
  },
};
</script>
<style lang=""></style>

<style>
#search .el-input {
  width: 250px;
  margin: 0px 10px 0px 0px;
}
</style>
