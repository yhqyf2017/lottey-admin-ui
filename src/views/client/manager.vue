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
      <el-form-item label="手机号" prop="phone">
        <el-input
          v-model="queryParams.phone"
          placeholder="请输入手机号"
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
          @click="newUser"
          plain
        >添加</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="voList" border stripe>
      <!-- <el-table-column label="用户ID" align="center" prop="id" width="100" fixed /> -->
      <el-table-column
        label="昵称"
        align="center"
        prop="nickname"
        :show-overflow-tooltip="true"
        width="120"
        fixed
      />
      <el-table-column
        label="手机号"
        align="center"
        prop="phone"
        :show-overflow-tooltip="true"
        width="120"
      />
      <el-table-column label="性别" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ getUserSex(scope.row) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="彩金" align="center" prop="gold" width="120" />
      <el-table-column label="奖金" align="center" prop="price" width="120" />
      <!-- <el-table-column label="姓名" align="center" prop="name" width="120" />
      <el-table-column label="身份证号" align="center" prop="idCard" width="200" />-->
      <el-table-column label="是否模拟用户" align="center" prop="idCard" width="200">
        <template slot-scope="scope">
          <span>{{scope.row.moni == 0 ? '否' : '是'}}.</span>
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
      <el-table-column label="实名状态" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.isReal === "0" ? "未实名" : "实名" }}</span>
        </template>
      </el-table-column>
      <el-table-column label="是否是代理" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.isAgent === "1" ? "是" : "否" }}</span>
        </template>
      </el-table-column>
      <el-table-column label="账号状态" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.status === "0" ? "正常" : "禁用" }}</span>
        </template>
      </el-table-column>
      <el-table-column label="上级" align="center" prop="parentName" />
      <el-table-column label="操作" align="center" width="400">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            @click="addGodList(scope.row)"
          >{{scope.row.dsSort != null ? '取消大神榜' : '设置大神榜'}}</el-button>
          <el-button size="mini" :disabled="scope.row.id == 1" type="danger" @click="deleteUser(scope.row)">删除</el-button>
          <el-button size="mini" type="primary" @click="disableUser(scope.row)">
            {{
            scope.row.status === "1" ? "启用" : "禁用"
            }}
          </el-button>
          <el-button size="mini" type="success" @click="recharegeUser(scope.row)">充值</el-button>
          <el-button size="mini" type="danger" @click="updateUserPwdAction(scope.row)">修改密码</el-button>
          <el-button
            size="mini"
            :type="getUserAgentStyle(scope.row)"
            @click="setUserAgent(scope.row)"
          >{{ scope.row.isAgent === "1" ? "取消代理" : "设置代理" }}</el-button>
          <el-button
            size="mini"
            :type="getUserSaleStyle(scope.row)"
            @click="setUserSale(scope.row)"
          >{{ scope.row.isAgent === "2" ? "取消销售" : "设置销售" }}</el-button>
          <el-button
            size="mini"
            :type="getUserMn(scope.row)"
            @click="setUserAsMoni(scope.row)"
          >{{ scope.row.moni === 1 ? "取消模拟" : "设置模拟" }}</el-button>
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

    <AddUserDialog
      ref="addUser"
      :visible.sync="dialogAddVisible"
      title="添加用户"
      @confirm="addUserReq"
    />

    <update-user-pwd-dialog
      :visible.sync="dialogUpdatePwdVisible"
      title="修改密码"
      @confirm="updatePwdReq"
    ></update-user-pwd-dialog>

    <verify-pwd-dialog :visible.sync="dialogVerifyPwdVisible" title="交易密码确认" @confirm="verifyPwd"></verify-pwd-dialog>

    <RechargeDialog
      :visible.sync="dialogRechargeVisible"
      title="充值设置"
      :nickname="updateUser.nickname"
      @confirm="rechargeReq"
    ></RechargeDialog>
    <el-dialog title="大神榜设置" :visible.sync="addGodFlag" width="400px" append-to-body>
      <el-form
        :model="addValidateForm"
        ref="addValidateForm"
        label-width="80px"
        class="demo-ruleForm"
      >
        <el-form-item label="用户昵称" prop="name">
          <el-input v-model="addValidateForm.name" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item
          label="序号"
          prop="sort"
          :rules="[
      { required: true, message: '排序不能为空'},
      { type: 'number', message: '序号必须为数字值'}
    ]"
        >
          <el-input v-model.number="addValidateForm.sort" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="addGodFlag = false">取 消</el-button>
        <el-button type="primary" @click="savePerson">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  getUserList,
  addUser,
  deleteUser,
  updateUser,
  verifyUserPwd,
  rechargeUser,
  updateUserPwd,
  setUserAsMoni,
  setDS
} from "@/api/client";
import AddUserDialog from "./components/AddUserDialog.vue";
import UpdateUserPwdDialog from "./components/UpdateUserPwdDialog.vue";
import { deepClone } from "@/utils";
import { Message, MessageBox } from "element-ui";
import VerifyPwdDialog from "./components/VerifyPwdDialog.vue";
import RechargeDialog from "./components/RechargeDialog.vue";

