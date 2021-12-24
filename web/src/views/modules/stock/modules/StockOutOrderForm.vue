<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['materialCode', validatorRules.materialCode]"
                :trigger-change="true"
                org-fields="id,code,name"
                dest-fields="materialId,materialCode,materialName"
                code="bb_hh_material"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料SN" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['batchSn']" placeholder="请输入物料SN" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="出库类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['outType', validatorRules.outType]" :trigger-change="true" dictCode="stock_out_type" placeholder="请选择出库类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="仓库编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['warehouseCode', validatorRules.warehouseCode]"
                :trigger-change="true"
                org-fields="id,code,name"
                dest-fields="warehouseId,warehouseCode,warehouseName"
                code="bb_hh_warehouse"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="仓库名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['warehouseName']" placeholder="请输入仓库名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="库位编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['locationCode', validatorRules.locationCode]"
                :trigger-change="true"
                org-fields="id,code"
                dest-fields="locationId,locationCode"
                code="bb_hh_location"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="批次编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['batchCode', validatorRules.batchCode]" placeholder="请输入批次编码"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="领料单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['pickCode']" placeholder="请输入领料单号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="盘点单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['inventoryCode']" placeholder="请输入盘点单号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="出库数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['outQty']" placeholder="请输入出库数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="出库时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择出库时间" v-decorator="['outTime']" :trigger-change="true" :show-time="true" date-format="YYYY-MM-DD HH:mm:ss" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="出库操作人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['outUser']" placeholder="请输入出库操作人"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col v-if="showFlowSubmitButton" :span="24" style="text-align: center">
            <a-button @click="submitForm">提 交</a-button>
          </a-col>
        </a-row>
      </a-form>
    </j-form-container>
  </a-spin>
</template>

<script>

  import { httpAction, getAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: 'StockOutOrderForm',
    components: {
    },
    props: {
      //流程表单data
      formData: {
        type: Object,
        default: ()=>{},
        required: false
      },
      //表单模式：true流程表单 false普通表单
      formBpm: {
        type: Boolean,
        default: false,
        required: false
      },
      //表单禁用
      disabled: {
        type: Boolean,
        default: false,
        required: false
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        confirmLoading: false,
        validatorRules: {
          materialCode: {
            rules: [
              { required: true, message: '请输入物料编码!'},
            ]
          },
          outType: {
            rules: [
              { required: true, message: '请输入出库类型!'},
            ]
          },
          warehouseCode: {
            rules: [
              { required: true, message: '请输入仓库编号!'},
            ]
          },
          locationCode: {
            rules: [
              { required: true, message: '请输入库位编号!'},
            ]
          },
          batchCode: {
            rules: [
              { required: true, message: '请输入批次编码!'},
            ]
          },
        },
        url: {
          add: "/stock/stockOutOrder/add",
          edit: "/stock/stockOutOrder/edit",
          queryById: "/stock/stockOutOrder/queryById"
        }
      }
    },
    computed: {
      formDisabled(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return false
          }
          return true
        }
        return this.disabled
      },
      showFlowSubmitButton(){
        if(this.formBpm===true){
          if(this.formData.disabled===false){
            return true
          }
        }
        return false
      }
    },
    created () {
      //如果是流程中表单，则需要加载流程表单data
      this.showFlowData();
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'code','materialCode','materialName','batchSn','status','outType','warehouseCode','warehouseName','locationCode','batchCode','pickCode','inventoryCode','outQty','outTime','outUser','createBy','createTime','updateBy','updateTime'))
        })
      },
      //渲染流程表单数据
      showFlowData(){
        if(this.formBpm === true){
          let params = {id:this.formData.dataId};
          getAction(this.url.queryById,params).then((res)=>{
            if(res.success){
              this.edit (res.result);
            }
          });
        }
      },
      submitForm () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            console.log("表单提交数据",formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
            })
          }
         
        })
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'code','materialCode','materialName','status','outType','warehouseCode','warehouseName','locationCode','batchCode','pickCode','inventoryCode','outQty','outTime','outUser','createBy','createTime','updateBy','updateTime'))
      },
    }
  }
</script>