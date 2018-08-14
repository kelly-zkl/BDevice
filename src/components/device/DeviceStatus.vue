<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/deviceList' }">设备列表</el-breadcrumb-item>
        <el-breadcrumb-item>设备状态</el-breadcrumb-item>
      </el-breadcrumb>
    </section>

    <hr/>

    <section class="content">
      <el-form label-width="150px" label-position="left">
        <el-row style="margin-left: 20px">
          <el-col :span="11" style="text-align: left">
            <h4>状态上报设置 <span style="margin-left: 20px;font-size: 15px;color: #888">设备ID：{{deviceId}}</span></h4>
          </el-col>
        </el-row>
        <el-row style="margin-top: 20px;margin-left: 50px">
          <el-col :span="12">
            <el-form-item label="实时运行状态上报">
              <el-row>
                <el-col :span="4">{{statusSwitch ? "已开启" : "已关闭"}}</el-col>
                <el-col :span="2" :offset="2">
                  <el-switch v-model="statusSwitch" on-color="#13ce66" off-color="#ff4949"
                             @change="setStatusSwitch()"></el-switch>
                </el-col>
              </el-row>
            </el-form-item>
            <!--<el-form-item label="上报时间间隔">-->
            <!--<el-row>-->
            <!--<el-col :span="4">-->
            <!--<el-select v-model="uploadPeriod" placeholder="请选择">-->
            <!--<el-option v-for="item in periods" :key="item.value" :label="item.name" :value="item.value">-->
            <!--</el-option>-->
            <!--</el-select>-->
            <!--</el-col>-->
            <!--<el-col :span="2">-->
            <!--<label style="width: 100%">秒</label>-->
            <!--</el-col>-->

            <!--<el-col :span="2">-->
            <!--<el-button type="success" style="width: 120px" @click="setTerminateUploadPeriod">设置</el-button>-->
            <!--</el-col>-->
            <!--</el-row>-->
            <!--</el-form-item>-->
            <el-form-item label="回传速率">
              <el-row>
                <el-col :span="4">{{retrievalRate1}}秒</el-col>
                <el-col :span="4" :offset="1">
                  <el-button type="success" style="width: 100%" @click="diagnose" :loading="logining" disabled>诊断
                  </el-button>
                </el-col>
              </el-row>
            </el-form-item>
          </el-col>
        </el-row>

        <br/>

        <el-row>
          <el-col :span="11" style="text-align: left;margin-left: 20px">
            <h4>设备状态</h4>
          </el-col>
        </el-row>
        <el-row style="margin-top: 20px;margin-left: 50px">
          <el-col :span="6">
            <el-form-item label="温度："><span v-bind:class="temperatureStyle">{{deviceStatus.temperature}}°C</span>
            </el-form-item>
            <el-form-item label="cpu占用率："><span v-bind:class="temperatureStyle">{{deviceStatus.cpu}}%</span>
            </el-form-item>
            <el-form-item label="内存占用率："><span v-bind:class="temperatureStyle">{{deviceStatus.memory}}%</span>
            </el-form-item>
          </el-col>
          <el-col :span="6" :offset="2">
            <el-form-item label="抓取记录："><span v-bind:class="temperatureStyle">{{deviceStatus.catchNum}}条</span>
            </el-form-item>
            <el-form-item label="持续工作时间："><span v-bind:class="temperatureStyle">{{deviceStatus.workDuration}}秒</span>
            </el-form-item>
            <el-form-item label="侦码状态："><span v-bind:class="temperatureStyle">上报，{{deviceStatus.criminalCodeStatus}}/min(单路)</span>
            </el-form-item>
          </el-col>
        </el-row>

        <br/>

        <el-row>
          <el-col :span="11" style="text-align: left;margin-left: 20px">
            <h4>子模块状态</h4>
          </el-col>
        </el-row>
        <br/>
        <el-tabs v-model="activeItem" @tab-click="handleClick" type="card" style="margin-left: 50px">
          <el-tab-pane v-bind:label="tab.name" v-for="tab in activeName"
                       :key="tab.type" v-bind:name="tab.type" :value="tab.moduleID">
            <el-row style="margin-top: 20px">
              <el-col :span="6">
                <el-form-item label="PA温度："><span v-bind:class="temperatureStyle">{{patemp}}°C</span></el-form-item>
                <el-form-item label="PA开关："><span
                  v-bind:class="temperatureStyle">{{paSwitch === 1 ? "已开启" : "已关闭"}}</span></el-form-item>
                <el-form-item label="无线电开关："><span
                  v-bind:class="temperatureStyle">{{wirelessSwitch === 1 ? "已开启" : "已关闭"}}</span>
                </el-form-item>
              </el-col>
              <el-col :span="6" :offset="2">
                <el-form-item label="输出功率："><span v-bind:class="temperatureStyle">{{deviceStatus.power}}mW</span>
                </el-form-item>
                <el-form-item label="驻波比："><span
                  v-bind:class="temperatureStyle">{{deviceStatus.stationedInBobbi}}</span>
                </el-form-item>
              </el-col>
            </el-row>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </section>
  </div>
