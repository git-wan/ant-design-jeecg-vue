<template>
  <div class="user-wrapper" :class="theme">
    <span class="action">
      <a-icon type="question-circle-o"></a-icon>
    </span>
    <header-notice class="action"/>
    <a-dropdown>
      <span class="action action-full ant-dropdown-link user-dropdown-menu">
        <a-avatar class="avatar" size="small" :src="getAvatar()"/>
        <span v-if="isDesktop()">欢迎您，{{ nickname() }}</span>
      </span>
      <a-menu slot="overlay" class="user-dropdown-menu-wrapper">
        <a-menu-item key="0">
          <router-link :to="{ name: 'account-center' }">
            <a-icon type="user"/>
            <span>个人中心</span>
          </router-link>
        </a-menu-item>
        <a-menu-item key="1">
          <router-link :to="{ name: 'account-settings' }">
            <a-icon type="setting"/>
            <span>账户设置</span>
          </router-link>
        </a-menu-item>
        <a-menu-item key="2" @click="updatePassword">
            <a-icon type="setting"/>
            <span>修改密码</span>
        </a-menu-item>
      </a-menu>
    </a-dropdown>
    <span class="action">
      <a class="logout_title" href="javascript:;" @click="handleLogout">
        <a-icon type="logout"/>
        <span v-if="isDesktop()">&nbsp;退出登录</span>
      </a>
    </span>
    <user-password ref="userPassword"></user-password>
  </div>
</template>

<script>
  import HeaderNotice from './HeaderNotice'
  import { mapActions, mapGetters } from 'vuex'
  import { mixinDevice } from '@/utils/mixin.js'
  import UserPassword from './UserPassword'
  export default {
    name: "UserMenu",
    mixins: [mixinDevice],
    components: {
      HeaderNotice,
      UserPassword
    },
    props: {
      theme: {
        type: String,
        required: false,
        default: 'dark'
      }
    },
    methods: {
      ...mapActions(["Logout"]),
      ...mapGetters(["nickname", "avatar","userInfo"]),
      getAvatar(){
        console.log('url = '+ window._CONFIG['imgDomainURL']+"/"+this.avatar())
        return window._CONFIG['imgDomainURL']+"/"+this.avatar()
      },
      updatePassword(){
        let username = this.userInfo().username;
        this.$refs.userPassword.show(username);
      },
      handleLogout() {
        const that = this
        this.$confirm({
          title: '提示',
          content: '真的要注销登录吗 ?',
          onOk() {
            return that.Logout({}).then(() => {
                window.location.href="/";
              //window.location.reload()
            }).catch(err => {
              that.$message.error({
                title: '错误',
                description: err.message
              })
            })
          },
          onCancel() {
          },
        });
      },
    }
  }
</script>

<style scoped>
  .logout_title {
    color: inherit;
    text-decoration: none;
  }
</style>