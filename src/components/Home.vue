<template>
<!-- 头部区域 -->
  <el-container  class="home-container">
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="图片">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
      </el-header>
    <!-- 页面主体 -->
    <el-container>
      <!-- 侧边栏  -->
      <!-- 动态绑定侧边栏高度 -->
      <el-aside :width="isCollapse? '64px':'200px'">
        <div class="toggle-button"  @click="toggleCollapse">|||</div>
        <!-- unique-opened="true"控制菜单项是否指展开一项      router开启路由属性       :default-active="$route.path"可以调试一下-->
        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409EFF" unique-opened :collapse='isCollapse' :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单 -->
          <!-- v-for将数据渲染到网页，index，key动态绑定唯一值，item.id+''将id转化为字符串 -->
          <el-submenu :index="item.id + ''" v-for='item in menuList' :key='item.id'>
            <!-- 一级菜单模板区域 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本区域 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' + subItem.path"  v-for="subItem in item.children" :key="subItem.id"  @click="saveNavState('/' + subItem.path)">
              <template slot="title">
              <!-- 图标 -->
              <i class="el-icon-menu"></i>
              <!-- 文本区域 -->
              <span>{{subItem.authName}}</span>
            </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menuList: [],
      iconsObj: {
        125: 'icon-font icon-user',
        103: 'icon-font icon-tijikongjian',
        101: 'icon-font icon-shangpin',
        102: 'icon-font icon-danju',
        145: 'icon-font icon-baobiao'
      },
      isCollapse: false,
      // 被激活的路由链接地址
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear() // 退出后清除token
      this.$router.push('/login') // 跳转回登录界面
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    // 点击按钮菜单折叠,collapse属性
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保存被激活的路由链接
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
    }
  }
}
</script>

<style lang="less" scoped>
.home-container{
  height:100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;//内容左右贴边对齐
  padding-left: 0;//图片贴边
  align-items: center;//按钮居中
  color:#fff;
  font-size: 20px;
  div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
  }
}
.el-aside{
  background-color: #333744;
  .el-menu {
    border-right:none;
  }
}
.el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin-left:10px;
}
.toggle-button{
  background-color: #4A5064;
  font-size:10px;
  line-height: 24px;
  color:#fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
