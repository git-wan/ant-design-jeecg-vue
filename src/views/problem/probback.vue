<template>
  <div>
    <a-form :form="form" style="max-width: 500px; margin: 40px auto 0;">
      <a-form-item
        label="回应人"
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        hasFeedback>
        <a-input placeholder="请输入回应人" v-decorator="[ 'backer', {rules: [{required: true, message: '请录入记录人！'}]}]" />
      </a-form-item>
      <a-form-item
        label="回应日期"
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        hasFeedback>
        <a-date-picker placeholder="请输入记录时间" v-decorator="['backdate', {rules: [{required: true, message: '记录时间不能为空！'}]}]" />
      </a-form-item>
      <a-form-item
        label="回应说明"
        :labelCol="labelCol"
        :wrapperCol="wrapperCol"
        hasFeedback>
        <a-textarea placeholder="请输入回应说明" v-decorator="['backnote', {rules: [{required: true, message: '请输入回应说明！'}]}]" :autosize="{ minRows: 3, maxRows: 6 }"/>
      </a-form-item>

     <!-- <a-form-item :wrapperCol="{span: 19, offset: 5}">
        <a-button type="primary" @click="nextStep">下一步</a-button>
      </a-form-item>-->
    </a-form>
  </div>
</template>

<script>
  export default {
    name: "Step1",
    data () {
      return {
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        form: this.$form.createForm(this),
      }
    },
    methods: {
      a(){
        alert(1)
      }
    /*  nextStep () {
        this.form.validateFields((err, values) => {
          if(err){
            this.$message.warning("请输入正确的数据！");
          }else{
            this.$emit('nextStep')
          }
        })
      }*/
    }
  }
</script>

<style scoped>

</style>