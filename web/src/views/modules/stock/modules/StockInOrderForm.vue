<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="12">
            <a-form-item label="入库单编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['code', validatorRules.code]" placeholder="请输入入库单编码"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
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
          <a-col :span="12">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="物料SN" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['batchSn']" placeholder="请输入物料SN" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="入库类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['stockInType', validatorRules.stockInType]" :trigger-change="true" dictCode="stock_in_type" placeholder="请选择入库类型" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="仓库编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
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
          <a-col :span="12">
            <a-form-item label="仓库名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['warehouseName']" placeholder="请输入仓库名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="库位编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['locationCode']"
                :trigger-change="true"
                org-fields="id,code"
                dest-fields="locationId,locationCode"
                code="bb_hh_location"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="批次编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['batchCode']" placeholder="请输入批次编号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="盘点单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['inventoryCode']" placeholder="请输入盘点单号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="盘点单号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['inQty']" placeholder="请输入盘点单号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="入库时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择入库时间" v-decorator="['inTime']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="入库操作人" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['inOper', validatorRules.inOper]" placeholder="请输入入库操作人"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item label="入库状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['inStatus', validatorRules.inStatus]" :trigger-change="true" dictCode="stock_in_status" placeholder="请选择入库状态" />
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
  import JFormContainer from '@/components/jeecg/JFormContainer'
  import JDate from '@/components/jeecg/JDate'
  import JDictSelectTag from "@/components/dict/JDictSelectTag"

  export default {
    name: 'StockInOrderForm',
    components: {
      JFormContainer,
      JDate,
      JDictSelectTag,
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
          code: {
            rules: [
              { required: true, message: '请输入入库单编码!'},
            ]
          },
          materialCode: {
            rules: [
              { required: true, message: '请输入物料编码!'},
            ]
          },
          stockInType: {
            rules: [
              { required: true, message: '请输入入库类型!'},
            ]
          },
          warehouseCode: {
            rules: [
              { required: true, message: '请输入仓库编码!'},
            ]
          },
          inOper: {
            rules: [
              { required: true, message: '请输入入库操作人!'},
            ]
          },
          inStatus: {
            rules: [
              { required: true, message: '请输入入库状态!'},
            ]
          },
        },
        url: {
          add: "/stock/stockInOrder/add",
          edit: "/stock/stockInOrder/edit",
          queryById: "/stock/stockInOrder/queryById"
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
          this.form.setFieldsValue(pick(this.model,'code','materialCode','materialName','batchSn','stockInType','warehouseCode','warehouseName','locationCode','batchCode','inventoryCode','inQty','inTime','inOper','inStatus'))
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
        this.form.setFieldsValue(pick(row,'code','materialCode','materialName','stockInType','warehouseCode','warehouseName','locationCode','batchCode','inventoryCode','inQty','inTime','inOper','inStatus'))
      },
    }
  }
</script>