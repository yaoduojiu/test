<template>
  <j-modal
    title="盘点"
    width="1200px"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    <div>
      <a-row>
        <a-col class="mb-10" :span="8">
          <div>
            批次编码：{{ model.code ? model.code : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            物料编码：{{ model.materialCode ? model.materialCode : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            物料名称：{{ model.materialName ? model.materialName : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            库位：{{ model.locationCode ? model.locationCode : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            在库数量：{{ model.inQty ? model.inQty : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            盘点数量：{{ model.qty ? model.qty : '--' }}
          </div>
        </a-col>
        <a-col class="mb-10" :span="8">
          <div>
            盘点结果：{{ model.status_dictText ? model.status_dictText : '--' }}
          </div>
        </a-col>
      </a-row>
      <a-form :form="form">
        <a-row>
          <a-col :span="24">
            <a-form-item label="盘点数量" :labelCol="labelCol" :wrapperCol="wrapperCol">
              <a-input-number
                :precision="0"
                :min="0"
                :max="999999999"
                 v-decorator="[
                  'qty',
                  { rules: [{ required: true, message: '请输入盘点数量' }] },
                ]"
              placeholder="请输入盘点数量" style="width: 100%" />
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </div>
  </j-modal>
</template>

<script>

  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'

  export default {
    name: "InventoryDetailModal",
    components: {
    },
    props:{
      model:{
        type:Object,
        required:true,
        default: function() {
          return {}
        }
      }
    },
    data () {
      return {
        form: this.$form.createForm(this),
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        visible: false,
        confirmLoading: false,
        validatorRules: {
        },
        url: {
          add: "/inventory/inventory/addInventoryDetail",
          edit: "/inventory/inventory/editInventoryDetail",
        }

      }
    },
    created () {
      
    },
    mounted() {
    },
    methods: {
      close () {
        this.$emit('close');
        this.form.setFieldsValue({
          qty: ''
        })
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '/inventory/inventory/editInventoryDetail';
            let method = 'put';
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
              this.form.setFieldsValue({
                qty: ''
              })
              that.confirmLoading = false;
              that.close();
            })
          }

        })
      },
      handleCancel () {
        this.close()
      }
    }
  }
</script>
<style scoped>
  .mb-10 {
    margin-bottom: 10px;
  }
</style>
