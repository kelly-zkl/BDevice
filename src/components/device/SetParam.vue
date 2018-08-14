<template>
  <section class="content">
    <br/>
    <el-tabs v-model="activeItem" @tab-click="handleClick">
      <el-tab-pane v-bind:label="tab.name" v-for="tab in activeName"
                   :key="tab.type" v-bind:name="tab.type" :value="tab.moduleID">
        <el-form label-width="120px" label-position="left" :rules="rules" :model="opDeviceParameter" required>

          <!--<label style="width: 100%;text-align: left;margin-top: 10px">无线参数</label>-->
          <hr/>

          <!--<el-form-item label="网络格式" class="item text-left" required>-->
          <!--<el-select v-model="opDeviceParameter.network" placeholder="请选择">-->
          <!--<el-option v-for="item in networks" :key="item.name" :label="item.name" :value="item.name">-->
          <!--</el-option>-->
          <!--</el-select>-->
          <!--</el-form-item>-->

          <el-row>
            <el-col :span="11">
              <!--<el-form-item label="工作频点" required>-->
              <!--<el-input v-model="opDeviceParameter.upFrequency">-->
              <!--<template slot="prepend">上行</template>-->
              <!--</el-input>-->
              <!--</el-form-item>-->
              <!--<el-form-item>-->
              <!--<el-input v-model="opDeviceParameter.downFrequency">-->
              <!--<template slot="prepend">下行</template>-->
              <!--</el-input>-->
              <!--</el-form-item>-->
              <el-form-item label="mcc" prop="mcc" required>
                <el-tooltip placement="bottom">
                  <div slot="content">移动国家号：由三个十进制数组成，编码范围为十进制的000－999,中国-460</div>
                  <el-input v-model="opDeviceParameter.mcc" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="mnc" prop="mnc" required>
                <el-tooltip placement="bottom">
                  <div slot="content">移动网号：由二个十进制数组成，编码范围为十进制的00－99<br/>中国范围:
                    <br/>联通[1,6,9,10]<br/>移动[0,2,4,7,8]<br/>电信[3,11,12]
                  </div>
                  <el-input v-model="opDeviceParameter.mnc" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="lac" prop="lac" required v-show="opDeviceParameter.network =='GSM'">
                <el-tooltip placement="bottom">
                  <div slot="content">位置区码 取值范围：[0001－FFFEH]，码组0000H和FFFFH不可以使用</div>
                  <el-input v-model="opDeviceParameter.lac" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="pci" prop="pci" required v-show="opDeviceParameter.network!='GSM'">
                <el-tooltip placement="bottom">
                  <div slot="content">物理小区标识 取值范围：[0-504]</div>
                  <el-input v-model="opDeviceParameter.pci" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="band" prop="band" required>
                <el-tooltip placement="bottom">
                  <div slot="content">基站频段号 取值范围：<br/>GSM：900/1800<br/>FDD：1/3<br/>TDD：[38-41]</div>
                  <el-input v-model="opDeviceParameter.band" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
            </el-col>

            <el-col :span="11" :offset="2">
              <el-form-item label="bts" prop="bts" required>
                <el-input v-model="opDeviceParameter.bts" :maxlength=10 disabled>
                </el-input>
              </el-form-item>
              <el-form-item label="bcc" prop="bcc" required>
                <el-tooltip placement="bottom">
                  <div slot="content">频点号 取值范围：<br/>GSM：移动[1-94][512-562],联通[96-124][686-735]<br/>FDD：联通[0-599],电信[200-1949]<br/>TDD：移动-38 [37750-38249],移动-39 [38250-38649],移动-40 [38650-39649]
                  </div>
                  <el-input v-model="opDeviceParameter.bcc" :maxlength=10>
                  </el-input>
                </el-tooltip>
              </el-form-item>
              <el-form-item label="rssimin" prop="rssimin" required>
                <el-tooltip placement="bottom">
                  <div slot="content">信号强度 取值范围：固定80</div>
                  <el-input v-model="opDeviceParameter.rssimin" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>

              <el-form-item label="tac" prop="tac" required v-show="opDeviceParameter.network!='GSM'">
                <el-tooltip placement="bottom">
                  <div slot="content">跟踪区域码 取值范围：[0001－FFFEH]，码组0000H和FFFFH不可以使用</div>
                  <el-input v-model="opDeviceParameter.tac" :maxlength=10></el-input>
                </el-tooltip>
              </el-form-item>
            </el-col>
          </el-row>

          <hr/>
          <el-row>
            <el-col :span="11">
              <el-form-item label="重复抓取时间" prop="repeat">
                <el-row>
                  <el-col :span="16">
                    <el-input v-model="opDeviceParameter.repeat" placeholder="点击设置将下发参数至设备" :maxlength=5
                              @change="checkNo(opDeviceParameter.repeat)" @keyup.enter.native="keyPre1()"
                              style="width: 100%">
                      <template slot="append">分</template>
                    </el-input>
                  </el-col>

                  <el-col :span="6" :offset="2">
                    <el-button type="success" style="width: 100%" @click="setRepeatCatch">设置</el-button>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-col>
            <el-col :span="11" :offset="2">
              <el-form-item label="PA增益调节">
                <el-row>
                  <el-col :span="16" class="item text-left">
                    <el-input-number v-model="paNum" :min="0" :max="20" :maxlength=10
                                     style="width: 100%"></el-input-number>
                  </el-col>

                  <el-col :span="6" :offset="2">
                    <el-button type="success" style="width: 100%" @click="setPA">设置</el-button>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-col>
          </el-row>

          <hr/>
          <el-row>
            <el-col :span="11">
              <el-form-item label="功放">
                <el-row>
                  <el-col :span="12">{{wirelessSwitch ? "已开启" : "已关闭"}}
                  </el-col>
                  <el-col :span="4" :offset="1">
                    <el-switch v-model="wirelessSwitch" on-color="#13ce66" off-color="#ff4949"
                               @change="setWirelessSwitch"></el-switch>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-col>
            <el-col :span="11" :offset="2" v-show="false">
              <el-form-item label="功放">
                <el-row>
                  <el-col :span="12">{{powerAmplifierSwitch ? "已开启" : "已关闭"}}
                  </el-col>
                  <el-col :span="4" :offset="1">
                    <el-switch v-model="powerAmplifierSwitch" on-color="#13ce66" off-color="#ff4949"
                               @change="setPASwitch"></el-switch>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-col>
          </el-row>
          <hr/>

          <el-form-item>
            <el-row>
              <el-col :span="2" :offset="8">
                <el-button type="success" @click="updateParam">保存</el-button>
              </el-col>
              <el-col :span="2" :offset="8" style="margin-left: 20px">
                <el-button type="success" @click="setBaseParam()">参数下发</el-button>
              </el-col>
            </el-row>
          </el-form-item>
        </el-form>
      </el-tab-pane>
    </el-tabs>
  </section>
