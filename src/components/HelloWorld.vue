<template>
  <div class="app-container">
    <!-- 移动端汉堡菜单按钮（小于768显示） -->
    <button
      class="hamburger-btn"
      :class="{ active: isMenuOpen }"
      @click="toggleMenu"
      aria-label="切换菜单"
    >
      <span class="hamburger-line"></span>
      <span class="hamburger-line"></span>
      <span class="hamburger-line"></span>
    </button>

    <!-- 左侧导航栏 -->
    <nav class="sidebar" :class="{ open: isMenuOpen, mobile: isMobile }">
      <ul class="nav-menu">
        <li
          v-for="item in menus"
          :key="item.id"
          class="nav-item"
          :class="{ active: activeItem === item.id }"
          @click="selectItem(item.id)"
        >
          <span class="nav-text">{{ item.name }}</span>
        </li>
      </ul>
    </nav>

    <!-- 右侧内容区域 -->
    <div class="content-area">
      <p>滚动测试标题</p>
      <div class="test-scroll">
        <div class="test-box" v-for="n in 30" :key="n">
          <h4>测试 {{ n }}</h4>
        </div>
      </div>
    </div>

    <!-- 移动端菜单遮罩层 -->
    <div
      v-if="isMobile && isMenuOpen"
      class="menu-overlay"
      @click="toggleMenu"
    ></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

// 响应式数据
const isMenuOpen = ref(false);
const isMobile = ref(false);
const screenWidth = ref(window.innerWidth);
const activeItem = ref(1);

// 菜单项数据
const menus = [
  {
    id: "system-user-role",
    name: "Role Settings",
    path: "/system/user/role",
    parentId: "system-user",
  },
  { id: "dashboard", name: "Dashboard", path: "/dashboard", parentId: null },
  {
    id: "reports-sales-monthly",
    name: "Monthly Sales",
    path: "/reports/sales/monthly",
    parentId: "reports-sales",
  },
  {
    id: "dashboard-analysis",
    name: "Analysis",
    path: "/dashboard/analysis",
    parentId: "dashboard",
  },
  { id: "system", name: "System Management", path: "/system", parentId: null },
  {
    id: "dashboard-analysis-realtime",
    name: "Realtime Data",
    path: "/dashboard/analysis/realtime",
    parentId: "dashboard-analysis",
  },
  { id: "reports", name: "Reports", path: "/reports", parentId: null },
  {
    id: "system-permission",
    name: "Permission",
    path: "/system/permission",
    parentId: "system",
  },
  {
    id: "dashboard-analysis-trend",
    name: "Trend",
    path: "/dashboard/analysis/trend",
    parentId: "dashboard-analysis",
  },
  {
    id: "reports-sales",
    name: "Sales Reports",
    path: "/reports/sales",
    parentId: "reports",
  },
  {
    id: "dashboard-monitor-system",
    name: "System Monitor",
    path: "/dashboard/monitor/system",
    parentId: "dashboard-monitor",
  },
  {
    id: "system-permission-menu",
    name: "Menu Permission",
    path: "/system/permission/menu",
    parentId: "system-permission",
  },
  {
    id: "dashboard-monitor",
    name: "Monitor",
    path: "/dashboard/monitor",
    parentId: "dashboard",
  },
  {
    id: "system-user",
    name: "User Management",
    path: "/system/user",
    parentId: "system",
  },
  {
    id: "system-user-list",
    name: "User List",
    path: "/system/user/list",
    parentId: "system-user",
  },
  {
    id: "reports-sales-daily",
    name: "Daily Sales",
    path: "/reports/sales/daily",
    parentId: "reports-sales",
  },
];

// 切换菜单状态
const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

// 选择菜单项
const selectItem = (id) => {
  activeItem.value = id;
  if (isMobile.value) {
    isMenuOpen.value = false;
  }
};

// 检测屏幕宽度
const checkScreenWidth = () => {
  screenWidth.value = window.innerWidth;
  isMobile.value = screenWidth.value < 768;

  // 在桌面端，菜单始终展开
  if (!isMobile.value) {
    isMenuOpen.value = true;
  } else {
    isMenuOpen.value = false;
  }
};

// 生命周期钩子
onMounted(() => {
  checkScreenWidth();
  window.addEventListener("resize", checkScreenWidth);
});
onBeforeUnmount(() => {
  window.removeEventListener("resize", checkScreenWidth);
});
</script>

<style lang="less">
.app-container {
  box-sizing: border-box;
  display: flex;
  height: calc(100vh - 16px);
  background-color: white;
  color: black;
  overflow: hidden;

  /* 移动端汉堡菜单按钮 */
  .hamburger-btn {
    display: none;
    position: fixed;
    top: 10px;
    left: 10px;
    width: 30px;
    height: 30px;
    background-color: #1a252f;
    border: none;
    border-radius: 4px;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    z-index: 1001;
    transition: all 0.3s ease;

    .hamburger-line {
      display: block;
      width: 18px;
      height: 2px;
      background-color: white;
      margin: 2px 0;
      border-radius: 2px;
    }
  }
  .hamburger-btn:hover {
    background-color: #34495e;
  }
  .hamburger-btn.active .hamburger-line:nth-child(1) {
    transform: translateY(6px) rotate(45deg);
  }
  .hamburger-btn.active .hamburger-line:nth-child(2) {
    opacity: 0;
  }
  .hamburger-btn.active .hamburger-line:nth-child(3) {
    transform: translateY(-6px) rotate(-45deg);
  }

  /* 左侧导航栏 */
  .sidebar {
    width: 200px;
    background-color: #2c3e50;
    color: white;
    display: flex;
    flex-direction: column;
    transition: transform 0.3s ease;
    z-index: 1000;
    flex-shrink: 0;

    .nav-menu {
      list-style: none;
      flex: 1;
      padding: 30px 0;
      overflow-y: auto;

      .nav-item {
        padding: 12px 20px;
        display: flex;
        align-items: center;
        cursor: pointer;
        transition: all 0.2s ease;
        border-left: 4px solid transparent;

        .nav-text {
          font-size: 16px;
        }
      }
      .nav-item:hover {
        background-color: #34495e;
      }
      .nav-item.active {
        background-color: #3498db;
        border-left-color: greenyellow;
      }
    }
  }

  .sidebar.mobile {
    position: fixed;
    top: 0;
    left: 0;
    height: 100%;
    transform: translateX(-100%);
    z-index: 1000;
  }
  .sidebar.mobile.open {
    transform: translateX(0);
    box-shadow: 5px 0 15px rgba(0, 0, 0, 0.2);
  }

  /* 右侧内容区域 */
  .content-area {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;

    .test-scroll {
      display: flex;
      flex-direction: column;
      // justify-content: center;
      // align-items: flex-start;
      gap: 20px;
      overflow-y: auto;
      overflow-x: hidden;

      .test-box {
        width: 100%;
        height: 200px;
        padding: 10px 20px;
        background-color: #fff;
      }
    }
  }

  /* 移动端遮罩层 */
  .menu-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 999;
  }

  /* 响应式设计 */
  @media (max-width: 768px) {
    .hamburger-btn {
      display: flex;
    }
    .sidebar:not(.mobile) {
      display: none;
    }
  }
}
</style>
