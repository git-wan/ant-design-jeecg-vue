<template>
  <div>
  <a-form :form="form" style="max-width: 500px; margin: 40px auto 0;">
    <a-form-item
      :labelCol="labelCol"
      :wrapperCol="wrapperCol"
      label="操作关键词"
      hasFeedback >
      <a-input placeholder="请输入操作关键词" v-decorator="['opkey', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 10,message: '最大长度为10！'}]}]" />
    </a-form-item>
    <a-form-item
      :labelCol="labelCol"
      :wrapperCol="wrapperCol"
      label="操作步骤"
      hasFeedback >
      <a-input placeholder="请输入操作步骤" v-decorator="['opstep', {rules: [{required: true, message: '请输入负责人名称！'}, {max: 10,message: '最大长度为10！'}]}]" />
    </a-form-item>
  <!--  <a-form-item
      :labelCol="labelCol"
      :wrapperCol="wrapperCol">
      <a-button :loading="loading" type="primary" @click="subStep">提交</a-button>
      <a-button style="margin-left: 8px" @click="prevStep">上一步</a-button>
    </a-form-item>-->
  </a-form>
  </div>
</template>

<script>
  import ProbModal from './modules/ProbModal'
  export default {
    name: "Step2",
    components: {

    },
    data () {
      return {
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        form: this.$form.createForm(this),
        loading: false
      }
    },
    methods: {
    /*  subStep(){
        this.$refs.jeecgDemoTabsModal
      },*/
   /*   prevStep () {
        this.$emit('prevStep')
      },
      finish () {
        this.$emit('finish')
      },*/
      toOrderList () {
        this.$router.push('/list/query-list')
      }
    }
  }
</script>
<style lang="scss" scoped>
  .information {
    line-height: 22px;

    .ant-row:not(:last-child) {
      margin-bottom: 24px;
    }
  }
  .money {
    font-family: "Helvetica Neue",sans-serif;
    font-weight: 500;
    font-size: 20px;
    line-height: 14px;
  }
</style>