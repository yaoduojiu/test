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
            <a-form-item label="订单编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['orderCode']" placeholder="请输入订单编号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="混料批次号" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['blendLotId']" placeholder="请输入混料批次号" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="产成品物料编码" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-popup
                v-decorator="['materialCode']"
                :trigger-change="true"
                org-fields="code,name,id"
                dest-fields="materialCode,materialName,materialId"
                code="material_product"
                @callback="popupCallback"
                />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="产品名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialName']" placeholder="请输入产品名称" disabled></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item hidden label="物料ID" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['materialId']" placeholder="请输入物料ID" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['status']" :trigger-change="true" dictCode="order_status" placeholder="请选择状态" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="生产数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['qty']" placeholder="请输入生产数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="实际数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['actualQty']" placeholder="请输入实际数量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="产线" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['lineId']" :trigger-change="true" dictCode="line,line_name,id" placeholder="请选择产线" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="投料订单" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['orderId']" :trigger-change="true" dictCode="hh_order,code,id" placeholder="请选择投料订单" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="开始时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择开始时间" v-decorator="['startTime']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="结束时间" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-date placeholder="请选择结束时间" v-decorator="['endTime']" :trigger-change="true" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="蛋白配方名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['dbFormulaname']" placeholder="请输入蛋白配方名称" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="蛋白配方重量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['dbFormulaweiht']" placeholder="请输入蛋白配方重量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="蛋白总批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['dbTotalbatches']" placeholder="请输入蛋白总批次" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="蛋白打包状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['dbStatus']" :trigger-change="true" dictCode="pack_status" placeholder="请选择蛋白打包状态" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂1配方名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['db1Formulaname']" placeholder="请输入添加剂1配方名称" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂1配方重量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['db1Formulaweiht']" placeholder="请输入添加剂1配方重量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂1总批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['db1Totalbatches']" placeholder="请输入添加剂1总批次" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂1打包状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['db1Status']" :trigger-change="true" dictCode="pack_status" placeholder="请选择添加剂1打包状态" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂2配方名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input v-decorator="['db2Formulaname']" placeholder="请输入添加剂2配方名称" ></a-input>
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂2配方重量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['db2Formulaweiht']" placeholder="请输入添加剂2配方重量" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂2总批次" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number v-decorator="['db2Totalbatches']" placeholder="请输入添加剂2总批次" style="width: 100%" />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item label="添加剂2打包状态" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <j-dict-select-tag type="list" v-decorator="['db2Status']" :trigger-change="true" dictCode="pack_status" placeholder="请选择添加剂2打包状态" />
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
    name: "BlendLotModal",
    components: {
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
        },
        url: {
          add: "/orderBlendLot/blendLot/add",
          edit: "/orderBlendLot/blendLot/edit",
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
          this.form.setFieldsValue(pick(this.model,'orderCode','blendLotId','materialCode','materialName','materialId','status','qty','actualQty','lineId','orderId','startTime','endTime','dbFormulaname','dbFormulaweiht','dbTotalbatches','dbStatus','db1Formulaname','db1Formulaweiht','db1Totalbatches','db1Status','db2Formulaname','db2Formulaweiht','db2Totalbatches','db2Status','createBy','createTime'))
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
        this.form.setFieldsValue(pick(row,'orderCode','blendLotId','materialCode','materialName','materialId','status','qty','actualQty','lineId','orderId','startTime','endTime','dbFormulaname','dbFormulaweiht','dbTotalbatches','dbStatus','db1Formulaname','db1Formulaweiht','db1Totalbatches','db1Status','db2Formulaname','db2Formulaweiht','db2Totalbatches','db2Status','createBy','createTime'))
      },


    }
  }
</script>