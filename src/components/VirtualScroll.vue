<template>
  <div class="virtual-scroll-section">
    <h2>长列表（虚拟滚动）</h2>
    <div
      class="virtual-scroll-container"
      ref="scrollContainer"
      @scroll="handleScroll"
    >
      <div
        class="virtual-scroll-placeholder"
        :style="{ height: totalHeight + 'px' }"
      ></div>
      <div
        class="virtual-scroll-content"
        :style="{ transform: `translateY(${offsetY}px)` }"
      >
        <div v-for="item in visibleItems" :key="item.id" class="virtual-item">
          <div class="item-index">{{ item.index + 1 }}.</div>
          <div class="item-content">{{ item.content }}</div>
          <div class="item-timestamp">{{ item.timestamp }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {
  ref,
  defineProps,
  defineExpose,
  computed,
  onMounted,
  onBeforeUnmount,
  onUnmounted,
} from "vue";

const props = defineProps({
  itemHeight: {
    type: Number,
    default: 60,
  },
  containerHeight: {
    type: Number,
    default: 500,
  },
  totalCount: {
    type: Number,
    default: 10000,
  },
  bufferCount: {
    type: Number,
    default: 5,
  },
});

// 响应式数据
const items = ref([]);
const scrollTop = ref(0);
const scrollContainer = ref(null);
let scrollAnimationFrame = null;

// 生成列表数据
const generateItems = () => {
  const newItems = [];
  for (let i = 0; i < props.totalCount; i++) {
    newItems.push({
      id: `item-${i}`,
      index: i,
      content: `虚拟滚动列表项 #${
        i + 1
      } - 详细内容blabla...`,
      timestamp: new Date(Date.now() - i * 60000).toLocaleTimeString(),
    });
  }
  items.value = newItems;
};

// 计算属性
const totalHeight = computed(() => items.value.length * props.itemHeight);
const visibleCount = computed(
  () => Math.ceil(props.containerHeight / props.itemHeight) + props.bufferCount
);
const startIndex = computed(() =>
  Math.max(
    0,
    Math.floor(scrollTop.value / props.itemHeight) -
      Math.floor(props.bufferCount / 2)
  )
);
const endIndex = computed(() =>
  Math.min(
    items.value.length,
    startIndex.value + visibleCount.value + props.bufferCount
  )
);
const visibleItems = computed(() =>
  items.value.slice(startIndex.value, endIndex.value)
);
const offsetY = computed(() => startIndex.value * props.itemHeight);

// 滚动事件处理
const handleScroll = () => {
  if (scrollAnimationFrame) {
    cancelAnimationFrame(scrollAnimationFrame);
  }

  scrollAnimationFrame = requestAnimationFrame(() => {
    if (scrollContainer.value) {
      scrollTop.value = scrollContainer.value.scrollTop;
    }
  });
};

// 滚动到指定位置
const scrollToPosition = (position) => {
  if (scrollContainer.value) {
    scrollContainer.value.scrollTop = position;
  }
};

// 滚动到指定索引
const scrollToIndex = (index) => {
  if (index >= 0 && index < items.value.length) {
    const position = index * props.itemHeight;
    scrollToPosition(position);
  }
};

// 生命周期
onMounted(() => {
  generateItems();

  setTimeout(() => {
    if (scrollContainer.value) {
      scrollContainer.value.style.scrollBehavior = "smooth";
    }
  }, 100);
});
onBeforeUnmount(() => {
  if (scrollAnimationFrame) {
    cancelAnimationFrame(scrollAnimationFrame);
  }
});
onUnmounted(() => {
  if (scrollAnimationFrame) {
    cancelAnimationFrame(scrollAnimationFrame);
  }
});

// 暴露给父组件
defineExpose({
  scrollToPosition,
  scrollToIndex,
});
</script>

<style lang="less" scoped>
.virtual-scroll-section {
  width: 100%;
  margin-bottom: 20px;

  .virtual-scroll-container {
    position: relative;
    height: 500px;
    width: 100%;
    overflow-y: auto;
    border: 1px solid #e0e0e0;
    border-radius: 8px;
    background-color: #ffffff;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.08);
    scroll-behavior: smooth;
    -webkit-overflow-scrolling: touch;

    /* 滚动条样式修改，避免相似冲突 */
    &::-webkit-scrollbar {
      width: 8px;
    }

    &::-webkit-scrollbar-track {
      background: #f1f1f1;
      border-radius: 4px;
    }

    &::-webkit-scrollbar-thumb {
      background: #c1c1c1;
      border-radius: 4px;
    }

    &::-webkit-scrollbar-thumb:hover {
      background: #a8a8a8;
    }
  }

  .virtual-scroll-placeholder {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    pointer-events: none;
  }

  .virtual-scroll-content {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    will-change: transform;
  }

  .virtual-item {
    height: 60px;
    display: flex;
    align-items: center;
    padding: 0 20px;
    box-sizing: border-box;
    border-bottom: 1px solid #e8e8e8;
    font-size: 14px;
    color: #333;
    background-color: #ffffff;
    transition: background-color 0.2s ease;

    &:hover {
      background-color: #f8f9fa;
    }

    &:nth-child(even) {
      background-color: #fafafa;

      &:hover {
        background-color: #f0f2f5;
      }
    }

    .item-index {
      width: 40px;
      font-weight: 600;
      color: #3498db;
      font-size: 15px;
    }

    .item-content {
      flex: 1;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      padding-right: 20px;
    }

    .item-timestamp {
      color: #7f8c8d;
      font-size: 12px;
      white-space: nowrap;
      background-color: #f1f2f3;
      padding: 4px 8px;
      border-radius: 4px;
      min-width: 80px;
      text-align: center;
    }
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .virtual-scroll-section {
    .virtual-scroll-container {
      height: 400px;
    }

    .virtual-item {
      height: 55px;
      padding: 0 15px;
      flex-wrap: wrap;

      .item-content {
        width: 100%;
        order: 1;
        padding-right: 0;
        margin-top: 5px;
        white-space: normal;
        font-size: 13px;
      }

      .item-index {
        order: 0;
      }

      .item-timestamp {
        order: 2;
        margin-left: auto;
        font-size: 11px;
        min-width: 70px;
      }
    }
  }
}
</style>
