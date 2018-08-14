<template>
  <div>
    <section class="content">

      <br/>

      <el-form label-width="100px" label-position="left" ref="deviceMonitor" :rules="rules" :model="deviceMonitor">
        <el-form-item label="设备ID" style="text-align: left">{{deviceMonitor.deviceId}}
        </el-form-item>
        <el-form-item label="设备名称" style="text-align: left">{{deviceMonitor.deviceName}}
        </el-form-item>

        <el-form-item class="item text-left" label="设备类型">
          <el-select v-model="deviceMonitor.devType" placeholder="请选择" style="width:250px">
            <el-option v-for="item in deviceTypes" :key="item.code" :label="item.name" :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item class="item text-left" label="设备形态">
          <el-select v-model="deviceMonitor.deviceForm" placeholder="请选择" style="width:250px">
            <el-option v-for="item in deviceForms" :key="item.code" :label="item.name" :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="安装地址" required>
          <el-row>
            <el-col :span="8">
              <el-cascader expand-trigger="hover" :options="provinceList" v-model="selectedOptions2"
                           :props="props" @change="areaChange" style="width: 100%">
              </el-cascader>
            </el-col>
            <el-col :span="15" :offset="1">
              <el-input v-model="deviceMonitor.detailAddress" placeholder="请输入详细地址，例：某大厦x楼/某小区x房间号"
                        :maxlength=50></el-input>
            </el-col>
          </el-row>
        </el-form-item>

        <el-form-item label="经纬度" class="item">
          <el-row>
            <el-col :span="10" style="margin-right: 10px">
              <el-input v-model="deviceMonitor.devicePos.longitude" disabled>
                <template slot="prepend">经度</template>
              </el-input>
            </el-col>
            <el-col :span="10">
              <el-input v-model="deviceMonitor.devicePos.latitude" disabled>
                <template slot="prepend">纬度</template>
              </el-input>
            </el-col>

            <el-col :span="2" :offset="1">
              <el-button type="primary" @click="selectAdd()">地图选址
              </el-button>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="高度（米）" class="item" prop="height">
          <el-input v-model="deviceMonitor.devicePos.height" placeholder="请输入高度" :maxlength=5
                    @change="checkHeight(deviceMonitor.devicePos.height)" @keypress.enter.native="keypre2()">
          </el-input>
        </el-form-item>
        <el-form-item class="item text-left" label="场所编码" prop="serviceCode">
          <el-row>
            <el-col :span="10" style="margin-right: 10px">
              <el-input v-model="placeCode" disabled></el-input>
            </el-col>
            <el-col :span="10">
              <el-tooltip placement="bottom">
                <div slot="content">4位场所编码</div>
                <el-input v-model="numberCode" :maxlength=4 @change="checkNo(numberCode)"
                          @keypress.enter.native="keyPre1()"></el-input>
              </el-tooltip>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item class="item text-left" label="场所类型" prop="serviceType">
          <el-select v-model="deviceMonitor.serviceType" placeholder="请选择" @change="typeChange" style="width:250px">
            <el-option v-for="item in serviceTypes" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="数据卡电话" class="item" prop="phoneNumber">
          <el-input v-model="deviceMonitor.phoneNumber" placeholder="请输入数据卡电话号码" type="number" :maxlength=20></el-input>
        </el-form-item>

        <el-form-item label="MAC地址" class="item" prop="mac" style="text-align: left">
          {{deviceMonitor.mac ? deviceMonitor.mac : "未上报"}}
        </el-form-item>

        <!--<el-form-item label="当前频点" class="item" prop="currentFrequency">-->
        <!--<el-input v-model="deviceMonitor.currentFrequency" placeholder="请输入当前频点" type="number"-->
        <!--:maxlength=15></el-input>-->
        <!--</el-form-item>-->

        <el-form-item label="远程隧道维护">
          <el-row>
            <el-col :span="12">{{deviceMonitor.remoteSwitch ? "已开启" : "已关闭"}}
            </el-col>
            <el-col :span="4" :offset="1">
              <el-switch v-model="deviceMonitor.remoteSwitch" on-color="#13ce66" off-color="#ff4949"
                         @change="setRemoteSwitch()"></el-switch>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="固件版本" class="item" prop="deviceVersion">
          <el-row>
            <el-col :span="9" style="text-align: left">
              {{deviceMonitor.deviceVersion ? deviceMonitor.deviceVersion : "未上报"}}
            </el-col>
            <el-col :span="2" :offset="5">
              <el-button type="primary" @click.stop="updateSys()">升级
              </el-button>
            </el-col>
          </el-row>
        </el-form-item>

        <el-form-item label="恢复出厂设置">
          <el-row>
            <el-col :span="12">恢复出厂设置将丢失当前数据，请谨慎操作！
            </el-col>
            <el-col :span="4" :offset="1">
              <el-button type="success" style="width: 100%" @click="resetDefault" disabled>恢复出厂设置</el-button>
            </el-col>
          </el-row>
        </el-form-item>

        <!--<el-form-item label="软件版本" class="item" prop="softVersion" disabled>-->
        <!--<el-input v-model="deviceMonitor.softVersion" placeholder="请输入软件版本" :maxlength=50></el-input>-->
        <!--</el-form-item>-->
        <hr/>
        <el-form-item>
          <el-row>
            <el-col :span="2" :offset="8">
              <el-button type="success" @click="updateBaseInfo">保存</el-button>
            </el-col>
            <el-col :span="2" style="margin-left: 20px">
              <el-button type="success" @click="setBaseParam">参数下发</el-button>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
    </section>
    <el-dialog title="" :visible.sync="mapVisible">
      <MapView @getLocation="getLocation" v-bind:formattedAddress="getAddress"></MapView>
      <el-row>
        <el-col :span="8" :offset="8" style="margin-top: 10px">
          <el-button type="primary" @click="setLocation">确定</el-button>
          <el-button @click="mapVisible = false">取消</el-button>
        </el-col>
      </el-row>
    </el-dialog>
  </div>
