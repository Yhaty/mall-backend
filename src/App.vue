<template>
  <div class="layout">
    <!-- 是否展示菜单，实现单独展示登录界面 -->
    <el-container v-if="state.showMenu" class="container">
      <el-container class="container">
        <el-aside class="aside">
          <!--系统名称+logo-->
          <div class="head">
            <div>
              <img src="//s.weituibao.com/1582958061265/mlogo.png" alt="logo" />
              <span>vue3 admin</span>
            </div>
          </div>
          <!--一条为了美观的线条-->
          <div class="line" />
          <!-- 菜单 -->
          <el-menu
            background-color="#222832"
            text-color="#fff"
            :router="true"
            :default-openeds="state.defaultOpen"
            :default-active="state.currentPath"
          >
            <!-- 一级菜单Dashboard -->
            <el-sub-menu index="1">
              <template #title>
                <span>Dashboard</span>
              </template>
              <el-menu-item-group>
                <el-menu-item index="/"
                  ><el-icon><Odometer /></el-icon>首页</el-menu-item
                >
                <el-menu-item index="/add"
                  ><el-icon><Plus /></el-icon>添加商品</el-menu-item
                >
              </el-menu-item-group>
            </el-sub-menu>
            <!-- 一级菜单首页配置 -->
            <el-sub-menu index="2">
              <template #title>
                <span>首页配置</span>
              </template>
              <el-menu-item-group>
                <el-menu-item index="/swiper"
                  ><el-icon><Picture /></el-icon>轮播图配置</el-menu-item
                >
                <el-menu-item index="/hot"
                  ><el-icon><StarFilled /></el-icon>热销商品配置</el-menu-item
                >
                <el-menu-item index="/new"
                  ><el-icon><Sell /></el-icon>新品上线配置</el-menu-item
                >
                <el-menu-item index="/recommend"
                  ><el-icon><ShoppingCart /></el-icon>为你推荐配置</el-menu-item
                >
              </el-menu-item-group>
            </el-sub-menu>
            <!-- 一级菜单模块管理 -->
            <el-sub-menu index="3">
              <template #title>
                <span>模块管理</span>
              </template>
              <el-menu-item-group>
                <el-menu-item index="/category"
                  ><el-icon><Menu /></el-icon>分类管理</el-menu-item
                >
                <el-menu-item index="/good"
                  ><el-icon><Goods /></el-icon>商品管理</el-menu-item
                >
                <el-menu-item index="/order"
                  ><el-icon><List /></el-icon>订单管理</el-menu-item
                >
                <el-menu-item index="/guest"
                  ><el-icon><User /></el-icon>会员管理</el-menu-item
                >
              </el-menu-item-group>
            </el-sub-menu>
            <el-sub-menu index="4">
              <template #title>
                <span>系统管理</span>
              </template>
              <el-menu-item-group>
                <el-menu-item index="/account"
                  ><el-icon><Lock /></el-icon>修改密码</el-menu-item
                >
              </el-menu-item-group>
            </el-sub-menu>
          </el-menu>
        </el-aside>
        <el-container class="content">
          <Header />
          <div class="main">
            <router-view />
          </div>
          <Footer />
        </el-container>
      </el-container>
    </el-container>
    <!-- 单独显示登录界面 -->
    <el-container v-else class="container">
      <router-view />
    </el-container>
  </div>
</template>

<!-- <script>
import Header from "@/components/Header.vue";
import Footer from "@/components/Footer.vue";
export default {
  name: "App",
  components: {
    Header,
    Footer,
  },
};
</script> -->

<script setup>
import { reactive, toRefs } from "vue";
import { useRouter } from "vue-router";
import Header from "@/components/Header.vue";
import Footer from "@/components/Footer.vue";
import { localGet } from "@/utils";
import { pathMap } from "./utils";

// 不需要菜单的路径数组
const noMenu = ["/login"];
const router = useRouter();
const state = reactive({
  showMenu: true, // 是否需要显示菜单
  defaultOpen: ["1", "2", "3", "4"], //默认展开菜单
  currentPath: "/", //设置当前选中栏高亮
});
// 监听路由的变化，前置守卫监听to路径是否包含login，包含则转登录
router.beforeEach((to, from, next) => {
  if (to.path == "/login") {
    // 如果路径是 /login 则正常执行
    next();
  } else {
    // 如果不是 /login，判断是否有 token
    if (!localGet("token")) {
      // 如果没有，则跳至登录页面
      next({ path: "/login" });
    } else {
      // 否则继续执行
      next();
    }
  }
  state.showMenu = !noMenu.includes(to.path);
  document.title = pathMap[to.name];
  state.currentPath = to.path;
});
</script>

<style scoped>
.layout {
  min-height: 100vh;
  background-color: #ffffff;
}
.container {
  height: 100vh;
}
.aside {
  width: 200px !important;
  background-color: #222832;
}
.head {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 50px;
}
.head > div {
  display: flex;
  align-items: center;
}

.head img {
  width: 50px;
  height: 50px;
  margin-right: 10px;
}
.head span {
  font-size: 20px;
  color: #ffffff;
}
.line {
  border-top: 1px solid hsla(0, 0%, 100%, 0.05);
  border-bottom: 1px solid rgba(0, 0, 0, 0.2);
}

.content {
  display: flex;
  flex-direction: column;
  max-height: 100vh;
  overflow: hidden;
}
.main {
  height: calc(100vh - 100px);
  overflow: auto;
  padding: 10px;
}

/* 避免左侧mune少边框 */
.el-menu {
  border: 0 !important;
}
</style>

<style>
body {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
</style>
