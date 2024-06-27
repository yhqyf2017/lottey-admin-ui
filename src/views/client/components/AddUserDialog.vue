<template>
  <div>
    <el-dialog v-bind="$attrs" v-on="$listeners" :width="widthFlag ? '90%' : '500'" @open="onOpen" @close="onClose">
      <el-row :gutter="10">
        <el-form :model="form" ref="addUserForm" :rules="rules" size="medium" label-width="100px">
          <el-col :span="24" v-if="form.isAgent === '2'">
            <el-form-item label="上級">
              <el-select v-model="form.pid" placeholder="请选择上级">
                <el-option
                  v-for="item in groupList"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="是否销售">
              <el-radio v-model="form.isAgent" label="2">是</el-radio>
              <el-radio v-model="form.isAgent" label="0">否</el-radio>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="手机号" prop="phone">
              <el-input
                v-model="form.phone"
                placeholder="请输入手机号"
                maxlength="11"
                oninput="this.value = this.value.replace(/[^0-9]/g, '');"
              ></el-input>
            </el-form-item>
          </el-col>

          <el-col :span="24">
            <el-form-item label="密码" prop="password">
              <el-input v-model="form.password" type="password" placeholder="请输入密码"></el-input>
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>

      <div slot="footer">
        <el-button @click="close">取 消</el-button>
        <el-button type="primary" @click="handleConfirm">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { validPhone } from "@/utils/validate";
import { shopkeeperList } from "@/api/client";
export default {
  name: "AddUserDialog",
  components: {},
  props: {},
  data() {
    const validatePhone = (rule, value, callback) => {
      if (!validPhone(value)) {
        callback(new Error("请输入正确的手机号"));
      } else {
        callback();
      }
    };

    return {
      widthFlag:false,
      form: {
        phone: undefined,
        password: undefined,
        isAgent: "0",
        pid: null
      },
      groupList: [],
      rules: {
        phone: [
          {
            required: true,
            trigger: "blur",
            validator: validatePhone
          }
        ],
        password: [
          {
            required: true,
            message: "请输入密码",
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
    this.getshopkeeperList();
    let screenWidth = window.innerWidth;
    screenWidth <= 750 ? this.widthFlag = true : false
  },
  methods: {
    // 重置
    resetting() {
      this.form = {
        phone: undefined,
        password: undefined,
        isAgent: "0",
        pid: null
      };
    },
    // 获取店主
    getshopkeeperList() {
      shopkeeperList({ nickname: "" }).then(res => {
        res.voList.forEach(el => {
          this.groupList.push({
            label: el.nickname,
            value: el.id
          });
        });
        console.log(this.groupList, "res");
      });
    },
    onOpen() {},
    onClose() {},
    close() {
      this.resetForm("addUserForm");
      this.$emit("update:visible", false);
    },
    handleConfirm() {
      this.$refs.addUserForm.validate(valid => {
        if (!valid) return;
        this.$emit("confirm", { ...this.form });
        this.close();
      });
    }
  }
};
</script>

<style scoped lang="scss"></style>
