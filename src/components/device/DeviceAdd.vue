<template>
  <div>
    <section class="content-header text-left">
      <el-breadcrumb separator=">">
        <el-breadcrumb-item><i class="fa fa-dashboard" style="margin-right: 10px"></i>首页</el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/deviceList' }">设备列表</el-breadcrumb-item>
        <el-breadcrumb-item>添加设备</el-breadcrumb-item>
      </el-breadcrumb>
    </section>
    <section class="content">
      <el-form ref="form" :model="form" :rules="rules" label-width="150px" style="width: 50%">
        <el-form-item label="设备名称" class="item" prop="deviceName">
          <el-input v-model="form.deviceName" placeholder="请输入设备名称" :maxlength=50></el-input>
        </el-form-item>
        <el-form-item label="设备ID" class="item" prop="deviceId">
          <el-input v-model="form.deviceId" placeholder="请输入设备ID" :maxlength=50></el-input>
        </el-form-item>
        <el-form-item label="设备型号" class="item" prop="model">
          <el-input v-model="form.model" placeholder="请输入设备型号" :maxlength=50></el-input>
        </el-form-item>

        <el-form-item class="item text-left" label="设备类型" prop="devType">
          <el-select v-model="form.devType" placeholder="请选择">
            <el-option v-for="item in deviceTypes" :key="item.code" :label="item.name" :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>

        <el-form-item class="item text-left" label="设备形态" prop="deviceForm">
          <el-select v-model="form.deviceForm" placeholder="请选择">
            <el-option v-for="item in deviceForms" :key="item.code" :label="item.name" :value="item.code">
            </el-option>
          </el-select>
        </el-form-item>

        <!--
        <el-form-item label="运营商" class="item" prop="operations" required>
          <el-checkbox-group v-model="operation" class="text-left">
            <el-checkbox label="电信" name="T"></el-checkbox>
            <el-checkbox label="移动" name="M"></el-checkbox>
            <el-checkbox label="联通" name="U"></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-form-item label="网络制式" class="item" prop="networks" required>
          <el-checkbox-group v-model="form.networks" class="text-left">
            <el-checkbox label="2G" name="type"></el-checkbox>
            <el-checkbox label="3G" name="type"></el-checkbox>
            <el-checkbox label="4G" name="type"></el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        -->
        <el-form-item label="安装地址" class="demonstration" prop="areaCode">
          <el-col :span="15">
            <el-cascader expand-trigger="hover" :options="provinceList" v-model="selectedOptions2"
                         :props="props" @change="areaChange">
            </el-cascader>
          </el-col>

          <el-col :span="9">
            <el-input v-model="form.detailAddress" placeholder="请输入详细地址" :maxlength=100></el-input>
          </el-col>
        </el-form-item>

        <el-form-item label="经纬度" class="item text-left" required>
          <el-col :span="9">
            <el-input v-model="form.devicePos.longitude" placeholder="" readonly>
              <template slot="prepend">经度</template>
            </el-input>
          </el-col>
          <el-col :span="9" :offset="1">
            <el-input v-model="form.devicePos.latitude" placeholder="" readonly>
              <template slot="prepend">纬度</template>
            </el-input>
          </el-col>
          <el-col :span="2" :offset="1">
            <el-button type="primary" @click="mapVisible = true">地图选址</el-button>
          </el-col>
        </el-form-item>

        <el-form-item label="高度(米)" class="item" prop="devicePos.height">
          <el-input v-model="form.devicePos.height" placeholder="请输入高度" type="number" :maxlength=2></el-input>
        </el-form-item>

        <el-form-item label="数据卡电话号码" class="item" prop="phoneNumber">
          <el-input v-model="form.phoneNumber" placeholder="请输入数据卡电话号码" type="number" :maxlength=20></el-input>
        </el-form-item>

        <!--<el-form-item label="MAC地址" class="item" prop="mac">-->
        <!--<el-input v-model="form.mac" placeholder="请输入MAC地址" :maxlength=50></el-input>-->
        <!--</el-form-item>-->

        <el-form-item label="当前频点" class="item" prop="currentFrequency">
          <el-input v-model="form.currentFrequency" placeholder="请输入当前频点" type="number" :maxlength=15></el-input>
        </el-form-item>

        <el-form-item label="固件版本" class="item" prop="deviceVersion">
          <el-input v-model="form.deviceVersion" placeholder="请输入固件版本" :maxlength=50></el-input>
        </el-form-item>

        <el-form-item label="软件版本" class="item" prop="softVersion">
          <el-input v-model="form.softVersion" placeholder="请输入软件版本" :maxlength=50></el-input>
        </el-form-item>

        <!--
        <el-form-item label="备注" class="item" prop="remake">
          <el-input v-model="form.remake" placeholder="请输入200字以内的备注" type="textarea"></el-input>
        </el-form-item>
         -->

        <el-form-item>
          <el-button type="primary" @click="submitForm('form')">立即创建</el-button>
          <el-button @click="$router.back();">取消</el-button>
        </el-form-item>
      </el-form>
    </section>
    <el-dialog title="" :visible.sync="mapVisible">
      <MapView @getLocation="getLocation"></MapView>
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

  import MapView from '../device/map'

  import json from '../../assets/city.json';

  export default {
    data() {
      return {
        props: {
          value: 'o',
          label: 'n',
          children: 'c'
        },
        selectedOptions2: [110100],
        provinceList: json,
        mapVisible: false,
        position: {},
        map: {},
        geolocation: {},
        form: {
          deviceName: '',
          deviceId: '',
          model: '',
          operations: [],
          networks: [],
          devicePos: {
            longitude: '',
            latitude: '',
            height: ''
          },
          detailAddress: '',
          remake: '',
          areaCode: '',
          deviceForm: '',
          devType: '',
          phoneNumber: '',
          mac: '',
          currentFrequency: '',
          deviceVersion: '',
          softVersion: ''
        },
        operation: [],
        rules: {
          deviceName: [
            {required: true, message: '请输入设备名称', trigger: 'blur'},
            {maxlength: 10}
          ],
          deviceId: [
            {required: true, message: '请输入设备id', trigger: 'blur', maxlength: 20}
          ],
          model: [
            {required: true, message: '请输入设备型号', trigger: 'blur', maxlength: 20}
          ]
        },
        deviceForms: [],
        deviceTypes: []
      }
    },
    created() {
      this.getDeviceTypeAndForm();
    },
    methods: {
      submitForm() {
        this.$refs['form'].validate((valid) => {
          if (valid) {
            /**
             if (this.operation.length === 0) {
              this.$message.error('请选择运营商');
              return;
            }
             if (this.form.networks.length === 0) {
              this.$message.error('请选择网络制式');
              return;
            } */
            if (this.form.devicePos.longitude.length === 0 || this.form.devicePos.latitude.length === 0) {
              this.$message.error('请选择经纬度');
              return;
            }
            /**
             let arr = [];
             for (let item of this.operation) {
              arr.push(item === '电信' ? 'T' : item === '移动' ? 'M' : 'U');
            }
             this.form.operations = [];
             this.form.operations = arr;
             */
            this.$post("device/add", this.form, "创建成功");
            this.$router.back();
          }
        });
      },
      areaChange(value) {
        this.form.areaCode = value[1];
      },
      setLocation() {
        this.form.devicePos.longitude = this.position.getLng();
        this.form.devicePos.latitude = this.position.getLat();
        this.mapVisible = false;
      },
      getLocation(pos) {
        this.position = pos;
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
      }
    },
    mounted() {
      sessionStorage.setItem("active", 1);
    },
    components: {
      MapView
    }
  }
</script>



