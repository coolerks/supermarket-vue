<template>
  <el-container class="home-container">
    <!--    导航-->
    <el-header>
      <div>
        <span style="color: black;margin-left:20px;">
          小型超市进销存管理系统
        </span>
      </div>
      <el-dropdown>
        <div class="block">
          <el-avatar :size="50" :src="this.userInfo.avatar" style="cursor: pointer;"></el-avatar>
        </div>
        <el-dropdown-menu slot="dropdown" trigger="click">
          <el-dropdown-item>
            <span type="danger" @click="logout"><span class="el-icon-switch-button"></span> &nbsp;退出登入</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </el-header>
    <!--主体-->
    <el-container style="height: 500px;">
      <!--菜单-->
      <el-aside :width="isOpen===true?'64px':'200px'">
        <div class="toggle-btn" @click="toggleMenu">|||</div>
        <el-menu
            class="el-menu-vertical-demo"
            :collapse="isOpen"
            :router="true"
            :default-active="activePath"

            :collapse-transition="false"
            text-color="rgba(0, 0, 0)"
            unique-opened
        >
          <MenuTree :menuList="this.menuList"></MenuTree>
        </el-menu>
      </el-aside>

      <!--右边主体-->
      <el-main v-loading="loading">
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import MenuTree from "../components/MenuTree"; //引进菜单模板

export default {
  data() {
    return {
      loading: true,
      activePath: "", //激活的路径
      isOpen: false,
      menuList: {},
      userInfo: {},
      userCount: 0
    };
  },
  components: {
    MenuTree
  },
  methods: {
    /**
     *
     * 退出登入
     */
    async logout() {
      const res = await this.$confirm("此操作将退出系统, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).catch(() => {
        this.$message({
          type: "info",
          message: "已取消退出登入"
        });
      });
      if (res === "confirm") {
        LocalStorage.clearAll();
        await this.$router.push("/login");
      }
    },
    /**
     * 去系统首页
     */
    toWelcome() {
      this.$router.push("/welcome");
    },
    /**
     加载菜单数据
     */
    async getMenuList() {
      const {data: res} = await this.$http.get("system/user/findMenu");
      if (!res.success) {
        return this.$message.error("获取菜单失败:" + res.msg);
      }
      this.menuList = res.data;
    },
    async getUserCount() {
      const {data: res} = await this.$http.get("/system/loginLog/count");
      if (res.success) {
        const {data} = res;
        return this.$message.success("今日登录次数：" + data[data.length - 1]?.count);
      }
    },
    /**
     获取用户信息
     */
    async getUserInfo() {
      const {data: res} = await this.$http.get("system/user/info");
      if (!res.success) {
        return this.$message.error("获取用户信息失败:" + res.msg);
      } else {
        this.userInfo = res.data;
        this.$store.commit("setUserInfo", res.data);
      }
    },
    /**
     * 菜单伸缩
     */
    toggleMenu() {
      this.isOpen = !this.isOpen;
    },
    /**
     * 点击交流
     */
    getContact() {
      const w = window.open('about:blank');
      w.location.href = 'https://www.zykcoderman.xyz/';
    }
  },
  mounted() {
    this.getUserInfo();
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem("activePath");
    this.getUserCount();
    setTimeout(() => {
      this.loading = false;
    }, 500);

  }
};
</script>

<style>
/* 为对应的路由跳转时设置动画效果 */

.el-header {
  background-color: #ffffff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  font-size: 19px;

  padding-left: 0px;
}

.el-aside {
  background-color: #ffffff
}

.el-main {
  background-color: #eaedf1;
}

.home-container {
  width: 100%;
  height: 100% !important;
}

.toggle-btn {
  background-color: #c9c9c9 !important;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}

</style>
