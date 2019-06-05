<template>
  <a-modal
    :width="800"
    :visible="visible"
    :footer="null"
    @cancel="handleCancel"
    >

    <a-card :bordered="false">
      <a-steps class="steps" :current="current">
        <a-step title="回复"/>
        <a-step title="添加操作记录"/>
      </a-steps>
      <div class="content">
        <step1 ref="step1" v-if="current == 0"/>
        <step2 ref="step2" v-if="current == 1"/>
      </div>
      <div class="steps-action">
        <a-button
          v-if="current < steps - 1"
          type="primary"
          @click="nextStep"
        >
          下一步
        </a-button>
        <a-button
          v-if="current == steps - 1"
          type="primary"
          @click="finish"
        >
          完成
        </a-button>
        <a-button
          v-if="current>0"
          style="margin-left: 8px"
          @click="prevStep"
        >
          上一步
        </a-button>
      </div>
    </a-card>
  </a-modal>
</template>

<script>
  import Step1 from './probback'
  import Step2 from './probop'
  import pick from 'lodash.pick'
  import { httpAction, getAction, postAction } from '@/api/manage'
  export default {
    name: 'StepForm',
    components: {
      Step1,
      Step2

    },
    data() {
      return {
        description: '将一个冗长或用户不熟悉的表单任务分成多个步骤，指导用户完成。',
        current: 0,
        visible: false,
        steps:2,
        // form
        problemBackid:'',
        backs:{},
        ops:{}
      }
    },
    methods: {
      close(){
        this.visible = false;
      },
      add(id) {
        this.problemBackid=id;
        this.visible = true
      },

      handleCancel () {
        this.$emit('close');
        this.visible = false;
      },
      // handler
      nextStep() {
        this.$refs.step1.form.validateFields((err, values) => {
          if(err){
            this.$message.warning("请输入正确的数据！");
          }else{
            this.backs=values;
            if (this.current < 1) {
              this.current += 1
            }
          }
        })
      },
      prevStep() {
        if (this.current > 0) {
          this.current -= 1
        }
      },
      finish() {
        var that = this;
        this.$refs.step2.form.validateFields((err, values) => {
          if(err){
            this.$message.warning("请输入正确的数据！");
          }else{
            var httpurl = '/ws/prob/addBack';
            var dataParams = Object.assign({},this.backs,values,{id:that.problemBackid});
            dataParams.backdate = dataParams.backdate?dataParams.backdate.format('YYYY-MM-DD HH:mm:ss'):null;
            httpAction(httpurl,dataParams,'POST').then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              // that.confirmLoading = false;
              that.close();
            })
          }
        })
      }
    }
  }
</script>
<style lang="scss" scoped>
  .steps {
    max-width: 750px;
    margin: 16px auto;
  }
</style>