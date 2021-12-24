<template>
  <a-spin :spinning="confirmLoading">
    <j-form-container :disabled="formDisabled">
      <a-form :form="form" slot="detail">
        <a-row>
          <a-col :span="24">
            <a-form-item label="订单编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-search-select-tag v-decorator="['orderId', validatorRules.orderId]" dict="hh_order,code,id"  />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="混料批次号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['blendLotId']" placeholder="请输入混料批次号"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="原材料批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['batchCode', validatorRules.batchCode]"
                :trigger-change="true"
                org-fields="code,id,material_id,material_code,material_name"
                dest-fields="batchCode,batchId,materialId,materialCode,materialName"
                code="batch_popup"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialCode']" placeholder="请输入物料编码" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['feedQty', validatorRules.feedQty]" placeholder="请输入投料数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['type']" :trigger-change="true" dictCode="feed_type" placeholder="请选择投料类型" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料工位" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['stationCode']" placeholder="请输入投料工位"  ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料仓号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['stationWarehouseCode']" placeholder="请输入投料仓号"  ></a-input>
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
    name: 'FeedRecordForm',
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
          orderId: {
            rules: [
              { required: true, message: '请输入订单编号!'},
            ]
          },
          batchCode: {
            rules: [
              { required: true, message: '请输入原材料批次!'},
            ]
          },
          feedQty: {
            rules: [
              { required: true, message: '请输入投料数量!'},
              { pattern: /^-?\d+\.?\d*$/, message: '请输入数字!'},
            ]
          },
        },
        url: {
          add: "/orderFeedRecord/feedRecord/add",
          edit: "/orderFeedRecord/feedRecord/edit",
          queryById: "/orderFeedRecord/feedRecord/queryById"
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
          this.form.setFieldsValue(pick(this.model,'orderId','blendLotId','batchCode','materialCode','materialName','feedQty','type','stationCode','stationWarehouseCode','createBy','createTime','outOrderCode'))
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
        this.form.setFieldsValue(pick(row,'orderId','blendLotId','batchCode','materialCode','materialName','feedQty','type','stationCode','stationWarehouseCode','createBy','createTime','outOrderCode'))
      },
    }
  }
</script>