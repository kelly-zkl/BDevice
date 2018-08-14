<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item>设备列表</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <div class="center-block">
        <el-row class="panel-body">
          <el-col :span="18" class="text-left">
            <el-input placeholder="设备名称/ID" v-model="query.deviceName" :maxlength="30"
                      style="width: 150px;margin-right: 10px;margin-top: 10px"></el-input>
            <el-select v-model="query.devType" placeholder="全部类型"
                       style="width: 150px;margin-right: 10px;margin-top: 10px">
              <el-option v-for="item in deviceTypes" :key="item.code" :label="item.name" :value="item.code">
              </el-option>
            </el-select>
            <el-select v-model="query.deviceForm" placeholder="全部形态"
                       style="width: 150px;margin-right: 10px;margin-top: 10px">
              <el-option v-for="item in deviceForms" :key="item.code" :label="item.name" :value="item.code">
              </el-option>
            </el-select>
            <!--<el-select v-model="lineStatus" placeholder="全部状态"-->
            <!--style="width: 150px;margin-right: 10px;margin-top: 10px">-->
            <!--<el-option-->
            <!--v-for="item in online" :key="item.value" :label="item.label" :value="item.value">-->
            <!--</el-option>-->
            <!--</el-select>-->
            <el-button type="primary" icon="search" style="width: 80px;;margin-top: 10px"
                       @click.stop="getData()">搜索
            </el-button>
            <el-button style="width: 80px;;margin-top: 10px" @click.stop="clearData()">清除
            </el-button>
          </el-col>
          <el-col :span="6">
            <el-button type="success" style="width: 100px;margin-top: 10px" @click=""
                       :disabled="true">批量升级
            </el-button>
            <!--@click="$router.push('deviceAdd')" 添加设备页面版-->
            <el-button type="success" icon="plus" style="width: 130px;margin-top: 10px;margin-left: 20px"
                       @click="runningAddDevice = true">添加设备
            </el-button>
          </el-col>
        </el-row>

        <el-table :data="deviceList" border v-loading="listLoading" class="center-block"
                  @selection-change="selsChange">
          <el-table-column type="selection" width="45" align="left"></el-table-column>
          <el-table-column align="left" prop="deviceId" label="设备ID"
                           min-width="100" max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="deviceName" label="设备名称"
                           min-width="100" max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="serviceCode" label="场所编码"
                           width="150" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="deviceTypeVal" label="类型"
                           min-width="110" max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="deviceFormVal" label="形态"
                           min-width="125" max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="detailAddress" label="位置"
                           min-width="125" max-width="250" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" prop="lineStatus" label="在线状态"
                           width="100" :formatter="formatterAddress"></el-table-column>
          <el-table-column align="left" label="操作" width="230">
            <template slot-scope="scope">
              <el-button class="setting"
                         @click.stop="gotoSet(deviceList[scope.$index])" size="small">
              </el-button>
              <el-button class="status" @click.stop="gotoStatus(deviceList[scope.$index].deviceId)" size="small">
              </el-button>
              <el-button @click.stop="gotoTerminal(deviceList[scope.$index].deviceId)"
                         class="zmdata" size="small">
              </el-button>
              <el-popover
                ref="popover4" placement="left" width="180px" trigger="hover" visible-arrow="isShow">
                <el-row>
                  <el-col :span="24">
                    <el-button class="btn_green" type="text" @click.stop="gotoScan(deviceList[scope.$index])">
                      <i class="fa fa-crosshairs" style="margin-right: 10px"></i>扫频工具
                    </el-button>
                  </el-col>
                </el-row>
                <el-row>
                  <el-col :span="24">
                    <el-button class="btn_green" type="text" disabled>
                      <i class="fa fa-stethoscope" style="margin-right: 10px"></i>诊断工具
                    </el-button>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 30px">
                  <el-col :span="12">
                    <el-button class="btn_green" type="text"
                               @click.stop="restartDevice(deviceList[scope.$index].deviceId)">重启
                    </el-button>
                  </el-col>
                  <el-col :span="12">
                    <el-button class="btn_green" type="text" disabled>休眠
                    </el-button>
                  </el-col>
                </el-row>
              </el-popover>
              <el-button class="glmb" v-popover:popover4 size="small"></el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="block" style="margin-top: 20px">
          <el-pagination
            @size-change="handleSizeChange" @current-change="pageChange" :current-page="query.page"
            :page-sizes="[10, 15, 20, 30]" :page-size="query.size" layout="total, sizes, prev, pager, next, jumper"
            :total="count"></el-pagination>
        </div>
      </div>
      <el-dialog title="添加设备" :visible.sync="runningAddDevice" width="30%" center>
        <el-form ref="addDevice" :rules="rules" :model="addDevice" :label-position="labelPosition" label-width="100px">
          <el-form-item label="设备ID" prop="deviceId">
            <el-input v-model="addDevice.deviceId" auto-complete="off" :maxlength="30"></el-input>
          </el-form-item>
          <el-form-item label="设备名称" prop="deviceName">
            <el-input v-model="addDevice.deviceName" auto-complete="off" :maxlength="20"></el-input>
          </el-form-item>
          <el-form-item label="设备类型" style="margin-right: 200px" prop="devType">
            <el-select v-model="addDevice.devType" placeholder="请选择" style="width: 100%">
              <el-option v-for="item in deviceTypes" :key="item.code" :label="item.name" :value="item.code">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="设备形态" style="margin-right: 200px" prop="deviceForm">
            <el-select v-model="addDevice.deviceForm" placeholder="请选择" style="width: 100%">
              <el-option v-for="item in deviceForms" :key="item.code" :label="item.name" :value="item.code">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="runningAddDevice = false">取 消</el-button>
            <el-button type="primary" @click="deviceAdd()">确 定</el-button>
          </span>
      </el-dialog>
    </section>
  </div>
