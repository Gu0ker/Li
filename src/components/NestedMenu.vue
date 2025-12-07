<template>
  <ul class="nav-menu">
    <li
      v-for="item in props.menus"
      :key="item.id"
      class="nav-item"
      :class="{
        'has-children': hasChildren(item),
        expanded: expandedItems.includes(item.id),
        active: props.activeItem === item.id,
      }"
    >
      <!-- 菜单项内容 -->
      <div
        class="menu-item-content"
        @click.stop="handleItemClick(item.id, hasChildren(item))"
      >
        <span class="menu-text">{{ item.name }}</span>
        <!-- 展开/收起图标 -->
        <span
          v-if="hasChildren(item)"
          class="expand-icon"
          :class="{ expanded: expandedItems.includes(item.id) }"
        >
          ▶
        </span>
      </div>

      <!-- 子菜单 -->
      <NestedMenu
        v-if="hasChildren(item) && expandedItems.includes(item.id)"
        :menus="item.children"
        :active-item="props.activeItem"
        :level="props.level + 1"
        @select="$emit('select', $event)"
      />
    </li>
  </ul>
</template>

<script setup>
import { ref, defineProps, defineEmits } from "vue";

const props = defineProps({
  menus: {
    type: Array,
    required: true,
  },
  activeItem: {
    type: String,
    default: "",
  },
  level: {
    type: Number,
    default: 0,
  },
});

const emit = defineEmits(["select"]);

// 记录哪些菜单项是展开的
const expandedItems = ref([]);

// 检查是否有子菜单
const hasChildren = (item) => {
  return item.children && item.children.length > 0;
};

// 处理菜单项点击
const handleItemClick = (id, hasChildren) => {
  if (hasChildren) {
    // 有子菜单，切换展开状态
    const index = expandedItems.value.indexOf(id);
    if (index > -1) {
      expandedItems.value.splice(index, 1);
    } else {
      expandedItems.value.push(id);
    }
  } else {
    // 无子菜单，选择该项
    emit("select", id);
  }
};
</script>

<style lang="less" scoped>
.nav-menu {
  list-style: none;
  margin: 0;
  padding: 0;
  width: 100%;

  .nav-item {
    padding: 10px 0 10px 12px;
    box-sizing: border-box;
    cursor: pointer;
    transition: all 0.2s ease;
    position: relative;
    border-left: 4px solid transparent;

    &.has-children {
      & > .menu-item-content {
        position: relative;
        padding-right: 30px;
      }
    }

    &.expanded {
      background-color: rgba(255, 255, 255, 0.05);

      & > .menu-item-content {
        .expand-icon {
          transform: rotate(90deg);
        }
      }
    }

    &.active {
      background-color: #3498db;
      // border-left-color: greenyellow;

      .menu-text {
        font-weight: bold;
      }
    }

    .menu-item-content {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .menu-text {
      font-size: 16px;
      flex: 1;
    }

    .expand-icon {
      font-size: 12px;
      transition: transform 0.3s ease;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 20px;
      height: 20px;

      &.expanded {
        transform: rotate(90deg);
      }
    }
  }

  // 子菜单样式
  :deep(.nav-menu) {
    .nav-item {
      // padding: 20px 0 0 12px;
    }
  }
}
</style>