export default {
  name: "ClientManager",
  components: {
    AddUserDialog,
    UpdateUserPwdDialog,
    VerifyPwdDialog,
    RechargeDialog
  },
  props: {},
  data() {
    return {
      addValidateForm: {
        uid: null,
        sort: null,
        name: ""
      },
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 用户列表
      voList: [],
      groupList: [],
      // 遮罩层
      loading: true,
      // 用户查询
      queryParams: {
        pageNo: 1,
        pageSize: 10,
        nickname: undefined,
        phone: undefined,
        pid: undefined
      },
      // 添加用户弹窗
      dialogAddVisible: false,
      // 修改密码弹窗
      dialogUpdatePwdVisible: false,
      // 充值密码验证弹窗
      dialogVerifyPwdVisible: false,
      // 充值弹窗
      dialogRechargeVisible: false,
      // 当前需要修改的用户
      updateUser: {},
      addGodFlag: false
    };
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {
    this.getList();
  },
  methods: {
    addGodList(row) {
      if (row.dsSort == null) {
        this.addGodFlag = true;
        this.addValidateForm.name = row.nickname;
        this.addValidateForm.uid = row.id;
      } else {
        this.addValidateForm.uid = row.id;
        this.savePerson();
      }
    },
    async savePerson() {
      const res = await setDS(this.addValidateForm);
      if (res.success) {
        this.addGodFlag = false;
        this.getList();
        this.$refs.addValidateForm.resetFields();
      } else {
        this.$error(res.errorMsg);
      }
    },
    // 获取用户列表
    getList() {
      this.loading = true;
      getUserList(this.queryParams).then(response => {
        this.loading = false;
        this.total = response.total;
        this.voList = response.voList;
      });
    },
    // 用户性别
    getUserSex(user) {
      if (user.sex === "2") {
        return "未知";
      }
      if (user.sex == "1") {
        return "女";
      }
      return "男";
    },
    // 操作按钮-》设置代理
    getUserAgentStyle(user) {
      if (user.isAgent === "1") {
        return "info";
      }
      return "warning";
    },
    // 操作按钮-》设置销售
    getUserSaleStyle(user) {
      if (user.isAgent === "2") {
        return "info";
      }
      return "warning";
    },
    // 操作按钮-》设置模拟
    getUserMn(user) {
      if (user.moni === 1) {
        return "info";
      }
      return "warning";
    },
    // 条件查询
    handleQuery() {
      this.queryParams.pageNo = 1;
      this.getList();
    },
    // 重置查询条件
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // ---------------- 用户相关操作 -----------------------
    // 新增用户
    newUser() {
      this.$refs.addUser.resetting();
      this.dialogAddVisible = true;
    },

    // 发送新增用户请求
    addUserReq(data) {
      addUser(data).then(response => {
        console.log(data, "");
        if (!response.errorCode) {
          this.getList();
        }
      });
    },

    // 启用/禁用用户
    disableUser(row) {
      let { status, isAgent } = row;
      let updateStatus = status === "1" ? "0" : "1";
      this.updateUserReq(row.id, updateStatus, isAgent);
    },

    // 设置用户代理
    setUserAgent(row) {
      let { status, isAgent } = row;
      let updateAgent = isAgent === "1" ? "0" : "1";
      this.updateUserReq(row.id, status, updateAgent);
    },
    // 设置用户销售
    setUserSale(row) {
      let { status, isAgent } = row;
      let updateAgent = isAgent === "2" ? "0" : "2";
      this.updateUserReq(row.id, status, updateAgent);
    },
    // 设置模拟用户
    setUserAsMoni(row) {
      let { moni } = row;
      console.log(moni, "moni");
      let moniType = moni == 1 ? 0 : 1;
      setUserAsMoni({ userId: row.id, moni: moniType }).then(res => {
        if (res.success) {
          this.getList();
        } else {
          this.$message.error(res.errorMsg);
        }
      });
    },

    // 更新用户信息请求
    updateUserReq(id, status, isAgent) {
      let data = {
        status: status,
        isAgent: isAgent
      };
      console.log(data);
      updateUser(id, data).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    },

    // 修改密码事件
    updateUserPwdAction(row) {
      this.updateUser = deepClone(row);
      this.dialogUpdatePwdVisible = true;
      console.log(this.updateUser);
    },

    // 修改密码请求
    updatePwdReq(data) {
      let { phone } = this.updateUser;
      let { password } = data;
      let queryParams = {
        phone: phone,
        password: password
      };
      updateUserPwd(queryParams).then(response => {
        if (!response.errorCode) {
          Message({
            message: "密码修改成功",
            type: "success"
          });
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
          deleteUser(row.id).then(response => {
            if (!response.errorCode) {
              Message({
                type: "success",
                message: "删除成功!"
              });
              this.getList();
            }
          });
        })
        .catch(() => {});
    },

    // 充值
    recharegeUser(row) {
      console.log(row.moni, "模拟");
      this.updateUser = deepClone(row);
      // this.dialogVerifyPwdVisible = true;
      this.dialogRechargeVisible = true;
      // if (row.isAgent == '0') {
      //   this.dialogVerifyPwdVisible = true;
      // } else {
      //   this.dialogRechargeVisible = true;
      // }
    },

    // 交易密码确认
    verifyPwd(form) {
      let { payPwd } = form;
      let queryParams = {
        username: this.$store.getters.username,
        payPwd: payPwd
      };
      verifyUserPwd(queryParams).then(response => {
        if (!response.errorCode) {
          this.dialogRechargeVisible = true;
        }
      });
    },

    // 充值请求
    rechargeReq(form) {
      rechargeUser(this.updateUser.id, form).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    }
  }
};
</script>

<style scoped lang="scss">
.el-button {
  margin-bottom: 5px !important;
}
</style>