</template>

<script>
  import baseInfo from '../device/BaseInfo'
  import setParam from '../device/SetParam'
  import networkData from '../device/NetworkData'
  import deviceStatus from '../device/DeviceStatus'
  import update from '../device/Update'
  import logList from '../device/LogsList'
  import terminal from '../device/Terminal'

  import json from '../../assets/city.json';
  import axios from "axios";

  export default {
    data() {
      let idValidator = (rule, value, callback) => {
        if (!/[a-zA-Z0-9]$/.test(value)) {
          callback(new Error("请输入正确的设备id，由英文字母、数字组成"));
        } else {
          callback();
        }
      };
      let devValidator = (rule, value, callback) => {
        if (!/[A-Za-z0-9\u4e00-\u9fa5]$/.test(value)) {
          callback(new Error("请输入正确的设备名称，由汉字、数字、英文字母组成"));
        } else {
          callback();
        }
      };
      return {
        deviceObj: {},
        props: {
          value: 'code',
          label: 'name',
          children: 'child'
        },
        lineStatus: '',
        provinceList: json,
        listLoading: false,
        labelPosition: 'right',
        deviceList: [],
        query: {
          page: 1,
          size: 10,
          deviceForm: '',
          devType: '',
          deviceName: ''
        },
        count: 0,
        runningAddDevice: false,
        activeName: '1',
        deviceName: '',
        addDevice: {
          deviceId: '',
          deviceName: '',
          deviceForm: '',
          devType: ''
        },
        online: [{
          value: true,
          label: '在线'
        }, {
          value: false,
          label: '离线'
        }],
        rules: {
          deviceName: [
            {required: true, message: '请输入设备名称', trigger: 'blur', maxlength: 20},
            {validator: devValidator, trigger: "change,blur"}
          ],
          deviceId: [
            {required: true, message: '请输入设备id', trigger: 'blur', maxlength: 30},
            {validator: idValidator, trigger: "change,blur"}
          ],
          devType: [
            {required: true, message: '请选择设备类型', trigger: 'blur'}
          ],
          deviceForm: [
            {required: true, message: '请选择设备形态', trigger: 'blur'}
          ]
        },
        sels: [],
        Id: '',
        deviceId: '',
        retrievalRate: '',
        client: {},
        isShow: true,
        deviceForms: [],
        deviceTypes: [],
        intervalid: null
      }
    },
    //页面关闭时停止更新设备在线状态
    beforeDestroy() {
      clearInterval(this.intervalid);
    },
    methods: {

      notifyClose() {
        this.$refs.deviceStatus.dialogCloseHandle();
      },
      showList(msg) {
        this.isShow = msg;
      },
      rowClick(row, event, column) {
        this.runningStatus(row, '1');
      },
      //跳转设备的侦码数据页面
      gotoTerminal(deviceId) {
        this.$parent.$data.active = 1;
        this.$router.push({path: 'terminal', query: {deviceId: deviceId}});
      },
      //跳转到设备状态页面
      gotoStatus(deviceId) {
        this.$parent.$data.active = 1;
        this.$router.push({path: 'deviceStatus', query: {deviceId: deviceId}});
      },
      //跳转到设备设置页面
      gotoSet(device) {
        this.$parent.$data.active = 1;
        this.$router.push({path: 'deviceSet', query: {Id: device.id, deviceId: device.deviceId}});
      },
      //跳转到扫频工具页面
      gotoScan(device) {
        this.$parent.$data.active = 1;
        this.$router.push({path: 'networkData', query: {deviceId: device.deviceId}});
      },
      runningStatus(device, type) {
        this.activeName = type;
        this.isShow = true;
        this.deviceName = device.deviceName;
        this.deviceId = device.deviceId;
        this.Id = device.id;
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
      selsChange(sels) {
        this.sels = sels;
      },
      //重启设备
      restartDevice(deviceId) {
        this.$confirm('确认重启当前设备？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post('set/restart/' + deviceId, {}, "重启命令下发成功", "重启命令下发失败").then((data) => {
          })
        }).catch(() => {
        });
      },
      //删除设备
      clickDelete(id) {
        this.$confirm('确定要删除该设备吗？', '提示', {
          type: 'warning'
        }).then(() => {
          this.$post("device/delete/" + id, {}, "删除成功", "删除失败")
            .then(() => {
              this.getData();
            });
        }).catch(() => {
        });
      },
      //定时刷新设备的在线状态
      statusTask() {
        if (!this.intervalid) {
          this.intervalid = setInterval(() => {
            this.getDeviceStatus();
          }, 20000);
        }
      },
      //获取设备的在线状态
      getDeviceStatus() {
        if (this.deviceList.length > 0) {
          let deviceIds = [];
          this.deviceList.forEach((device) => {
            deviceIds.push(device.deviceId);
          });
          axios.post("device/get/lineStatus", deviceIds).then((res) => {
            let data = res.data;
            if (res.status == 200) {
              this.deviceList.forEach((device) => {
                if (data[device.deviceId] == 0) {
                  this.$set(device, 'lineStatus', "离线");//在数组中增加属性
                } else if (data[device.deviceId] == 1) {
                  this.$set(device, 'lineStatus', "在线");//在数组中增加属性
                } else {
                  this.$set(device, 'lineStatus', "未知");//在数组中增加属性
                }
              });
            } else {
              this.deviceList.forEach((device) => {
                this.$set(device, 'lineStatus', "未知");//在数组中增加属性
              });
            }
          });
        }
      },
      //添加设备
      deviceAdd() {
        if (/[+@#><!?，。》《、；‘’;'`~\$%\^&\*]+/g.test(this.addDevice.deviceId)) {
          this.$message.error('请输入正确的设备id，由英文字母、数字组成');
          return;
        }
        if (/[+@#><!?，。》《、；‘’;'`~\$%\^&\*]+/g.test(this.addDevice.deviceName)) {
          this.$message.error('请输入正确的设备名称，由汉字、数字、英文字母组成');
          return;
        }
        this.$refs['addDevice'].validate((valid) => {
          if (valid) {
            this.$post("device/add", this.addDevice, "创建成功", "创建失败");
            this.addDevice = {
              deviceId: '',
              deviceName: '',
              deviceForm: '',
              devType: ''
            };
            this.runningAddDevice = false;
            this.getData();
          }
        })
      },
      //获取设备列表
      getData() {
        if (this.query.deviceName) {
          if (/[+@#><!?，。》《、；‘’;'`~\$%\^&\*]+/g.test(this.query.deviceName)) {
            this.$message.error('请输入正确的设备名称/ID');
            return;
          }
        }
        this.listLoading = true;

        this.$post("device/query", this.query).then((data) => {
          this.deviceList = data.data.list;
          this.count = data.data.count;
          this.getDeviceStatus();
          setTimeout(() => {
            this.listLoading = false;
          }, 500);
        }).catch((err) => {
          this.listLoading = false;
          this.deviceList = [];
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
      //搜索设备列表
      clearData() {
        this.query = {
          page: 1,
          size: 10,
          deviceForm: '',
          devType: '',
          deviceName: ''
        };
        this.lineStatus = '';
        this.getData();
      },
      formatterArray(row, column) {
        return row[column.property].join(",")
      },
      formatterOperations(row, column) {
        let str = '';
        if (row && row.operations) {
          for (let item of row.operations) {
            str += item === 'U' ? '联通' : item === 'M' ? '移动' : '电信';
            str += ',';
          }
          return str.substr(0, str.length - 1);
        }
      },
      formatterLatLon(row, column) {
        return row.devicePos && row.devicePos.longitude ? (row.devicePos.longitude + "," + row.devicePos.latitude) : "";
      },
      //格式化内容   有数据就展示，没有数据就显示--
      formatterAddress(row, column) {
        return row[column.property] && row[column.property] !== "null" ? row[column.property] : '--';
      },
      areaChange(value) {
        this.query.areaCode = value[1];
      },
      //mqtt消息
      initMqtt() {
        let _this = this;
        this.client = new Paho.MQTT.Client("117.48.193.122", Number(38083), sessionStorage.getItem("clientId"));//建立客户端实例

        this.client.onConnectionLost = this.onConnectionLost;//注册连接断开处理事件
        this.client.onMessageArrived = this.onMessageArrived;//注册消息接收处理事件

        this.client.connect({
          onSuccess: _this.onConnect,
          userName: "fc69d333449f4c728bd5a4c55e0c618c",
          password: "1ec3a8887f694925916a3cd6b189df59"
        });//连接服务器并注册连接成功处理事件
      },
      onConnect() {
        console.log("onConnected");
      },
      changeSub(msg) {
        console.log('订阅 = ' + this.deviceId);
        this.client.subscribe("fc69d333449f4c728bd5a4c55e0c618c/WEBUI/" + this.deviceId);//订阅主题
      },
      onMessageArrived(message) {
        console.log("收到消息message.payloadString = " + message.payloadString);
        let msg = JSON.parse(message.payloadString).metaInfo;
        let data = JSON.parse(msg.content);
//        console.log(data);
        if (this.deviceId) {
          if (this.deviceId === data.deviceId) {
            this.retrievalRate = data.rate;
          }
        }
      },
      onConnectionLost(responseObject) {
//        if (responseObject.errorCode !== 0) {
//          console.log("连接已断开onConnectionLost:" + responseObject.errorMessage);
//          this.initMqtt();
//        }
      },
      //用于生成uuid
      S4() {
        return (((1 + Math.random()) * 0x10000) | 0).toString(16).substring(1);
      },
      guid() {
        return (this.S4() + this.S4() + "-" + this.S4() + "-" + this.S4() + "-" +
          this.S4() + "-" + this.S4() + this.S4() + this.S4());
      }
    },
    mounted() {
      sessionStorage.setItem("active", 1);
      let clientId = this.guid();
      let client_id = sessionStorage.getItem("clientId");
      if (client_id === null || client_id === undefined || client_id === '') {
        sessionStorage.setItem("clientId", clientId);
      }
      this.getData();
//      this.initMqtt();
      this.getDeviceTypeAndForm();
      this.statusTask();
    },
    components: {
      baseInfo,
      setParam,
      networkData,
      deviceStatus,
      update,
      logList,
      terminal
    }
  }
</script>

<style scoped>
  .setting, .setting:visited, .setting:link {
    width: 40px;
    height: 30px;
    background: url("../../assets/img/icon_set.png") no-repeat;
    background-position: center;
    border: none;
  }

  .setting:hover, .setting:active, .setting:focus {
    background-image: none;
    font-size: 0;
  }

  .setting:hover:before {
    color: #41DC7E;
    content: '设置';
    font-size: 14px;
  }

  .status, .status:visited, .status:link {
    width: 40px;
    height: 30px;
    background: url("../../assets/img/icon_status.png") no-repeat;
    background-position: center;
    border: none;
  }

  .status:hover, .status:active, .status:focus {
    background-image: none;
    font-size: 0;
  }

  .status:hover:before {
    color: #41DC7E;
    content: '状态';
    font-size: 14px;
  }

  .zmdata, .zmdata:visited, .zmdata:link {
    width: 40px;
    height: 30px;
    background: url("../../assets/img/icon_data.png") no-repeat;
    background-position: center;
    border: none;
  }

  .zmdata:hover, .zmdata:active, .zmdata:focus {
    background-image: none;
    font-size: 0;
  }

  .zmdata:hover:before {
    color: #41DC7E;
    content: '侦码';
    font-size: 14px;
  }

  .glmb, .glmb:visited, .glmb:link {
    width: 40px;
    height: 30px;
    background: url("../../assets/img/icon_manager.png") no-repeat;
    background-position: center;
    border: none;
  }

  .glmb:hover, .glmb:active, .glmb:focus {
    background-image: none;
    font-size: 0;
  }

  .glmb:hover:before {
    color: #41DC7E;
    content: '更多';
    font-size: 14px;
  }

  .btn_green, .btn_green:visited, .btn_green:link {
    width: 100%;
    text-align: center;
    color: #41DC7E;
    font-size: 14px;
    background-color: #ffffff;
    border: none;
  }

  .btn_green:hover, .btn_green:active, .btn_green:focus {
    color: #ffffff;
    background-color: #41DC7E;
  }

  .btn_green[disabled] {
    color: #bbb;
    background-color: #fff;
  }
</style>
