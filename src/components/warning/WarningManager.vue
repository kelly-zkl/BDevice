<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item>告警列表</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <div class="center-block">
        <el-row class="panel-body" style="margin-top: 30px;">
          <el-col :span="24" class="text-left">
            <el-input placeholder="设备名称/ID" v-model="query.deviceId"
                      style="width: 150px;margin-right: 10px;margin-top: 10px"></el-input>
            <el-select v-model="query.wrType" placeholder="告警类型"
                       style="width: 300px;margin-right: 10px;margin-top: 10px" @change="handleChange()">
              <el-option
                v-for="item in alarmTypes" :key="item.type"
                :label="item.name" :value="item.type">
              </el-option>
            </el-select>
            <el-select v-model="query.moduleID" placeholder="告警等级" style="width: 120px;margin-top: 10px">
              <el-option
                v-for="item in options" :key="item.value"
                :label="item.label" :value="item.value">
              </el-option>
            </el-select>
            <el-button type="primary" icon="search" style="width: 80px;margin-left: 10px;margin-top: 10px"
                       @click="getData()">搜索
            </el-button>
            <el-button style="width: 80px;margin-top: 10px" @click="clearData()">清除</el-button>
            <!--<el-button style="width: 120px;margin-top: 10px" type="success" @click="$router.push('warningSet')">-->
            <!--告警设置-->
            <!--</el-button>-->
          </el-col>
        </el-row>
        <el-table :data="warningList" border v-loading="listLoading" class="center-block">
          <el-table-column align="center" type="index" label="序号"
                           width="70"></el-table-column>
          <el-table-column align="left" prop="wrType" label="告警代码"
                           min-width="100" max-width="200" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" :formatter="formatterType" label="告警类型"
                           min-width="150" max-width="300"></el-table-column>
          <el-table-column align="left" prop="msg" label="告警详情"
                           min-width="150" max-width="300" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" label="告警等级" max-width="200" min-width="100"
                           :formatter="formatterLevel"></el-table-column>
          <el-table-column align="left" prop="deviceId" label="目标设备（ID）" min-width="140"
                           max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="model" label="时间" :formatter="formatTime" min-width="150"
                           max-width="200"></el-table-column>
        </el-table>
        <div class="block" style="margin-top: 20px">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="pageChange"
            :current-page="query.page"
            :page-sizes="[10, 15, 20, 30]"
            :page-size="query.size"
            layout="total, sizes, prev, pager, next, jumper"
            :total="count"></el-pagination>
        </div>
      </div>
    </section>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        warningList: [],
        listLoading: false,
        count: 0,
        query: {
          page: 1,
          size: 10,
          moduleID: 0,
          wrType: 0,
          deviceId: ''
        },
        alarmTypes: [{type: 0, name: "全部类型"}],
        options: [{
          value: 0,
          label: '全部等级'
        }, {
          value: 255,
          label: '设备级'
        }, {
          value: 12,
          label: '子模块级'
        }]
      }
    },
    methods: {
      pageChange(index) {
        this.query.page = index;
        this.getData();
      },
      handleSizeChange(val) {
        this.query.size = val;
        this.getData();
      },
      handleChange(val) {
//        this.alarmType = val;
      },
      clearData() {
        this.query = {
          page: 1,
          size: 10,
          moduleID: 0,
          wrType: 0,
          deviceId: ''
        };
        this.getData();
      },
      getData() {
        if (this.query.deviceId) {
          if (/[+@#><!?，。》《、；‘’;'`~\$%\^&\*]+/g.test(this.query.deviceId)) {
            this.$message.error('请输入正确的设备名称/ID');
            return;
          }
        }
        this.listLoading = true;
        this.$post("alarm/get/list", this.query).then((data) => {
          this.warningList = data.data.list;
          this.count = data.data.count;
          setTimeout(() => {
            this.listLoading = false
          }, 500);
        }).catch((err) => {
          this.listLoading = false;
          this.warningList = [];
          this.$message.error(err);
        });
      },
      formatterLevel(row, column) {
        return row.moduleID && row.moduleID === 255 ? "设备级" : "子模块级";
      },
      //格式化内容   有数据就展示，没有数据就显示--
      formatterAddress(row, column) {
        return row[column.property] && row[column.property] !== "null" ? row[column.property] : '--';
      },
      formatterType(row, column) {
        let nType = "--";
        if (this.alarmTypes && this.alarmTypes.length > 0) {
          for (let item of this.alarmTypes) {
            if (row.wrType == item.type) {
              nType = item.name;
              break;
            }
          }
        }
        return nType;
      },
      //获取告警类型
      getAlarmType() {
        this.$post('alarm/get/alarmType').then(
          (data) => {
            if (data.code === '000000' && data.data) {
              this.alarmTypes = data.data;
              this.alarmTypes.unshift({type: 0, name: "全部类型"});
            }
          });
      },
      formatTime(row, column) {
        return row.time ? this.getLocalTime(row.time) : '--';
      },
      getLocalTime(time) {
        return new Date(parseInt(time * 1000)).toLocaleString().replace(/年|月/g, "-").replace(/日/g, " ");
      }
    },
    mounted() {
      sessionStorage.setItem("active", 3);
      this.getData();
      this.getAlarmType();
    }
  }
</script>