</template>
<script>
  import json from '../../assets/city.json';
  import MapView from '../device/map';
  import axios from "axios";

  export default {
    props: ['Id'],
    data() {
      return {
        props: {
          value: 'o',
          label: 'n',
          children: 'c'
        },
        selectedOptions2: [],
        position: {},
        getAddress: '',
        code: '',
        provinceList: json,
        mapVisible: false,
        runningStatusVisible: false,
        deviceMonitor: {
          ip: '',
          endTime: 0,
          startTime: 0,
          serviceType: '',
          devicePos: {
            height: '',
            longitude: '',
            latitude: ''
          }
        },
        rules: {
          serviceType: [
            {required: true, message: '请选择场所类型', trigger: 'blur'}
          ],
          serviceCode: [
//            {required: true, message: '请输入设备场所编码', trigger: 'blur', maxlength: 4},
          ]
        },
        operations: '',
        network: '',
        Id1: this.Id,
        deviceForms: [],
        deviceTypes: [],
        device: {
          remoteSwitch: true
        },
        placeCode: '',
        numberCode: '',
        serviceTypes: [
          {
            value: '0',
            label: '网吧'
          },
          {
            value: '1',
            label: '旅店宾馆类（住宿服务场所）'
          },
          {
            value: '2',
            label: '图书馆阅览室'
          },
          {
            value: '3',
            label: '电脑培训中心类'
          },
          {
            value: '4',
            label: '娱乐场所类'
          },
          {
            value: '5',
            label: '交通枢纽'
          },
          {
            value: '6',
            label: '公共交通工具'
          },
          {
            value: '7',
            label: '餐饮服务场所'
          },
          {
            value: '8',
            label: '金融服务场所'
          },
          {
            value: 'A',
            label: '购物场所'
          },
          {
            value: 'B',
            label: '公共服务场所'
          },
          {
            value: 'C',
            label: '文化服务场所'
          },
          {
            value: 'D',
            label: '公共休闲场所'
          },
          {
            value: '9',
            label: '其他'
          }
        ]
      }
    },
    watch: {
      Id: function () {
        this.Id1 = this.Id;
        this.getBaseInfo();
      }
    },
    created() {
      this.Id1 = this.Id;
      this.getDeviceTypeAndForm();
      this.getBaseInfo();
    },
    computed: {
      // 计算属性的 getter
      startTime: {
        get() {
          return this.getTimeStr(this.deviceMonitor.startTime);
        },
        set(newValue) {
          this.deviceMonitor.startTime = this.getTimeNum(newValue);
        },
      },
      endTime: {
        get() {
          return this.getTimeStr(this.deviceMonitor.endTime);
        },
        set(newValue) {
          this.deviceMonitor.endTime = this.getTimeNum(newValue);
        },
      }
    },
    methods: {
      getTimeStr(time) {
        if (isNaN(time) || time === 0) {
          return '00:00';
        }
        let hour = Math.floor(time / 60 / 60);
        let minute = time / 60 % 60;
        if (hour < 10) {
          hour = "0" + hour;
        }
        if (minute < 10) {
          minute = "0" + minute;
        }
        return hour + ":" + minute;
      },
      getTimeNum(newValue) {
        let hour = parseInt(newValue.substring(0, 2), 10);
        let minute = parseInt(newValue.substring(3, 5), 10);
        if (isNaN(hour) || isNaN(minute)) {
          return 0
        } else {
          return (hour * 60 + minute) * 60;
        }
      },
      keypre2() {
        let event = arguments.callee.caller.arguments[0] || window.event;//消除浏览器差异
        return ((event.keyCode >= 48 && event.keyCode <= 57) || event.keyCode == 46);
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
          if (new RegExp(reg).test(value) === false) {
            setTimeout(() => {
              this.numberCode = '';
            }, 500);
          }
        }
      },
      //只能输入小数点后2位--》高度
      checkHeight(value) {
        let reg = /^[1-9][0-9]*\.\d{2}$|^[0]\.\d{1,2}$/g;
        if (value) {
          if (new RegExp(reg).test(value) === false) {
            value = value.replace(reg, "");
          }
        }
      },
      //地图选址，显示dialog
      selectAdd() {
        if (!this.code) {
          this.$message.error('请选择省市区');
          return;
        }
        if (this.code) {
          if (this.deviceMonitor.detailAddress) {
            this.getAddress = this.getAreaLable(this.code) + this.deviceMonitor.detailAddress;
          } else {
            this.getAddress = this.getAreaLable(this.code);
          }
        }
        console.log(this.getAddress);
        this.mapVisible = true;
      },
      //地图选址完成设置经纬度
      setLocation() {
        this.deviceMonitor.devicePos.longitude = this.position.lng;
        this.deviceMonitor.devicePos.latitude = this.position.lat;
        this.deviceMonitor.detailAddress = this.position.detailAddress;
        this.selectedOptions2 = this.getCode(this.position.code);
        this.mapVisible = false;
      }
      ,
      //获得地图选择的经纬度
      getLocation(pos) {
        this.position = pos;
        console.log(pos);
      },
      //获取设备的基本信息
      getBaseInfo() {
        this.selectedOptions2 = [];
        this.$post("device/get/" + this.Id1, {}).then((data) => {
            if (data.code === '000000') {
              this.getLineStatus();//获取设备在线状态
              if (data.data) {
                this.deviceMonitor = data.data;

                if (!data.data.devicePos) {
                  this.deviceMonitor.devicePos = {
                    height: '',
                    longitude: '',
                    latitude: ''
                  }
                }

                //场所编码
                if (data.data.serviceCode) {
                  this.placeCode = data.data.serviceCode.substring(0, 10);
                  this.numberCode = data.data.serviceCode.substring(data.data.serviceCode.length - 4, data.data.serviceCode.length);
                }

                if (data.data.areaCode) {
                  this.selectedOptions2 = this.getCode(data.data.areaCode);
                  this.code = data.data.areaCode;
                } else if (data.data.cityCode) {
                  this.selectedOptions2 = this.getCode(data.data.cityCode);
                  this.code = data.data.cityCode;
                } else if (data.data.provinceCode) {
                  this.selectedOptions2 = this.getCode(data.data.provinceCode);
                  this.code = data.data.provinceCode;
                } else {
                  this.selectedOptions2 = [];
                  this.code = '';
                }
              }
            }
          }
        )
      },
      //获取设备类型和形态
      getDeviceTypeAndForm() {
        this.$post('device/get/deviceType').then(
          (data) => {
            if (data.code === '000000' && data.data) {
              this.deviceTypes = data.data;
            }
          });

        this.$post('device/get/deviceForm').then(
          (data) => {
            if (data.code === '000000' && data.data) {
              this.deviceForms = data.data;
            }
          });
      },
      //根据区域码找到对应的省市县
      getCode(code) {
        json.forEach((province) => {
          if (province.c) {
            province.c.forEach((city) => {
              if (city.c) {
                city.c.forEach((country) => {
                  if (code === country.o) {
                    this.selectedOptions2 = [province.o, city.o, country.o];
                  }
                })
              } else {
                if (code === city.o) {
                  this.selectedOptions2 = [province.o, city.o];
                }
              }
            })
          } else {
            if (code === province.o) {
              this.selectedOptions2 = [province.o];
            }
          }
        });
        return this.selectedOptions2;
      },
      //恢复出厂设置
      resetDefault() {
        this.$confirm('确定要恢复出厂设置吗？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/reset/' + this.deviceMonitor.deviceId, {}, "恢复出厂设置命令下发成功", "恢复出厂设置命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //远程隧道开启/关闭
      setRemoteSwitch() {
        this.$post("device/update/remoteSwitch/" + this.deviceMonitor.deviceId + '/' + this.deviceMonitor.remoteSwitch, {}, '设置成功', '设置失败').then((data) => {
          this.getBaseInfo();
        })
      },
      //升级
      updateSys() {
        this.$confirm('确认进行设备升级？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/upgrade/' + this.deviceMonitor.deviceId, {}, "升级命令下发成功", "升级命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //修改设备参数
      updateBaseInfo() {
        if (this.selectedOptions2.length === 0) {
          this.$message({
            type: 'info',
            message: '请选择安装地址省市区'
          });
          return;
        }
        if (this.numberCode.length !== 4) {
          this.$message({
            type: 'info',
            message: '场所编码需手动输入4位'
          });
          return;
        }
        this.getScode(this.selectedOptions2);
        this.deviceMonitor.serviceCode = this.placeCode + this.numberCode;
        this.$refs['deviceMonitor'].validate((valid) => {
          if (valid) {
            this.$post("device/modify/baseinfo", this.deviceMonitor, '修改成功', '修改失败').then((data) => {
              this.getBaseInfo();
            })
          }
        });
      },
      //省市区
      getScode(value) {
        if (value.length === 1) {
          this.deviceMonitor.provinceCode = value[0];
        } else if (value.length === 2) {
          this.deviceMonitor.provinceCode = value[0];
          this.deviceMonitor.cityCode = value[1];
        } else if (value.length === 3) {
          this.deviceMonitor.provinceCode = value[0];
          this.deviceMonitor.cityCode = value[1];
          this.deviceMonitor.areaCode = value[2];
        }
      },
      //省市县变化
      areaChange(value) {
        this.selectedOptions2 = value;
        if (value.length === 1) {
          this.code = value[0];
        } else if (value.length === 2) {
          this.code = value[1];
        } else if (value.length === 3) {
          this.code = value[2];
        }
        if (this.deviceMonitor.serviceType) {
          this.placeCode = this.code + '8' + this.deviceMonitor.serviceType + '01';
        } else {
          this.placeCode = this.code + '801';
        }
      },
      //场所类型的选择
      typeChange(value) {
        //value为场所类型
        this.deviceMonitor.serviceType = value;
        if (this.deviceMonitor.serviceType) {
          this.placeCode = this.code + '8' + this.deviceMonitor.serviceType + '01';
        } else {
          this.placeCode = this.code + '801';
        }
      },
      //获得省市县
      getAreaLable(code) {
        var lable = '';
        this.provinceList.forEach((province) => {
          if (province.c) {
            province.c.forEach((city) => {
              if (city.c) {//省级+市级+县级
                city.c.forEach((country) => {
                  if (code === country.o) {
                    lable = province.n + city.n + country.n;
                  }
                })
              } else {//省级+市级
                if (code === city.o) {
                  lable = province.n + city.n;
                }
              }
            })
          } else {//只包含省级
            if (code === province.o) {
              lable = province.n;
            }
          }
        });
        return lable;
      },
      //获取在线状态
      getLineStatus() {
        this.$post("device/get/lineStatus/" + this.deviceMonitor.deviceId, {}).then((res) => {
          this.deviceMonitor.lineStatus = '';
          if ("000000" === res.code) {
            if (res.data) {
              if (res.data.online === true) {
                this.deviceMonitor.lineStatus = 'ON';
              } else {
                this.deviceMonitor.lineStatus = 'OFF';
              }
            }
          }
        });
      },
      //下发参数
      setBaseParam() {
        this.deviceMonitor.areaCodeVal = this.getAreaLable(this.deviceMonitor.areaCode);
        this.$confirm('确认下发当前参数？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post("device/setBaseParam", this.deviceMonitor, '命令下发成功', '命令下发失败').then((data) => {
          })
        }).catch(() => {
        });
      }
    },
    mounted() {
      this.getBaseInfo();
    },
    components: {
      MapView
    }
  }
</script>