</template>
<script>
  import {numValid} from '../../api'

  export default {
    props: ['deviceId'],

    data() {
      let numValidator = (rule, value, callback) => {
        if (!/^[0-9]\d*$/.test(value)) {
          callback(new Error("只能输入整数"));
        } else {
          callback();
        }
      };
      return {
        tacPeriod: 1,
//        repeatModuleId:1,
        paNum: 1,
        subModuleStatus: {},
        wirelessSwitch: true,
        powerAmplifierSwitch: true,
//        paModuleId:1,
        opDeviceParameter: {
          moduleID: 1,
          bts: 1,
          network: 'GSM',
          band: 1,
          bcc: 1,
          lac: 0,
          mcc: 460,
          mnc: 1,
          rssimin: 80,
          pci: 0,
          tac: 0,
          repeat: 1,
          datatag: 'GSM1'
        },
        networks: [
          {name: "GSM"}, {name: "FDDLTE"}, {name: "TDDLTE"}, {name: "CDMA"}
        ],
        moduleIDs: [{module: 1}, {module: 2}, {module: 3}, {module: 4}, {module: 5}, {module: 6}],
        activeName: [
          {
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
          },
        ],
        activeItem: "GSM1",
        paramShow: false,
        deviceId1: this.deviceId,
        rules: {
          bts: [
            {required: true, message: '请输入bts'}
          ],
          band: [
            {required: true, message: '请输入band'}
          ],
          bcc: [
            {required: true, message: '请输入bcc'}
          ],
          lac: [
            {required: true, message: '请输入lac'}
          ],
          mcc: [
            {required: true, message: '请输入mcc'}
          ],
          mnc: [
            {required: true, message: '请输入mnc'}
          ],
          rssimin: [
            {required: true, message: '请输入rssimin'}
          ],
          pci: [
            {required: true, message: '请输入pci'}
          ],
          tac: [
            {required: true, message: '请输入tac'}
          ],
          repeat: [
            {validator: numValidator, trigger: "change,blur"}
          ]
        }
      }
    },
    watch: {
      deviceId: function () {
        this.deviceId1 = this.deviceId;
        this.opDeviceParameter.datatag = 'GSM1';
        this.opDeviceParameter.moduleID = 1;
        this.clear();
        this.getParam(this.opDeviceParameter.datatag);
      }
    },
    created() {
      this.deviceId1 = this.deviceId;
      this.opDeviceParameter.datatag = 'GSM1';
      this.opDeviceParameter.moduleID = 1;
      this.clear();
      this.getParam(this.opDeviceParameter.datatag);

    },
    methods: {
      handleClick(tab, event) {
        this.opDeviceParameter.datatag = this.activeItem;
        this.opDeviceParameter.moduleID = this.getModuleID();
        this.clear();
        this.getParam(this.opDeviceParameter.datatag);

        this.opDeviceParameter.network = this.getNetwork();
        this.opDeviceParameter.bts = this.opDeviceParameter.moduleID;
//        console.log(this.opDeviceParameter.network);
      },
      getNetwork() {
        switch (this.getModuleID()) {
          case 1:
          case 2:
            return "GSM";
          case 3:
          case 4:
            return "FDDLTE";
          case 5:
          case 6:
            return "TDDLTE";
        }
      },
      //输入数字
      keyPre1() {
        let event = arguments.callee.caller.arguments[0] || window.event;//消除浏览器差异
        return event.keyCode >= 48 && event.keyCode <= 57;
      },
      //只能输入正整数--》场所编码
      checkNo(value) {
        let reg = /^[0-9]\d*$/;
        if (value) {
          if (new RegExp(reg).test(value) == false) {
            setTimeout(() => {
              this.numberCode = '';
            }, 500);
          }
        }
      },
      clear() {
        //初始化
        this.opDeviceParameter.bts = 1;
        this.opDeviceParameter.network = 'GSM';
        this.opDeviceParameter.band = 1;
        this.opDeviceParameter.bcc = 1;
        this.opDeviceParameter.lac = 0;
        this.opDeviceParameter.mcc = 460;
        this.opDeviceParameter.mnc = 1;
        this.opDeviceParameter.rssimin = 80;
        this.opDeviceParameter.pci = 0;
        this.opDeviceParameter.tac = 0;
        this.opDeviceParameter.repeat = 1;

        this.paNum = 1;
      },
      getModuleID() {
        for (let item of this.activeName) {
          if (this.activeItem == item.type) {
            return item.moduleID;
          }
        }
      },
      getParam(datatag) {
        this.$post('device/get/byDeviceId/' + this.deviceId1, {}).then(
          (data) => {
            if (data.data && data.data.wirlessSet && data.data.wirlessSet.length > 0) {
              //无线参数
              for (let wirless of data.data.wirlessSet) {
                if (wirless.datatag && datatag == wirless.datatag) {
                  this.opDeviceParameter = $.Bean.copyProperty(wirless, this.opDeviceParameter);
                  break;
                }
              }
            }
            //子模块状态
            if (data.data && data.data.opDeviceStatus && data.data.opDeviceStatus.length > 0) {
              var deviceStatus = data.data.opDeviceStatus[0];
              if (deviceStatus.subModuleDataList && deviceStatus.subModuleDataList.length > 0) {
                for (let item of deviceStatus.subModuleDataList) {
                  if (item.moduleId == this.getModuleID()) {
                    this.subModuleStatus = item;
                    this.wirelessSwitch = (item.wirelessSwitch == 1);
                    this.powerAmplifierSwitch = (item.paSwitch == 1);
                  }
                }
              }
            }
            //重复抓取 时间
            if (data.data && data.data.repeatCatchTime && data.data.repeatCatchTime.length > 0) {
              for (let repeatCatch of data.data.repeatCatchTime) {
                if (repeatCatch.moduleID && this.getModuleID() == repeatCatch.moduleID) {
                  this.opDeviceParameter.repeat = repeatCatch.tacPeriod;
                  break;
                }
              }
            }
            //PA调节
            if (data.data && data.data.paGainControl && data.data.paGainControl.length > 0) {
              for (let pa of data.data.paGainControl) {
                if (pa.moduleID && this.getModuleID() == pa.moduleID) {
                  this.paNum = pa.powerValue;
                  break;
                }
              }
            }
          })
      },
      //设置无线电开关
      setWirelessSwitch(val) {
        let url = 'set/' + (val ? 'openWirless/' : 'closeWirless/') + this.deviceId1 + "/" + this.getModuleID();
        this.$post(url, {}, '设置成功', '设置失败').then((data) => {
        })
      },
      //设置功放开关
      setPASwitch(val) {
        let url = 'set/' + (val ? 'openPA/' : 'closePA/') + this.deviceId1 + "/" + this.getModuleID();
        this.$post(url, {}, '设置成功', '设置失败').then((data) => {
        })
      },
      //设置重复抓取时间
      setRepeatCatch() {
        if (!/^[0-9]\d*$/.test(this.opDeviceParameter.repeat)) {
          this.$message.error('重复抓取时间只能输入整数');
          return;
        }
        this.$post("set/repeatCatch/" + this.deviceId1, {
          tacPeriod: this.opDeviceParameter.repeat,
          moduleID: this.getModuleID()
        }, '设置成功', '设置失败').then((data) => {
          this.getParam(this.opDeviceParameter.datatag);
        })
      },
      //设置PA
      setPA() {
        this.$post("set/paGain/" + this.deviceId1, {
          powerValue: this.paNum,
          moduleID: this.getModuleID()
        }, '设置成功', '设置失败').then((data) => {
          this.getParam(this.opDeviceParameter.datatag);
        })
      },
      //下发参数
      setBaseParam() {
        var data = {};
        $.extend(true, data, this.opDeviceParameter);
        delete data.repeat;

        this.$confirm('确认下发当前参数？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('wirlessSet/add/' + this.deviceId1, data,
            '命令下发成功', '命令下发失败').then((data) => {
          })
        }).catch(() => {
        });
      },
      updateParam() {
        if (!numValid(this.opDeviceParameter.band)) {
          this.$message.error('band只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.pci)) {
          this.$message.error('pci只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.tac)) {
          this.$message.error('tac只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.bcc)) {
          this.$message.error('bcc只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.lac)) {
          this.$message.error('lac只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.mcc)) {
          this.$message.error('mcc只能输入整数');
          return;
        }
        if (!numValid(this.opDeviceParameter.mnc)) {
          this.$message.error('mnc只能输入整数');
          return;
        }

        var data = {};
        $.extend(true, data, this.opDeviceParameter);
        delete data.repeat;

        this.$post('wirlessSet/update/' + this.deviceId1, data,
          '修改成功', '修改失败').then(
          (data) => {
            this.getParam();
            setTimeout(() => {
              this.closeDialog();
            }, 500);
          })

      },
      closeDialog() {
        this.$emit('setClose', false);
      }
    },
    mounted() {
      this.opDeviceParameter.datatag = 'GSM1';
      this.opDeviceParameter.moduleID = 1;
      this.clear();
      this.getParam(this.opDeviceParameter.datatag);
    }
  }
</script>
