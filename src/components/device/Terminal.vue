<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/deviceList' }">设备列表</el-breadcrumb-item>
        <el-breadcrumb-item>侦码数据</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <hr/>
      <el-form label-width="150px" label-position="left">
        <el-row style="margin-left: 20px">
          <el-col :span="11" style="text-align: left">
            <h4>侦码数据 <span style="margin-left: 20px;font-size: 15px;color: #888">设备ID：{{deviceId}}</span></h4>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="13" style="margin-top: 20px;margin-left: 50px">
            <el-form-item label="侦码">
              <el-row>
                <el-col :span="4" style="text-align: center">{{criminalCodeSwitch ? "已开启" : "已关闭"}}</el-col>
                <el-col :span="4" :offset="1">
                  <el-switch v-model="criminalCodeSwitch" on-color="#13ce66" off-color="#ff4949"
                             @change="setCriminalCodeSwitch()"></el-switch>
                </el-col>
              </el-row>
            </el-form-item>
            <!--<el-form-item label="数据上报">-->
            <!--<el-row>-->
            <!--<el-col :span="4" style="text-align: center">{{dataUploadSwitch ? "已开启" : "已关闭"}}-->
            <!--</el-col>-->
            <!--<el-col :span="4" :offset="1">-->
            <!--<el-switch v-model="dataUploadSwitch" on-color="#13ce66" off-color="#ff4949"-->
            <!--@change="setDataUploadSwitch()"></el-switch>-->
            <!--</el-col>-->
            <!--</el-row>-->
            <!--</el-form-item>-->
            <el-form-item label="上报时间间隔">
              <el-row>
                <el-col :span="4">
                  <el-select v-model="uploadPeriod" placeholder="请选择">
                    <el-option v-for="item in periods" :key="item.value" :label="item.name" :value="item.value">
                    </el-option>
                  </el-select>
                </el-col>
                <el-col :span="2">
                  <label style="width: 100%">秒</label>
                </el-col>

                <el-col :span="2">
                  <el-button type="success" style="width: 120px" @click="setTerminateUploadPeriod">设置</el-button>
                </el-col>
              </el-row>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <hr/>

      <el-table :data="list" v-loading="listLoading" border class="center-block" style="margin-left: 20px">
        <el-table-column align="left" type="index" label="序号" width="70"></el-table-column>
        <el-table-column align="left" prop="deviceId" label="设备ID"
                         max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="imei" label="终端imei"
                         max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="imsi" label="终端imsi"
                         max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="tmsi" label="终端tmsi"
                         max-width="200" :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" prop="isp" label="运营商" max-width="100"
                         :formatter="formatOperator"></el-table-column>
        <el-table-column align="left" prop="uptime" label="抓取时间" max-width="150"
                         :formatter="formatTime"></el-table-column>
        <el-table-column align="left" prop="netType" label="网络类型" max-width="100"
                         :formatter="formatterAddress"></el-table-column>
        <el-table-column align="left" label="归属地" max-width="150"
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
//    props: ['deviceId'],
    data() {
      return {
        deviceId: this.$route.query.deviceId || '',
//        deviceId1: this.deviceId,
        dataUploadSwitch: false,
        criminalCodeSwitch: false,
        uploadPeriod: 6,
        count: 0,
        listLoading: false,
        list: [],
        query: {
          page: 1,
          size: 10,
          isp: '',
          deviceId: this.deviceId,
          imsi: ''
        },
        periods: [
          {
            value: '3',
            name: '3'
          },
          {
            value: '6',
            name: '6'
          },
          {
            value: '10',
            name: '10'
          }
        ],
        options: [{
          value: '',
          label: '全部'
        }, {
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
          value: '',
          label: '全部'
        }, {
          value: 'isBlack',
          label: '黑名单'
        }
//          ,{
//          value: 'isWhite',
//          label: '白名单'
//        }
        ],
        type: ''
      }
    },
//    watch: {
//      deviceId: function () {
//        this.deviceId1 = this.deviceId;
//        this.dataUploadSwitch = false;
//        this.criminalCodeSwitch = false;
//        this.uploadPeriod = 6;
//        this.getData();
//        this.getDevice();
//        this.query.deviceId = this.deviceId;
//      }
//    },
//    created() {
//      this.deviceId = this.$route.query.deviceId || '';
//      this.deviceId1 = this.deviceId;
//      this.getData();
//      this.getDevice();
//      this.query.deviceId = this.deviceId;
//    },
    methods: {
      formatTime(row, column) {
        return row.uptime ? this.getLocalTime(row.uptime) : '--';
      },
      formatOperator(row, column) {
        return row.isp === 0 ? '移动' : row.isp === 1 ? '联通' : row.isp === 2 ? '电信' : '未知';
      },
      formatArea(row, column) {
        let str = '--';
        if (row.areaCode) {
          json.forEach((province) => {
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
      getData() {
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
          deviceId: this.deviceId,
          imsi: ''
        };
        this.type = '';
        this.getData();
      },
      getLocalTime(nS) {
        return new Date(parseInt(nS)).toLocaleString().replace(/年|月/g, "-").replace(/日/g, " ");
      },
      //格式化内容   有数据就展示，没有数据就显示--
      formatterAddress(row, column) {
        return row[column.property] && row[column.property] !== "null" ? row[column.property] : '--';
      },
      //数据上报开关
      setDataUploadSwitch() {
        this.$post("device/update/dataUploadSwitch/" + this.deviceId + '/' + this.dataUploadSwitch, {}, '设置成功', '设置失败').then((data) => {
          //this.getDevice();
        })
      },
      //侦码开关
      setCriminalCodeSwitch() {
        this.$post("device/update/criminalCodeSwitch/" + this.deviceId + '/' + this.criminalCodeSwitch, {}, '设置成功', '设置失败').then((data) => {
          //this.getDevice();
        })
      },
      setTerminateUploadPeriod() {
        this.$post("set/terUploadPeriod/" + this.deviceId, {dataUpCycSeconds: this.uploadPeriod}, '设置成功', '设置失败').then((data) => {
          //this.getDevice();
        })
      },
      getDevice() {
//        this.device = {};
        this.$post('device/get/byDeviceId/' + this.deviceId).then(
          (data) => {
            if (data.code === '000000') {
              if (data.data) {
                var device = data.data;
                this.dataUploadSwitch = device.isPauseDataUpload;
                this.criminalCodeSwitch = device.isCriminalCode;
                if (device.terUploadPeriod) {
                  this.uploadPeriod = device.terUploadPeriod.dataUpCycSeconds;
                }
              }
            }
          })
      }
    },
    mounted() {
      this.deviceId = this.$route.query.deviceId || '';
      this.query.deviceId = this.deviceId;
      sessionStorage.setItem("active", 1);
      this.getData();
      this.getDevice();
    }
  }
</script>
