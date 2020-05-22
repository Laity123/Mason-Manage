<template>
  <el-container class="home_container">
    <el-header>
      <div>
        <span>MASON-MANAGE.</span>
      </div>
      <el-button type="info" @click="LoginOut">
        <strong>EXIT</strong>
      </el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          background-color="#353b48"
          text-color="#fff"
          active-text-color="#8c7ae6"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]" class="iconfont"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/'+subItem.path + ''"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="saveNavState('/'+subItem.path)"
            >
              <template slot="title">
                <i class="el-icon-tickets"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: "",
  data() {
    return {
      // 左侧菜单数据
      menuList: [],
      iconsObj: {
        "125": "el-icon-user",
        "103": "el-icon-eleme",
        "101": "el-icon-goods",
        "102": "el-icon-document",
        "145": "el-icon-wallet"
      },
      isCollapse: false,
      activePath:''
    };
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    LoginOut() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    async getMenuList() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.menuList = res.data;
    },
    // 折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse;
    },
    // 保存链接的激活状态
    saveNavState(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath = activePath
    }
  }
};
</script>

<style lang='less' scoped>
.home_container {
  height: 100%;
}
.el-header {
  padding-left: 5px;
  background-color: #2f3640;
  display: flex;
  justify-content: space-between;
  align-items: center;
  > div {
    color: #8c7ae6;
    font-size: 20px;
    font-weight: 700;
  }
}
.el-aside {
  background-color: #353b48;
  .el-menu {
    border-right: 0;
  }
}
.el-main {
  background-color: #f1f3f4;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #576574;
  font-size: 16px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  letter-spacing: 0.1em;
  cursor: pointer;
}
</style>
