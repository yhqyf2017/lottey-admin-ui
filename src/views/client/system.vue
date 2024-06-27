<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" size="mini">
      <el-form-item label="昵称" prop="nickname">
        <el-input
          v-model="queryParams.nickname"
          placeholder="请输入昵称"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="用户名" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入用户名"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="danger"
          icon="el-icon-circle-plus-outline"
          size="mini"
          @click="newSysUser"
          plain
        >添加</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="voList" border stripe>
      <el-table-column label="用户ID" align="center" prop="id" width="100" fixed />
      <el-table-column
        label="用户名"
        align="center"
        prop="username"
        :show-overflow-tooltip="true"
        width="120"
      />
      <el-table-column
        label="昵称"
        align="center"
        prop="name"
        :show-overflow-tooltip="true"
        width="120"
      />
      <el-table-column label="支付密码" align="center" width="100">
        <template slot-scope="scope">
          <span>******</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center" width="200">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新时间" align="center" width="200">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.updateTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button size="mini" type="danger" @click="deleteUser(scope.row)">删除</el-button>
          <el-button size="mini" type="primary" @click="updateUser(scope.row)">更改</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNo"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <add-sys-user-dialog title="添加用户" :visible.sync="dialogVisible" @confirm="addUserReq"></add-sys-user-dialog>
    <!-- 修改 -->
    <div>
      <!-- 修改 -->
      <el-dialog title="用户更改" :visible.sync="dialogFormVisible">
        <el-form :model="form" :rules="rules" label-width="100px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="form.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="旧密码" prop="oldPassword">
            <el-input type="password" v-model="form.oldPassword"></el-input>
          </el-form-item>
          <el-form-item label="新密码" prop="password">
            <el-input type="password" v-model="form.password"></el-input>
          </el-form-item>
          <el-form-item label="旧支付密码" prop="oldPayPwd">
            <el-input type="password" v-model="form.oldPayPwd" oninput="this.value = this.value.replace(/[^0-9]/g, '');"></el-input>
          </el-form-item>
          <el-form-item label="新支付密码" prop="payPwd">
            <el-input type="password" v-model="form.payPwd" oninput="this.value = this.value.replace(/[^0-9]/g, '');"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="submitAdmin">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import {
  addSysUser,
  getSysList,
  deleteSysUser,
  updateAdmin
} from "@/api/client";
import AddSysUserDialog from "./components/AddSysUserDialog.vue";
import { Message, MessageBox } from "element-ui";

export default {
  name: "ClientSystem",
  components: {
    AddSysUserDialog
  },
  props: {},
  data() {
    return {
      // 遮罩层
      loading: false,
      // 总数据条数
      total: 0,
      // 列表数据
      voList: [],
      // 查询参数
      queryParams: {
        username: undefined,
        name: undefined,
        pageNo: 1,
        pageSize: 10
      },
      // 是否显示搜索
      showSearch: true,
      // 弹窗添加用户
      dialogVisible: false,
      dialogFormVisible: false,
      form: {
        username: "",
        oldPassword: "",
        password: "",
        oldPayPwd: "",
        payPwd: ""
      },
      rules: {
        username: [
          {
            required: true,
            trigger: "blur",
            message: "用户名不能为空"
          }
        ],
        oldPassword: [
          {
            required: true,
            message: "旧密码不能为空",
            trigger: "blur"
          }
        ],
        password: [
          {
            required: true,
            message: "密码不能为空",
            trigger: "blur"
          }
        ],
        oldPayPwd: [
          {
            required: true,
            message: "旧支付密码不能为空",
            trigger: "blur"
          }
        ],
        payPwd: [
          {
            required: true,
            message: "支付密码不能为空",
            trigger: "blur"
          }
        ]
      }
    };
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {
    this.getList();
  },
  methods: {
    updateUser(item) {
      console.log(item);
      this.dialogFormVisible = true;
      this.form.username = item.username;
    },
    submitAdmin() {
      updateAdmin(this.form).then(res => {
        console.log(res, "res");
        this.dialogFormVisible = false;
        this.getList();
      });
    },
    // 获取统计列表
    getList() {
      this.loading = true;
      getSysList(this.queryParams).then(response => {
        this.loading = false;
        if (!response.errorCode) {
          this.total = response.total;
          this.voList = response.voList;
        }
      });
    },
    // 查询数据
    handleQuery() {
      this.queryParams.pageNo = 1;
      this.getList();
    },
    // 重置数据
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 新增用户
    newSysUser() {
      this.dialogVisible = true;
    },

    // 新增用户请求
    addUserReq(data) {
      addSysUser(data).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    },

    // 删除用户
    deleteUser(row) {
      MessageBox.confirm(
        "删除该用户将会清理所有相关联的数据,确定要进行操作吗?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      )
        .then(() => {
          deleteSysUser(row.id).then(response => {
            if (!response.errorCode) {
              Message({
                type: "success",
                message: "删除成功"
              });
              this.getList();
            }
          });
        })
        .catch(() => {});
    }
  }
};
</script>

<style scoped lang="scss"></style>
