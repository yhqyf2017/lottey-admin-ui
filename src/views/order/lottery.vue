<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" size="mini">
      <el-form-item label="订单号" prop="orderId">
        <el-input
          v-model="queryParams.orderId"
          placeholder="请输入订单号"
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
      <el-form-item label="订单状态" prop="state">
        <el-select v-model="queryParams.state" placeholder="订单状态" clearable>
          <el-option
            v-for="item in stateOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </el-form-item>

      <el-form-item label="赛事类型" prop="type">
        <el-select v-model="queryParams.type" placeholder="赛事类型" clearable>
          <el-option
            v-for="item in typeOptions"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <!-- <el-col :span="1.5">
        <el-button
          type="danger"
          icon="el-icon-receiving"
          size="mini"
          plain
          @click="deleteOrder"
        >一键清单</el-button>
      </el-col> -->

      <!-- <el-col :span="1.5">
        <el-button
          type="warning"
          icon="el-icon-takeaway-box"
          size="mini"
          plain
          @click="ticketAll"
        >
          一键出票
        </el-button>
      </el-col>-->

      <el-col :span="1.5">
        <el-button type="info" icon="el-icon-sort" size="mini" plain @click="toggleExpandAll">展开/折叠</el-button>
      </el-col>

      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table
      ref="table"
      v-if="refreshTable"
      v-loading="loading"
      :data="voList"
      border
      stripe
      row-key="id"
      :default-expand-all="isExpand"
    >
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-card>
            <el-form label-position="left" inline class="demo-table-expand">
              <el-row :gutter="5">
                <el-col :span="12">
                  <el-form-item label="订单号：">
                    <span>{{ scope.row.orderId }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="用户名：">
                    <span>{{ scope.row.nickname }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="上级：">
                    <span>{{ scope.row.parentName }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="订单状态：">
                    <span :class="getAward(scope.row)">
                      {{
                      getOrderState(scope.row)
                      }}
                    </span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="下注金额：">
                    <span>{{ scope.row.price }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="预计奖金：">
                    <span>{{ scope.row.forecast }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="中奖金额：">
                    <span :class="getAward(scope.row)">{{ scope.row.winPrice }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item>
                    <el-tag size="mini" type="success">
                      {{
                      getBettingNotes(scope.row)
                      }}
                    </el-tag>
                    <el-tag class="ml5" size="mini" type="warning">
                      {{
                      getBettingTime(scope.row)
                      }}
                    </el-tag>
                    <template v-if="isSportRace(scope.row)">
                      <el-tag
                        v-for="item in getBettingStyle(scope.row)"
                        :key="item"
                        class="ml5"
                        size="mini"
                        type="danger"
                      >{{ item }}</el-tag>
                    </template>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="创建时间：">
                    <span>{{ parseTime(scope.row.createTime) }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="出票时间：">
                    <span>{{ parseTime(scope.row.ticketingTime) }}</span>
                  </el-form-item>
                </el-col>
                <el-col :span="24">
                  <template v-if="isSportRace(scope.row)">
                    <el-table :data="scope.row.racingBallList" border stripe>
                      <el-table-column label="编号" width="80" align="center">
                        <template slot-scope="inner">{{ inner.row.content.number }}</template>
                      </el-table-column>
                      <el-table-column label="比赛队伍" align="center">
                        <template slot-scope="inner">
                          <span>{{ inner.row.content.homeTeam }}</span>
                          <span :class="getRaceletBallStyle(inner.row)">
                            {{
                            getRaceletBall(inner.row)
                            }}
                          </span>
                          VS
                          <span>{{ inner.row.content.visitingTeam }}</span>
                        </template>
                      </el-table-column>
                      <el-table-column label="下注内容" align="center">
                        <template slot-scope="inner">{{ getRaceContent(inner.row) }}</template>
                      </el-table-column>
                      <el-table-column label="赛果(全/半)" align="center">
                        <template slot-scope="inner">{{ getRaceResult(inner.row) }}</template>
                      </el-table-column>
                    </el-table>
                  </template>
                  <template v-else>
                    <el-table :data="scope.row.racingBallList" border stripe>
                      <el-table-column label="期号" prop="no" width="80" align="center"></el-table-column>
                      <el-table-column label="投注方式" align="center">
                        <template slot-scope="inner">
                          <span>{{ getNoSportsBettingStyle(inner.row) }}</span>
                        </template>
                      </el-table-column>
                      <el-table-column label="下注内容" prop="content" align="center"></el-table-column>
                      <el-table-column label="赛果" prop="reward" align="center"></el-table-column>
                    </el-table>
                  </template>
                </el-col>
                <el-col :span="24">
                  <el-form-item>
                    <template v-if="scope.row.state === '0'">
                      <el-button size="mini" type="success" @click="ticketSigle(scope.row)">出票</el-button>
                      <el-button size="mini" type="warning" @click="refuseSigle(scope.row)">拒绝</el-button>
                    </template>
                    <el-button size="mini" type="danger" @click="retreatSigle(scope.row)">退票</el-button>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </el-card>
        </template>
      </el-table-column>
      <el-table-column
        label="昵称"
        align="center"
        prop="nickname"
        width="100"
        :show-overflow-tooltip="true"
      />
      <el-table-column label="中奖金额" align="center" width="80">
        <template slot-scope="scope">
          <span :class="getAward(scope.row)">{{ scope.row.winPrice }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="下注金额"
        align="center"
        prop="price"
        width="100"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="预测金额"
        align="center"
        prop="forecast"
        width="100"
        :show-overflow-tooltip="true"
      />
      <el-table-column label="订单状态" align="center" width="80" :show-overflow-tooltip="true">
        <template slot-scope="scope">
          <span :class="getAward(scope.row)">{{ getOrderState(scope.row) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="赛事类型" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ getMatchType(scope.row) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center" show-overflow-tooltip>
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新时间" align="center" show-overflow-tooltip>
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.updateTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="出票时间" align="center" show-overflow-tooltip>
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.ticketingTime) }}</span>
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
  </div>
</template>

<script>
import {
  getLottryOrderList,
  orderAward,
  orderTicketing,
  orderRetreat,
  deleteOrder
} from "@/api/order";
import { removeUser } from "@/utils/auth";

export default {
  name: "OrderLottery",
  components: {},
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
        orderId: undefined,
        state: undefined,
        type: undefined,
        phone: undefined,
        pageNo: 1,
        pageSize: 10
      },
      // 是否显示搜索
      showSearch: true,
      // 是否全部展开
      isExpand: true,
      // 重新渲染表格状态
      refreshTable: true,
      // 订单状态
      stateOptions: [
        {
          label: "待出票",
          value: "0"
        },
        {
          label: "待开奖",
          value: "1"
        },
        {
          label: "未中奖",
          value: "2"
        },
        {
          label: "待派奖",
          value: "3"
        },
        {
          label: "已派奖",
          value: "4"
        },
        {
          label: "已拒绝",
          value: "5"
        },
        {
          label: "已退票",
          value: "6"
        }
      ],
      // 赛事类型
      typeOptions: [
        {
          label: "竞彩足球",
          value: "0"
        },
        {
          label: "竞彩篮球",
          value: "1"
        },
        {
          label: "北京单场",
          value: "2"
        },
        {
          label: "排列3",
          value: "3"
        },
        {
          label: "排列5",
          value: "4"
        },
        {
          label: "七星彩",
          value: "5"
        },
        {
          label: "14场胜负",
          value: "6"
        },
        {
          label: "任选九",
          value: "7"
        },
        {
          label: "大乐透",
          value: "8"
        }
      ]
    };
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {
    this.getList();
  },
  methods: {
    // 获取统计列表
    getList() {
      this.loading = true;
      getLottryOrderList(this.queryParams).then(response => {
        this.loading = false;
        if (!response.errorCode) {
          this.total = response.total;
          const volist = response.voList;
          for (let index = 0; index < volist.length; index++) {
            const outter = volist[index];
            for (let i = 0; i < outter.racingBallList.length; i++) {
              const inner = outter.racingBallList[i];
              const content = inner.content;
              try {
                const contentObject = JSON.parse(content);
                inner.content = contentObject;
              } catch (error) {}
            }
          }
          this.voList = volist;
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
    // 展开/收缩
    toggleExpandAll() {
      this.refreshTable = false;
      this.isExpand = !this.isExpand;
      this.$nextTick(() => {
        this.refreshTable = true;
      });
    },
    // ---------------- 比赛 -----------------------
    // 根据比赛显示数据
    // 0 足彩 1 篮彩 2 北京单场 3 排列3 4排列5  5七星彩  6 14场胜负 7任选九 8大乐透
    isSportRace(row) {
      if (
        row.type === "3" ||
        row.type === "4" ||
        row.type === "5" ||
        row.type === "8"
      ) {
        return false;
      }
      return true;
    },
    // 比赛让球数
    getRaceletBall(row) {
      // 不存在让球
      if (!row.content.letBall) {
        return "";
      }
      // 让球为0
      if (row.content.letBall === "0") {
        return "";
      }
      return "(" + row.content.letBall + ")";
    },
    // 比赛让球数样式
    getRaceletBallStyle(row) {
      const content = row.content;
      // 不存在让球
      if (!content.letBall) {
        return "";
      }
      // 让球为0
      if (content.letBall === "0") {
        return "";
      }
      let result = content.letBall.includes("+");
      return result ? "red" : "blue";
    },
    // 下注内容
    getRaceContent(row) {
      const content = row.content;
      let result = "";
      // 让球胜平负
      if (content.letOddsList && content.letOddsList.length) {
        let oddsList = [];
        for (let index = 0; index < content.letOddsList.length; index++) {
          const element = content.letOddsList[index];
          const result = "让" + element.describe + "(" + element.odds + ")";
          oddsList.push(result);
        }
        result = result + oddsList.join(",");
      }
      // 胜平负
      if (content.notLetOddsList && content.notLetOddsList.length) {
        if (result) {
          result = result + "|";
        }
        let oddsList = [];
        for (let index = 0; index < content.notLetOddsList.length; index++) {
          const element = content.notLetOddsList[index];
          const result = element.describe + "(" + element.odds + ")";
          oddsList.push(result);
        }
        result = result + oddsList.join(",");
      }
      // goalOddsList
      if (content.goalOddsList && content.goalOddsList.length) {
        if (result) {
          result = result + "|";
        }
        let oddsList = [];
        for (let index = 0; index < content.goalOddsList.length; index++) {
          const element = content.goalOddsList[index];
          const result = element.describe + "(" + element.odds + ")";
          oddsList.push(result);
        }
        result = result + oddsList.join(",");
      }
      // halfWholeOddsList
      if (content.halfWholeOddsList && content.halfWholeOddsList.length) {
        if (result) {
          result = result + "|";
        }
        let oddsList = [];
        for (let index = 0; index < content.halfWholeOddsList.length; index++) {
          const element = content.halfWholeOddsList[index];
          const result = element.describe + "(" + element.odds + ")";
          oddsList.push(result);
        }
        result = result + oddsList.join(",");
      }
      // scoreOddsList
      if (content.scoreOddsList && content.scoreOddsList.length) {
        if (result) {
          result = result + "|";
        }
        let oddsList = [];
        for (let index = 0; index < content.scoreOddsList.length; index++) {
          const element = content.scoreOddsList[index];
          const result = element.describe + "(" + element.odds + ")";
          oddsList.push(result);
        }
        result = result + oddsList.join(",");
      }
      return result;
    },
    // 赛果
    getRaceResult(row) {
      if (!row.reward) {
        return "";
      }
      let result = row.reward.split(",");
      if (result.length < 2) {
        return "";
      }
      let all = result[1];
      let half = result[0];
      return all + "\n" + "半" + half;
    },
    // ---------------- 其他格式化 -----------------------
    // 投注数量
    getBettingNotes(row) {
      if (row.racingBallList.length) {
        return row.racingBallList[0].notes + "注";
      }
      return "";
    },
    // 投注倍数
    getBettingTime(row) {
      if (row.racingBallList.length) {
        return row.racingBallList[0].times + "倍";
      }
      return "";
    },
    // 投注方式
    getBettingStyle(row) {
      const styleList = [];
      if (row.racingBallList.length && row.racingBallList[0].type) {
        let list = row.racingBallList[0].type.split(",");
        for (let index = 0; index < list.length; index++) {
          const element = list[index];
          styleList.push(element + "串一");
        }
      }
      return styleList;
    },
    // 非体育赛事投注方式
    getNoSportsBettingStyle(row) {
      // 0 直选 1 组三 2 组九
      switch (row.type) {
        case "0":
          return "直选";
        case "1":
          return "组三";
        case "2":
          return "组九";
        default:
          return "";
      }
    },
    // 中奖Style
    getAward(row) {
      if (row.state === "4") {
        return "award";
      }
      return "";
    },
    // 获取订单状态
    getOrderState(row) {
      let result = this.stateOptions.filter(item => {
        return item.value === row.state;
      });
      if (result.length === 0) {
        return "";
      }
      return result[0].label;
    },
    // 获取赛事类型
    getMatchType(row) {
      let result = this.typeOptions.filter(item => {
        return item.value === row.type;
      });
      if (result.length === 0) {
        return "";
      }
      return result[0].label;
    },
    // 一键清单
    async deleteOrder() {
      const res = deleteOrder();
      this.$message({
        message: "清除成功!",
        type: "success"
      });
      this.handleQuery()
    },
    // 单个派奖
    awardSigle(row) {},
    // 一键出票
    ticketAll() {},
    // 单个出票
    ticketSigle(row) {
      console.log(row);
      const data = {
        state: "1",
        id: row.id
      };
      orderTicketing(data).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    },
    // 单个拒绝
    refuseSigle(row) {
      const data = {
        state: "5",
        id: row.id
      };
      orderTicketing(data).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    },
    // 单个退票
    retreatSigle(row) {
      orderRetreat(row.id).then(response => {
        if (!response.errorCode) {
          this.getList();
        }
      });
    }
  }
};
</script>

<style scoped lang="scss">
::v-deep .el-table__expanded-cell[class*="cell"] {
  padding: 10px;
}
.award {
  color: red;
}
.red {
  color: red;
}
.blue {
  color: blue;
}
.demo-table-expand {
  font-size: 0;
  ::v-deep .el-form-item__label {
    padding: 0px;
  }
  .label {
    width: 90px;
    color: #99a9bf;
  }
  .el-form-item {
    margin-right: 0;
    margin-bottom: 0;
    width: 50%;
  }
}
</style>
