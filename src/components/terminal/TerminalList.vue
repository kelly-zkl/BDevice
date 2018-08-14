<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item>侦码数据</el-breadcrumb-item>
      </el-breadcrumb>
    </section>

    <section class="content">
      <el-row class="panel-body" style="margin-top: 30px">
        <el-col :span="24" class="text-left">
          <el-input placeholder="设备名称/ID" v-model="query.deviceId"
                    style="width: 160px;margin-right: 10px;margin-top: 10px" :maxlength="30"></el-input>
          <el-input placeholder="终端imsi" v-model="query.imsi"
                    style="width: 160px;margin-right: 10px;margin-top: 10px" :maxlength="30"></el-input>
          <el-input placeholder="网络类型" v-model="query.netType" :maxlength="10"
                    style="width: 160px;margin-right: 10px;margin-top: 10px"></el-input>
          <el-select v-model="type" placeholder="全部名单"
                     style="width: 110px;margin-right: 10px;margin-top: 10px">
            <el-option
              v-for="item in options1" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          <el-select v-model="query.isp" placeholder="全部运营商" style="width: 150px;margin-right: 10px;margin-top: 10px">
            <el-option
              v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
          <el-button type="primary" icon="search" style="width: 80px;margin-left: 10px;margin-top: 10px"
                     @click="getData()">搜索
          </el-button>
          <el-button style="width: 80px;margin-top: 10px" @click="clearData()">清除</el-button>
          <label style="margin:10px 15px 0 15px">{{count}}条</label>
          <el-button style="width: 120px;margin-top: 10px" type="success" @click="$router.push('blackManager')">
            黑名单管理
          </el-button>
          <el-button style="width: 120px;margin-top: 10px" type="success" @click="$router.push('whiteManager')">
            白名单管理
          </el-button>
        </el-col>
      </el-row>
      <el-table :data="list" v-loading="listLoading" border class="center-block">
        <el-table-column align="left" type="index" label="序号"
                         width="65"></el-table-column>
        <el-table-column align="left" prop="deviceId" label="设备ID"
                         min-width="100" max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="imei" label="终端imei"
                         min-width="100" max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="imsi" label="终端imsi"
                         min-width="150" max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="tmsi" label="终端tmsi"
                         min-width="100" max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="uptime" label="抓取时间" width="200"
                         :formatter="formatTime"></el-table-column>
        <el-table-column align="left" prop="isp" label="运营商" max-width="150" min-width="100"
                         :formatter="formatOperator"></el-table-column>
        <el-table-column align="left" prop="netType" label="网络类型" max-width="150" min-width="100"></el-table-column>
        <el-table-column align="left" label="归属地" max-width="150" min-width="100"
                         :formatter="formatArea"></el-table-column>
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
    </section>
  </div>
</template>

<script>
  import json from '../../assets/city.json';

  export default {
    data() {
      return {
        fileList: [],
        provinceList: json,
        count: 0,
        listLoading: false,
        list: [],
        query: {
          page: 1,
          size: 10,
          isp: '',
          deviceId: this.$route.query.deviceId || '',
          netType: '',
          imsi: ''
        },
        options: [{
          value: '2',
          label: '电信'
        }, {
          value: '0',
          label: '移动'
        }, {
          value: '1',
          label: '联通'
        }],
        options1: [{
          value: 'isBlack',
          label: '黑名单'
        }, {
          value: 'isWhite',
          label: '白名单'
        }],
        type: ''
      }
    },
    methods: {
      formatTime(row, column) {
        return row.uptime ? this.getLocalTime(row.uptime) : '--';
      },
      formatOperator(row, column) {
        return row.isp === 0 ? '移动' : row.isp === 1 ? '联通' : row.isp === 2 ? '电信' : '未知';
      },
      //格式化内容   有数据就展示，没有数据就显示--
      formatterAddress(row, column) {
        return row[column.property] && row[column.property] !== "null" ? row[column.property] : '--';
      },
      formatArea(row, column) {
        let str = '--';
        if (row.areaCode) {
          this.provinceList.forEach((province) => {
            if (province.s) {
              province.s.forEach((city) => {
                if (row.areaCode === city.id) {
                  str = province.n + '  ' + city.n;
                }
              })
            }
          });
        }
        return str;
      },
      clearType() {
        let type = ['isBlack', 'isWhite'];
        for (let item of type) {
          this.query[item] = undefined;
        }
      },
      getData() {
        if (this.query.deviceId) {
          if (/[+@#><!?，。》《、；‘’;'`~\$%\^&\*]+/g.test(this.query.deviceId)) {
            this.$message.error('请输入正确的设备设备名称/ID');
            return;
          }
        }
        if (this.query.imsi) {
          if (!/[0-9]$/.test(this.query.imsi)) {
            this.$message.error('请输入正确的imsi');
            return;
          }
        }
        if (this.query.netType) {
          if (!/[a-zA-Z0-9]$/.test(this.query.netType)) {
            this.$message.error('请输入正确的网络类型');
            return;
          }
        }
        this.clearType();
        if (this.type) {
          this.query[this.type] = true;
        }
        this.listLoading = true;
        this.$post("terminate/query", this.query).then((data) => {
          this.count = data.data.count;
          this.list = data.data.list;
          setTimeout(() => {
            this.listLoading = false
          }, 500);
        }).catch((err) => {
          this.listLoading = false;
          this.list = [];
          this.$message.error(err);
        });
      },
      pageChange(index) {
        this.query.page = index;
        this.getData();
      },
      handleSizeChange(val) {
        this.query.size = val;
        this.getData();
      },
      clearData() {
        this.query = {
          page: 1,
          size: 10,
          isp: '',
          deviceId: '',
          netType: '',
          imsi: ''
        };
        this.type = '';
        this.clearType();
        this.getData();
      },
      getLocalTime(nS) {
        return new Date(parseInt(nS)).toLocaleString().replace(/年|月/g, "-").replace(/日/g, " ");
      }
    },
    mounted() {
      sessionStorage.setItem("active", 2);
      this.getData();
    }
  }
</script>
