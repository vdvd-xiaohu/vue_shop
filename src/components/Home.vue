<template>


    <el-container class="home-container">
        <!-- 头部区域 -->
  <el-header>
      <div>
          <img src="../assets/heima.png" alt="">
          <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
      
      </el-header>

  <el-container>
      <!-- 侧边栏区域 -->
    <el-aside :width="isCollapse ? '64px':'200px' ">
        <div class="toggle-button" @click=" getCollapse">|||</div>
        <el-menu
     
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409EFF"
      unique-opened
      :collapse="isCollapse"
      :collapse-transition = "false"
      :default-active="activePath"
      :router="true"
      
       >
      <!-- 一级菜单 -->
       <el-submenu :index="item.id+''" v-for="item in menulist" :key="item.id">
          <!-- 一级菜单的模板区域 -->
        <template slot="title">
            <!-- 一级菜单的图标区域 -->
          <i :class="iconobj[item.id]"></i>
          <!-- 一级菜单的文字区域 -->
          <span>{{ item.authName }}</span>
        </template>
        
       <!-- 二级菜单区域 -->
          <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="SaveNavState('/'+subItem.path)">
              <template slot="title">
            <!-- 二级菜单的图标区域 -->
          <i class="el-icon-menu"></i>
          <!-- 二级菜单的文字区域 -->
          <span>{{ subItem.authName }}</span>
        </template>
          </el-menu-item>
      
      </el-submenu>
     
    </el-menu>
    </el-aside>

    <!-- 主体区域 -->
    <el-main>
        <router-view></router-view>
    </el-main>
    </el-container>

  </el-container>
</template>
<script>
export default {
    methods:{
        logout(){
            window.sessionStorage.clear()
            this.$router.push('/login')
        },
      async  getMenuList() {
           const {data:res} = await this.$http.get('menus')
           if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
           //console.log(res)
           this. menulist = res.data
        },
        getCollapse() {
            this.isCollapse = !this.isCollapse
        },
        SaveNavState(activePath) {
            window.sessionStorage.setItem('activePath',activePath)
            this.activePath = activePath
        }
    },
    created() {
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    data() {
        return {
            menulist:[],
            iconobj:{
                '125':'iconfont icon-user',
                '103':'iconfont icon-tijikongjian',
                '101':'iconfont icon-shangpin',
                '102':'iconfont icon-danju',
                '145':'iconfont icon-baobiao'

            },
            isCollapse:false,
            activePath:'',
        }
    }
}
</script>
<style scoped>
.el-header {
    background-color: #373D41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    
}
.el-header div {
    display: flex;
    align-items: center;
}
.el-header div span {
    margin-left: 15px;
}
.el-aside {
    background-color: #333744;
}
.el-main {
    background-color: #eaedf1;
}
.home-container {
    height: 100%;

}
.iconfont {
    margin-right: 10px;
}
.el-aside .el-menu {
    border-right: 0;
}
.toggle-button{
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>