</template>
<script>
  export default {
//    props: ['deviceId', "closeFlag"],
    data() {
      return {
        patemp: 0,
        paSwitch: 0,
        wirelessSwitch: 0,
        deviceStatus: {},
        uploadPeriod: 6,
        subModuleStatus: {},
        statusSwitch: true,
        logining: false,
        retrievalRate1: '',
        activeItem: "GSM1",
        activeName: [{
          moduleID: 1,
          name: '移动2G',
          type: 'GSM1'
        }, {
          moduleID: 2,
          name: '联通2G',
          type: 'GSM2'
        }, {
          moduleID: 3,
          name: '联通4G',
          type: 'FDD0'
        }, {
          moduleID: 4,
          name: '电信4G',
          type: 'FDD1'
        }, {
          moduleID: 5,
          name: '移动4G-38/40',
          type: 'TDD0'
        }, {
          moduleID: 6,
          name: '移动4G-39',
          type: 'TDD1'
        }],
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
        deviceId: this.$route.query.deviceId || '',
        intervalid: null
      }
    },
//    watch: {
//      deviceId: function () {
//        this.deviceId1 = this.deviceId;
//        this.getDevice();
//        this.statusTask();
//      }
//    },
//    created() {
//      this.deviceId = this.$route.query.deviceId || '';
//      this.deviceId1 = this.deviceId;
//      this.getDevice();
//      this.statusTask();
//    },
    beforeDestroy() {
      clearInterval(this.intervalid);
    },
    computed: {
      temperatureStyle() {
        return {
          'text-green': this.deviceStatus.temperature < 50 ? true : false,
          'text-yellow': (this.deviceStatus.temperature > 50 && this.deviceStatus.temperature < 70) ? true : false,
          'text-red': this.deviceStatus.temperature > 70 ? true : false
        }
      }
    },

    methods: {
      //定时刷新
      statusTask() {
        if (!this.intervalid) {
          this.intervalid = setInterval(() => {
            this.getDevice();
          }, 5000);
        }
      },
      handleClick(tab, event) {
        this.getSubStatus();
      },
      //关闭定时刷新
      dialogCloseHandle() {
        clearInterval(this.intervalid);
      },
      getSubStatus() {
        this.subModuleStatus = {};
        if (this.deviceStatus.subModuleDataList && this.deviceStatus.subModuleDataList.length > 0) {
          for (let item of this.deviceStatus.subModuleDataList) {
            if (item.moduleId === this.getModuleID()) {
              this.subModuleStatus = item;
              this.patemp = item.patemp;
              this.paSwitch = item.paSwitch;
              this.wirelessSwitch = item.wirelessSwitch;
            }
          }
        }
      },
      //设置上报时间间隔
      setTerminateUploadPeriod() {
        this.$post("set/terUploadPeriod/" + this.deviceId, {dataUpCycSeconds: this.uploadPeriod}, '设置成功', '设置失败').then((data) => {
          //this.getDevice();
        })
      },
      getModuleID() {
        for (let item of this.activeName) {
          if (this.activeItem === item.type) {
            return item.moduleID;
          }
        }
      },
      //诊断
      diagnose() {
        this.logining = true;
        this.$post('cmd/deliverGetRate/' + this.deviceId, {}, '诊断命令下发成功', '诊断命令下发失败').then((data) => {
          if (data.code === '000000') {
            this.$emit('subRate', false);
          }
          setTimeout(() => {
            this.logining = false
          }, 5000);
        })
      },
      getDeviceStatus() {
        this.deviceStatus = {};
        this.$post('device/get/deviceStatus/' + this.deviceId + '/' + this.activeItem, {}).then(
          (data) => {
            if (data.code === '000000') {
              if (data.data) {
                this.deviceStatus = data.data;
                this.deviceStatus.cpu = parseInt(parseFloat(this.deviceStatus.cpu));
                this.deviceStatus.memory = parseInt(parseFloat(this.deviceStatus.memory));
              }
            }
          });
        this.getSubStatus();
      },
      //设置状态开关
      setStatusSwitch() {
        this.$post("device/update/statusSwitch/" + this.deviceId + '/' + this.statusSwitch, {}, '设置成功', '设置失败');
      },
      getDevice() {
        this.$post('device/get/byDeviceId/' + this.deviceId).then(
          (data) => {
            if (data.code === '000000') {
              if (data.data) {
                var device = data.data;
                this.statusSwitch = device.statusSwitch;
                var opDeviceStatus = device.opDeviceStatus;
                if (opDeviceStatus && opDeviceStatus.length > 0) {
                  this.deviceStatus = opDeviceStatus[0];
                  this.deviceStatus.cpu = parseInt(parseFloat(this.deviceStatus.cpu));
                  this.deviceStatus.memory = parseInt(parseFloat(this.deviceStatus.memory));
                }
                this.getSubStatus();
              }
            }
          })
      }
    },
    mounted() {
      sessionStorage.setItem("active", 1);
      this.getDevice();
      this.statusTask();
    }
  }
</script>
<style scoped>
  .gray_circle {
    background-color: #CCCCCC;
    border-radius: 50%;
    width: 33px;
    height: 33px;
    margin-top: 6px;
  }

  .green_circle {
    background-color: #4CD963;
    border-radius: 50%;
    width: 33px;
    height: 33px;
    margin-top: 6px;
  }

  .yellow_circle {
    background-color: #FFC906;
    border-radius: 50%;
    width: 33px;
    height: 33px;
    margin-top: 6px;
  }

  .red_circle {
    background-color: #FF0000;
    border-radius: 50%;
    width: 33px;
    height: 33px;
    margin-top: 6px;
  }

</style>
