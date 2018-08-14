<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/deviceList' }">设备列表</el-breadcrumb-item>
        <el-breadcrumb-item>扫频工具</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <el-form label-width="150px" label-position="left">
        <el-row style="margin-left: 20px">
          <el-col :span="11" style="text-align: left">
            <h4>扫频设置<span style="margin-left: 20px;font-size: 15px;color: #888">设备ID：{{deviceId}}</span></h4>
          </el-col>
        </el-row>
        <el-row style="margin-left: 50px">
          <el-col :span="12">
            <!--<el-form-item label="自动扫频">-->
            <!--<el-row>-->
            <!--<el-col :span="4">{{enableSweepFrequency ? "已开启" : "已关闭"}}</el-col>-->
            <!--<el-col :span="2" :offset="2">-->
            <!--<el-switch v-model="enableSweepFrequency" on-color="#13ce66" off-color="#ff4949"-->
            <!--@change="setStatusSwitch()" disabled></el-switch>-->
            <!--</el-col>-->
            <!--</el-row>-->
            <!--</el-form-item>-->
          </el-col>
        </el-row>
      </el-form>
      <el-row style="margin-left: 20px">
        <el-col :span="11" style="text-align: left">
          <h4>扫频数据</h4>
        </el-col>
      </el-row>
      <el-tabs v-model="activeItem" @tab-click="handleClick" style="margin: 10px 0 0 40px">
        <el-tab-pane v-bind:label="tab.name" v-for="tab in activeName" :key="tab.moduleID"
                     v-bind:name="tab.type" :value="tab.moduleID">
          <el-table :data="networkData" v-loading="listLoading" border class="center-block" style="margin-top: 10px">
            <el-table-column align="left" type="index" label="序号" width="65"></el-table-column>
            <el-table-column align="left" prop="earfcn" label="频点" min-width="100" max-width="200"></el-table-column>
            <el-table-column align="left" prop="pci" label="PCI" min-width="100" max-width="200"></el-table-column>
            <el-table-column align="left" prop="band" label="band" min-width="100" max-width="200"></el-table-column>
            <el-table-column align="left" prop="rssi" label="rssi" min-width="100" max-width="200"></el-table-column>
            <el-table-column align="left" prop="rsrp" label="rsrp" min-width="100" max-width="200"></el-table-column>
            <el-table-column align="left" prop="network" label="network" min-width="100" max-width="200"
                             :formatter="formatterAddress"></el-table-column>
            <el-table-column align="left" prop="frameOffset" label="帧偏移" min-width="100"
                             max-width="200"></el-table-column>
            <el-table-column align="left" prop="freqOffset" label="频偏" min-width="100"
                             max-width="200"></el-table-column>
            <el-table-column align="left" prop="plmnID" label="PlmnID" min-width="100"
                             max-width="200"></el-table-column>
            <el-table-column align="left" prop="priority" label="小区优先级" min-width="100"
                             max-width="200"></el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </section>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        networkData: [],
        enableSweepFrequency: false,
        listLoading: false,
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
        query: {
          page: 1,
          size: 10,
        },
        count: 0,
        deviceId: this.$route.query.deviceId || ''
      }
    },
    methods: {
      handleClick(tab, event) {
        this.networkData = [];
        this.getNetworkData();
      },
      pageChange(index) {
        this.query.page = index;
        this.getNetworkData();
      },
      handleSizeChange(val) {
        this.query.size = val;
        this.getNetworkData();
      },
      getModuleID() {
        for (let item of this.activeName) {
          if (this.activeItem == item.type) {
            return item.moduleID;
          }
        }
      },
      //格式化内容   有数据就展示，没有数据就显示--
      formatterAddress(row, column) {
        return row[column.property] && row[column.property] != "null" ? row[column.property] : '--';
      },
      //自动扫频设置
      setStatusSwitch() {
      },
      getNetworkData() {
        this.listLoading = true;
        this.$post('device/get/byDeviceId/' + this.deviceId, {}).then((data) => {
          setTimeout(() => {
            this.listLoading = false
          }, 500);
          if (data.code === '000000') {
            if (data.data) {
//              this.networkData = data.data.sweepFrequencyList;
              data.data.sweepFrequencyList.forEach((item) => {
                if (this.getModuleID() == item.moduleId) {
                  this.networkData.push(item);
                }
              });
            } else {
              this.networkData = [];
            }
          } else {
            this.networkData = [];
          }
        }).catch((err) => {
          this.listLoading = false;
          this.networkData = [];
          this.$message.error(err);
        });
      }
    },
    mounted() {
      sessionStorage.setItem("active", 1);
      this.getNetworkData();
    }
  }
</script>
