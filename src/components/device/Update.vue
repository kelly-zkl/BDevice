<template>
  <section class="content">

    <el-row>
      <el-col :span="5">
        <label style="width: 100%;text-align: left;margin-top: 10px">远程隧道维护</label>
      </el-col>
      <el-col :span="12" style="text-align: left;margin: 10px auto">{{device.remoteSwitch ? "已开启" : "已关闭"}}</el-col>
      <el-col :span="4" :offset="1">
        <el-switch v-model="device.remoteSwitch" on-color="#13ce66" off-color="#ff4949"
                   @change="setRemoteSwitch()"></el-switch>
      </el-col>
    </el-row>
    <hr/>

    <div>
      <br/>
      <label style="width: 100%;text-align: left;margin-top: 10px">设备能力</label>
      <hr/>

      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">短信模块</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">{{smsSwitch ? "已开启" : "已关闭"}}</el-col>
        <el-col :span="4" :offset="1">
          <el-switch v-model="smsSwitch" on-color="#13ce66" off-color="#ff4949" disabled></el-switch>
        </el-col>
      </el-row>

      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">短信推送</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">{{smsPushSwitch ? "已开启" : "已关闭"}}</el-col>
        <el-col :span="4" :offset="1">
          <el-switch v-model="smsPushSwitch" on-color="#13ce66" off-color="#ff4949" disabled></el-switch>
        </el-col>
      </el-row>


      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">自动时间同步</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">{{autoSyncTimeSwitch ? "已开启" : "已关闭"}}</el-col>
        <el-col :span="4" :offset="1">
          <el-switch v-model="autoSyncTimeSwitch" on-color="#13ce66" off-color="#ff4949" disabled></el-switch>
        </el-col>
      </el-row>

      <br/>
      <label style="width: 100%;text-align: left;margin-top: 10px">维护</label>
      <hr/>

      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">设备重启</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">单击右侧按钮将使该设备重启</el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="restart">重启</el-button>
        </el-col>
      </el-row>


      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">休眠</label>
        </el-col>
        <el-col :span="7" style="text-align: left;margin: 10px auto">单击右侧按钮将使该设备休眠</el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="restart" disabled>休眠</el-button>
        </el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="restart" disabled>唤醒</el-button>
        </el-col>
      </el-row>


      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">远程升级</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">
          <el-input v-model="url" :maxlength=100 placeholder="请输入升级URl"></el-input>
          <el-input v-model="md5" :maxlength=64 placeholder="请输入MD5"></el-input>
        </el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="updateSys">升级</el-button>
        </el-col>
      </el-row>


      <el-row style="margin: 10px auto">
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">回传速率诊断</label>
        </el-col>
        <el-col :span="12" style="text-align: left">
          <el-input disabled v-model="retrievalRate1">
            <template slot="append">/秒</template>
          </el-input>
        </el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="diagnose" :loading="logining" disabled>诊断</el-button>
        </el-col>
      </el-row>


      <el-row>
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">调试日志</label>
        </el-col>
        <el-col :span="12" style="text-align: left;margin: 10px auto">查看设备上报的调试日志，以便排查问题
        </el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="checkOut" disabled>查看</el-button>
        </el-col>
      </el-row>


      <el-row style="margin: 10px auto">
        <el-col :span="5">
          <label style="width: 100%;text-align: left;margin-top: 10px">恢复出厂设置</label>
        </el-col>
        <el-col :span="12" style="text-align: left">恢复出厂设置将丢失当前数据，请谨慎操作！
        </el-col>
        <el-col :span="4" :offset="1">
          <el-button type="success" style="width: 100%" @click="resetDefault" disabled>恢复出厂设置</el-button>
        </el-col>
      </el-row>
    </div>
  </section>
</template>
<script>
  export default {
    props: ['retrievalRate', 'deviceId'],
    data() {
      return {
        url: "",
        md5: "",
        device: {},
        retrievalRate1: this.retrievalRate,
        deviceId1: this.deviceId,
        wirelessSwitch: true,
        smsSwitch: true,
        powerAmplifierSwitch: true,
        smsPushSwitch: true,
        autoSyncTimeSwitch: true,
        logining: false
      }
    },
    watch: {
      retrievalRate: function () {
        this.retrievalRate1 = this.retrievalRate;
      },
      deviceId: function () {
        this.deviceId1 = this.deviceId;
        this.getDevice();
      }
    },
    created() {
      this.retrievalRate1 = this.retrievalRate;
      this.deviceId1 = this.deviceId;
      this.getDevice();
    },
    methods: {
      setRemoteSwitch() {
        this.$post("device/update/remoteSwitch/" + this.deviceId1 + '/' + this.device.remoteSwitch, {}, '设置成功', '设置失败').then((data) => {
          this.getDevice();
        })
      },
      //重启
      restart() {
        this.$confirm('确定要远程重启设备吗？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/restart/' + this.deviceId1, {}, "重启命令下发成功", "重启命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //恢复出厂设置
      resetDefault() {
        this.$confirm('确定要恢复出厂设置吗？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/reset/' + this.deviceId1, {}, "恢复出厂设置命令下发成功", "恢复出厂设置命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //升级
      updateSys() {
        if (!this.url || !this.md5) {
          this.$alert('请输入URL和MD5', '升级', {
            confirmButtonText: '确定',
            callback: action => {
            }
          });
          return;
        }
        this.$confirm('确定要升级吗？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/upgrade/' + this.deviceId1, {
            md5sum: this.md5,
            patchUrl: this.url
          }, "升级命令下发成功", "升级命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //下发
      sendDown() {
        this.$post('device/deliverDevice/' + this.deviceId1, {}, "参数下发成功", "参数下发失败").then((data) => {
        })
      },
      //诊断
      diagnose() {
        this.logining = true;
        this.$post('cmd/deliverGetRate/' + this.deviceId1, {}, '诊断命令下发成功', '诊断命令下发失败').then((data) => {
          if (data.code === '000000') {
            this.$emit('subRate', false);
          }
          setTimeout(() => {
            this.logining = false
          }, 5000);
        })
      },
      getDevice() {
        this.device = {};
        this.$post('device/get/byDeviceId/' + this.deviceId1).then(
          (data) => {
            if (data.code === '000000') {
              if (data.data) {
                this.device = data.data;
                var deviceStatus = this.device.opDeviceStatus;
                if (deviceStatus && deviceStatus.length > 0) {
                  this.wirelessSwitch = deviceStatus[0].wirelessSwitch === 1;
                  this.smsSwitch = deviceStatus[0].smsSwitch === 1;
                  this.powerAmplifierSwitch = deviceStatus[0].paSwitch === 1;
//                    smsPushSwitch = deviceStatus.wirelessSwitch === 1;
//                    autoSyncTimeSwitch= deviceStatus.wirelessSwitch === 1;
                }
              }
            }
          })
      },
      //查看
      checkOut() {
        this.closeShow();
      },
      closeShow() {
        this.$emit('showLog', false);
      },
    },
    mounted() {
      this.getDevice();
    }
  }
</script>
