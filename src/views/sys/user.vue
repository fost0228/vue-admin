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
          <el-button
            type="primary"
            icon="el-icon-plus"
            @click="openEditUI(null)"
            circle
          >
          </el-button>
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
        <el-table-column prop="status" label="status" width="180">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1">Normal</el-tag>
            <el-tag v-if="scope.row.status == 0" type="danger">Banned</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="email" label="email"> </el-table-column>
        <el-table-column label="edit" width="180">
          <template slot-scope="scope">
            <el-button
              @click="openEditUI(scope.row.id)"
              type="primary"
              icon="el-icon-edit"
              size="mini"
              circle
            ></el-button>
            <el-button
              @click="deleteUser(scope.row)"
              type="danger"
              icon="el-icon-delete"
              size="mini"
              circle
            ></el-button>
          </template>
        </el-table-column>
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

    <el-dialog
      @close="clearForm"
      :title="title"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="userForm" ref="userFormRef" :rules="rules">
        <el-form-item
          label="username"
          prop="username"
          :label-width="formLabelWidth"
        >
          <el-input v-model="userForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item
          v-if="userForm.id == null || userForm.id == undefined"
          label="password"
          prop="password"
          :label-width="formLabelWidth"
        >
          <el-input
            type="password"
            v-model="userForm.password"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="phone" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="status" :label-width="formLabelWidth">
          <el-switch
            v-model="userForm.status"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item prop="email" label="email" :label-width="formLabelWidth">
          <el-input v-model="userForm.email" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">Cancel</el-button>
        <el-button type="primary" @click="saveUser">Add</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg =
        /^[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*@[a-zA-Z0-9]+([-_.][a-zA-Z0-9]+)*\.[a-z]{2,}$/;
      if (!reg.test(value)) {
        return callback(new Error("email format is wrong"));
      }
      callback();
    };

    return {
      formLabelWidth: "130px",
      userForm: {},
      dialogFormVisible: false,
      title: "",
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 10,
      },
      userList: [],
      rules: {
        username: [
          { required: true, message: "please input username", trigger: "blur" },
          { min: 3, max: 50, message: "3 to 50 letters", trigger: "blur" },
        ],
        password: [
          { required: true, message: "please input password", trigger: "blur" },
          { min: 6, max: 16, message: "6 to 16 letters", trigger: "blur" },
        ],
        email: [
          { required: true, message: "please input email", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    deleteUser(user) {
      this.$confirm(`Delete user ${user.username} ?`, `Notice`, {
        confirmButtonText: "Yes",
        cancelButtonText: "Cancel",
        type: "warning",
      })
        .then(() => {
          userApi.deleteUserById(user.id).then((response) => {
            this.$message({
              type: "success",
              message: response.message,
            });
            this.getUserList();
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Delete canceled",
          });
        });
    },
    saveUser() {
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          //submit to backend
          userApi.saveUser(this.userForm).then((response) => {
            this.$message({
              message: response.message,
              type: "success",
            });
            this.dialogFormVisible = false;
            this.getUserList();
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    clearForm() {
      this.userForm = {};
      this.$refs.userFormRef.clearValidate();
    },
    openEditUI(id) {
      if (id == null) {
        this.title = "Add User";
      } else {
        this.title = "Update User";
        userApi.getUserById(id).then((response) => {
          this.userForm = response.data;
        });
      }

      this.dialogFormVisible = true;
    },
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
.el-dialog {
  width: 600px;
}
.el-dialog .el-input {
  width: 70%;
}
</style>
