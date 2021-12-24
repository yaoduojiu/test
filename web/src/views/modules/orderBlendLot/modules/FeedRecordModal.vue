<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row>
          <a-col :span="24">
            <a-form-item label="混料批次号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['blendLotId']" placeholder="请输入混料批次号" ></a-input>
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
              <j-popup
                v-decorator="['materialCode']"
                :trigger-change="true"
                org-fields="code,name,id"
                dest-fields="materialCode,materialName,materialId"
                code="bb_hh_material"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="物料名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入物料名称" disabled></a-input>
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
              <a-input v-decorator="['stationCode']" placeholder="请输入投料工位" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料仓号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['stationWarehouseCode']" placeholder="请输入投料仓号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="出库单编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['outOrderCode']" placeholder="请输入出库单编码" ></a-input>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import { validateDuplicateValue } from '@/utils/util'

  export default {
    name: "FeedRecordModal",
    components: {
    },
    props:{
      mainId:{
        type:String,
        required:false,
        default:''
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        title:"操作",
        width:800,
        visible: false,
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
          add: "/orderBlendLot/blendLot/addFeedRecord",
          edit: "/orderBlendLot/blendLot/editFeedRecord",
        }

      }
    },
    created () {
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
          this.form.setFieldsValue(pick(this.model,'orderCode','orderId','blendLotId','batchCode','batchId','materialCode','materialName','materialId','feedQty','type','stationCode','stationWarehouseCode','createBy','createTime','updateBy','updateTime','sysOrgCode','outOrderCode'))
        })
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
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
            formData['orderId'] = this.mainId
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
              that.close();
            })
          }

        })
      },
      handleCancel () {
        this.close()
      },
      popupCallback(row){
        this.form.setFieldsValue(pick(row,'orderCode','orderId','blendLotId','batchCode','batchId','materialCode','materialName','materialId','feedQty','type','stationCode','stationWarehouseCode','createBy','createTime','updateBy','updateTime','sysOrgCode','outOrderCode'))
      },


    }
  }
</script>
