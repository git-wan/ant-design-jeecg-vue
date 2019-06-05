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
          label="提问人"
          hasFeedback >
          <a-input placeholder="请输入提问人" v-decorator="['problemhuman', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 10,message: '最大长度为10！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题类型"
          hasFeedback >
          <a-input placeholder="请输入问题类型" v-decorator="['problemtype', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 10,message: '最大长度为10！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="提问时间"
          hasFeedback >
          <a-date-picker placeholder="提问时间" v-decorator="['probdate', {rules: [{required: true, message: '请输入负责人名称！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="应用系统"
          hasFeedback >
          <a-input placeholder="请输入应用系统" v-decorator="['application', {rules: [{required: true, message: '请输入负责人名称！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="应用系统"
          hidden=true
          hasFeedback >
          <a-input placeholder="请输入应用系统" v-decorator="['problemstatus', {initialValue:'未解决'}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="问题描述"
          hasFeedback >
          <a-textarea placeholder="请输入问题描述" v-decorator="['problemnote', {rules: [{required: true, message: '请输入负责人名称！'}]}]" :autosize="{ minRows: 3, maxRows: 6 }"/>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="记录人"
          hasFeedback >
          <a-input placeholder="请输入记录人" v-decorator="[ 'recorder', {rules: [{required: true, message: '请录入记录人！'}]}]" />  <!--, {max: 10,message: '最大长度为10！'}-->
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="记录时间"
          hasFeedback >
          <a-date-picker placeholder="请输入记录时间" v-decorator="['indate', {rules: [{required: true, message: '记录时间不能为空！'}]}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="备注"
          hasFeedback >
          <a-textarea placeholder="请输入备注" v-decorator="['note', {rules: [{required: true, message: '请输入负责人名称！'}]}]" :autosize="{ minRows: 3, maxRows: 6 }"/>
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
    name: "AssInfoModal",
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
          add: "/ws/prob/add",
          edit: "/ws/prob/edit",
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
          this.form.setFieldsValue(pick(this.model,'problemhuman','problemtype','application','problemnote','recorder',"problemstatus",'note'))
          this.form.setFieldsValue({probdate:this.model.probdate?moment(this.model.probdate,'YYYY-MM-DD HH:mm:ss'):null})
          this.form.setFieldsValue({indate:this.model.indate?moment(this.model.indate,'YYYY-MM-DD HH:mm:ss'):null})
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
            formData.probdate = formData.probdate?formData.probdate.format('YYYY-MM-DD HH:mm:ss'):null;
            formData.indate = formData.indate?formData.indate.format('YYYY-MM-DD HH:mm:ss'):null;
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