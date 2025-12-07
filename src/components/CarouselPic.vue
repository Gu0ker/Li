<template>
  <div class="CarouselPic-container">
    <!-- 轮播图容器 (必须固定图片大小) -->
    <div class="CarouselPic-wrapper" ref="carouselWrapper">
      <div
        class="CarouselPic-slides"
        :style="slidesStyle"
        @touchstart="handleTouchStart"
        @touchmove="handleTouchMove"
        @touchend="handleTouchEnd"
      >
        <!-- 最后一张图片的复制，用于无限循环 -->
        <div
          v-if="props.images.length > 1"
          class="CarouselPic-slide"
          :key="'last-clone'"
        >
          <img
            :src="props.images[props.images.length - 1]"
            :alt="`图片 ${props.images.length}`"
            class="CarouselPic-image"
          />
        </div>

        <!-- 原始图片 -->
        <div
          v-for="(image, index) in props.images"
          :key="index"
          class="CarouselPic-slide"
        >
          <img
            :src="image"
            :alt="`图片 ${index + 1}`"
            class="CarouselPic-image"
          />
        </div>

        <!-- 第一张图片的复制，用于无限循环 -->
        <div
          v-if="props.images.length > 1"
          class="CarouselPic-slide"
          :key="'first-clone'"
        >
          <img :src="props.images[0]" alt="图片 1" class="CarouselPic-image" />
        </div>
      </div>

      <!-- 左右切换按钮 -->
      <button
        v-if="props.images.length > 1"
        class="CarouselPic-btn prev-btn"
        @click="prevSlide"
        aria-label="上一张"
      >
        <span class="btn-icon">‹</span>
      </button>
      <button
        v-if="props.images.length > 1"
        class="CarouselPic-btn next-btn"
        @click="nextSlide"
        aria-label="下一张"
      >
        <span class="btn-icon">›</span>
      </button>
    </div>

    <!-- 指示点 -->
    <div v-if="props.images.length > 1" class="CarouselPic-dots">
      <button
        v-for="(_, index) in props.images"
        :key="index"
        class="dot"
        :class="{ active: currentSlide === index }"
        @click="goToSlide(index)"
        :aria-label="`切换到第 ${index + 1} 张图片`"
      ></button>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, computed, onMounted, onBeforeUnmount } from "vue";

const props = defineProps({
  images: {
    type: Array,
    default: () => [],
  },
  interval: {
    type: Number,
    default: 3000,
  },
  transitionSpeed: {
    type: Number,
    default: 300,
  },
});

const currentSlide = ref(0);
const isTransitioning = ref(false);
const autoPlayTimer = ref(null);
const isAutoPlaying = ref(true);
const carouselWrapper = ref(null);
const startX = ref(0);
const currentX = ref(0);
const isDragging = ref(false);
const dragThreshold = 50;

// 计算当前展示的真实索引
computed(() => {
  if (props.images.length <= 1) return 0;
  if (currentSlide.value < 0) return props.images.length - 1;
  if (currentSlide.value >= props.images.length) return 0;
  return currentSlide.value;
});

// 计算轮播图样式
const slidesStyle = computed(() => {
  const offset = props.images.length > 1 ? 1 : 0;
  const translateX = -(currentSlide.value + offset) * 100;
  const transition = isTransitioning.value
    ? `transform ${props.transitionSpeed}ms ease`
    : "none";
  return {
    transform: `translateX(${translateX}%)`,
    transition: transition,
  };
});

// 切换指定幻灯片
const goToSlide = (index) => {
  if (isTransitioning.value || currentSlide.value === index) return;

  isTransitioning.value = true;
  currentSlide.value = index;
  resetAutoPlay();

  setTimeout(() => {
    isTransitioning.value = false;
  }, props.transitionSpeed);
};

// 下一张幻灯片
const nextSlide = () => {
  if (isTransitioning.value) return;
  isTransitioning.value = true;
  currentSlide.value++;

  if (currentSlide.value === props.images.length) {
    setTimeout(() => {
      isTransitioning.value = false;
      currentSlide.value = 0;
      // 防抖结束跳回
      setTimeout(() => {
        isTransitioning.value = false;
      }, 50);
    }, props.transitionSpeed);
  } else {
    setTimeout(() => {
      isTransitioning.value = false;
    }, props.transitionSpeed);
  }

  resetAutoPlay();
};

// 上一张幻灯片
const prevSlide = () => {
  if (isTransitioning.value) return;
  isTransitioning.value = true;
  currentSlide.value--;

  if (currentSlide.value === -1) {
    // 同上，跳回最后
    setTimeout(() => {
      isTransitioning.value = false;
      currentSlide.value = props.images.length - 1;
      setTimeout(() => {
        isTransitioning.value = false;
      }, 50);
    }, props.transitionSpeed);
  } else {
    setTimeout(() => {
      isTransitioning.value = false;
    }, props.transitionSpeed);
  }

  resetAutoPlay();
};

