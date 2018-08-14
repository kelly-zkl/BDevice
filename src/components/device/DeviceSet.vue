<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/deviceList' }">设备列表</el-breadcrumb-item>
        <el-breadcrumb-item>设备设置</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <div class="center-block">
        <el-tabs v-model="activeName" type="card" @tab-click="handleClick"
                 style="margin-top: 15px;width: 90%;margin-left: 2%">
          <el-tab-pane v-bind:label="deviceId" name="0" disabled>
          </el-tab-pane>
          <el-tab-pane label="基本信息" name="first">
            <baseInfo v-bind:Id="Id" style="width: 85%;margin-left:4%"></baseInfo>
          </el-tab-pane>
          <el-tab-pane label="载波信息" name="second">
            <setParam v-bind:deviceId="deviceId" style="width: 85%;margin-left: 4%"></setParam>
          </el-tab-pane>
        </el-tabs>
      </div>
    </section>
  </div>
</template>
<script>

  import baseInfo from '../device/BaseInfo'
  import setParam from '../device/SetParam'

  export default {
    data() {
      return {
        activeName: 'first',
        Id: this.$route.query.Id || '',
        deviceId: this.$route.query.deviceId || ''
      }
    },
    methods: {
      handleClick(tab, event) {
//        console.log(tab, event);
      }
    },
    mounted() {
      sessionStorage.setItem("active", 1);
    },
    components: {
      baseInfo,
      setParam
    }
  }
</script>
