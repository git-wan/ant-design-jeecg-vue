<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭">
    
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性编号"
          hasFeedback >
          <a-input placeholder="请输入属性编号" v-decorator="['propertyno', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 20,message: '最大长度为10！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性名称"
          hasFeedback >
          <a-input placeholder="请输入属性名称" v-decorator="['propertyname', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 20,message: '最大长度为10！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性操作"
          hasFeedback >
          <a-input placeholder="请输入属性操作" v-decorator="['propertyop']"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性标志"
          hasFeedback >
          <a-input  v-decorator="[ 'propertymark', {rules: [{required: true, message: '请录入属性标志！'}]}]" />  <!--, {max: 10,message: '最大长度为10！'}-->
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性取值类型"
          hasFeedback >
          <a-input  v-decorator="[ 'propertyvaluetype', {rules: [{required: true, message: '请录入属性取值类型！'}]}]" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性值正常标志"
          hasFeedback >
          <a-input  v-decorator="[ 'propertyflag']" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性值最大值"
          hasFeedback >
          <a-input  v-decorator="[ 'porpertymax']" />
        </a-form-item>

        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="属性值最小值"
          hasFeedback >
          <a-input  v-decorator="[ 'propertymin']" />
        </a-form-item>

      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "PropModal",
    data () {
      return {
        title:"操作",
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
        form: this.$form.createForm(this),
        validatorRules:{
        },
        url: {
          add: "/ws/property/add",
          edit: "/ws/property/edit",
        },
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
          this.form.setFieldsValue(pick(this.model,'propertyno','propertyname','propertyop','propertymark','propertyvaluetype','propertyflag','porpertymax','propertymin'))
        });
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
            console.log(formData)
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
    }
  }
</script>
<style scoped>
</style>