// 重置自动轮播计时器
const resetAutoPlay = () => {
  if (autoPlayTimer.value) {
    clearInterval(autoPlayTimer.value);
  }

  if (isAutoPlaying.value && props.images.length > 1) {
    autoPlayTimer.value = setInterval(() => {
      nextSlide();
    }, props.interval);
  }
};

// 切换自动轮播状态
() => {
  isAutoPlaying.value = !isAutoPlaying.value;
  resetAutoPlay();
};

// 触摸开始事件
const handleTouchStart = (event) => {
  isDragging.value = true;
  startX.value = event.touches[0].clientX;
  currentX.value = startX.value;
  isTransitioning.value = false;

  if (autoPlayTimer.value) {
    clearInterval(autoPlayTimer.value);
  }
};
// 触摸移动事件
const handleTouchMove = (event) => {
  if (!isDragging.value) return;

  currentX.value = event.touches[0].clientX;
  const diff = startX.value - currentX.value;

  // 阻止默认行为，避免页面滚动
  if (Math.abs(diff) > 10) {
    event.preventDefault();
  }
};

// 触摸结束事件
const handleTouchEnd = () => {
  if (!isDragging.value) return;
  isDragging.value = false;
  const diff = startX.value - currentX.value;

  if (Math.abs(diff) > dragThreshold) {
    if (diff > 0) {
      nextSlide();
    } else {
      prevSlide();
    }
  }
  resetAutoPlay();
};

// 生命周期钩子
onMounted(() => {
  if (props.images.length > 1) {
    resetAutoPlay();
  }

  // 监听鼠标事件
  if (carouselWrapper.value) {
    carouselWrapper.value.addEventListener("mouseenter", () => {
      if (autoPlayTimer.value) {
        clearInterval(autoPlayTimer.value);
      }
    });
    carouselWrapper.value.addEventListener("mouseleave", () => {
      if (isAutoPlaying.value) {
        resetAutoPlay();
      }
    });
  }
});
onBeforeUnmount(() => {
  if (autoPlayTimer.value) {
    clearInterval(autoPlayTimer.value);
  }
});
</script>

<style lang="less" scoped>
.CarouselPic-container {
  position: relative;
  width: 100%;
  max-width: 800px;
  margin: 0 auto 20px;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);

  .CarouselPic-wrapper {
    position: relative;
    width: 100%;
    height: 400px;
    overflow: hidden;

    .CarouselPic-slides {
      display: flex;
      height: 100%;
      width: 100%;

      .CarouselPic-slide {
        flex: 0 0 100%;
        height: 100%;

        .CarouselPic-image {
          width: 100%;
          height: 100%;
          object-fit: cover;
          display: block;
        }
      }
    }

    .CarouselPic-btn {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 50px;
      height: 50px;
      background-color: rgba(0, 0, 0, 0.5);
      border: none;
      border-radius: 50%;
      color: white;
      font-size: 24px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.3s ease;
      opacity: 0.7;

      &:hover {
        background-color: rgba(0, 0, 0, 0.8);
        opacity: 1;
      }

      &.prev-btn {
        left: 20px;
      }

      &.next-btn {
        right: 20px;
      }

      .btn-icon {
        display: block;
        line-height: 1;
      }
    }
  }

  .CarouselPic-dots {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 14px 0;
    gap: 10px;

    .dot {
      width: 12px;
      height: 16px;
      border-radius: 50%;
      // background-color: rgba(0, 0, 0, 0.3);
      border: 2px solid transparent;
      cursor: pointer;
      transition: all 0.3s ease;

      &:hover {
        background-color: rgba(0, 0, 0, 0.5);
      }

      &.active {
        background-color: #3498db;
      }
    }
  }
}

// 响应式设计
@media (max-width: 768px) {
  .CarouselPic-container {
    .CarouselPic-wrapper {
      height: 300px;

      .CarouselPic-btn {
        width: 40px;
        height: 40px;
        font-size: 20px;

        &.prev-btn {
          left: 10px;
        }

        &.next-btn {
          right: 10px;
        }
      }
    }
  }
}

@media (max-width: 480px) {
  .CarouselPic-container {
    .CarouselPic-wrapper {
      height: 200px;

      .CarouselPic-btn {
        width: 30px;
        height: 30px;
        font-size: 16px;
      }
    }

    .CarouselPic-dots {
      .dot {
        width: 10px;
        height: 10px;
      }
    }
  }
}
</style>
