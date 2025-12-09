<template>
  <div class="weather-section">
    <div class="weather-container" v-if="weatherData">
      <!-- 天气概览 -->
      <div class="weather-overview">
        <div class="weather-icon">
          <img :src="weatherIconUrl" :alt="weatherData.condition" />
        </div>
        <div class="weather-basic">
          <h3 class="city">{{ weatherData.city }}</h3>
          <div class="temperature">
            <span class="current-temp">{{ weatherData.currentTemp }}℃</span>
            <span class="temp-range"
              >{{ weatherData.minTemp }}℃ ~ {{ weatherData.maxTemp }}℃</span
            >
          </div>
          <div class="condition">{{ weatherData.condition }}</div>
        </div>
      </div>

      <!-- 天气详情 -->
      <div class="weather-details">
        <div class="detail-item">
          <div class="detail-label">湿度</div>
          <div class="detail-value">{{ weatherData.humidity }}%</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">风力</div>
          <div class="detail-value">{{ weatherData.windPower }}</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">风向</div>
          <div class="detail-value">{{ weatherData.windDirection }}</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">空气质量</div>
          <div class="detail-value" :class="aqiClass">
            {{ weatherData.aqi }} ({{ weatherData.aqiLevel }})
          </div>
        </div>
        <div class="detail-item">
          <div class="detail-label">日出</div>
          <div class="detail-value">{{ weatherData.sunrise }}</div>
        </div>
        <div class="detail-item">
          <div class="detail-label">日落</div>
          <div class="detail-value">{{ weatherData.sunset }}</div>
        </div>
      </div>

      <!-- 天气预警 -->
      <div class="weather-alert" v-if="weatherData.alert">
        <div class="alert-icon">⚠️</div>
        <div class="alert-content">{{ weatherData.alert }}</div>
      </div>
    </div>

    <!-- 加载状态 -->
    <div class="weather-loading" v-else-if="loading">加载天气信息中...</div>

    <!-- 错误状态 -->
    <div class="weather-error" v-else-if="error">
      {{ error }}
      <button @click="fetchWeather" class="retry-btn">重试</button>
    </div>

    <!-- 刷新按钮 -->
    <div class="weather-footer">
      <div class="update-time" v-if="weatherData">
        更新时间：{{ weatherData.updateTime }}
      </div>
      <button @click="fetchWeather" class="refresh-btn" :disabled="loading">
        <span v-if="loading">刷新中...</span>
        <span v-else>刷新天气</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, defineExpose } from "vue";

// 响应式数据
const weatherData = ref(null);
const loading = ref(false);
const error = ref(null);
const API_KEY = "ABCD1234EFGH";
const API_HOST = "https://n26apx22rr.re.qweatherapi.com";

// 计算属性
const weatherIconUrl = computed(() => {
  if (!weatherData.value) return "";
  return `${API_HOST}/weather-icons/${weatherData.value.iconCode}.png`;
});

const aqiClass = computed(() => {
  if (!weatherData.value) return "";
  const aqi = weatherData.value.aqi;
  if (aqi <= 50) return "aqi-excellent";
  if (aqi <= 100) return "aqi-good";
  if (aqi <= 150) return "aqi-moderate";
  if (aqi <= 200) return "aqi-poor";
  return "aqi-bad";
});

// 获取天气数据
const fetchWeather = async () => {
  loading.value = true;
  error.value = null;

  try {
    // 获取用户位置
    const location = await getUserLocation();

    // 构建API请求URL
    const url = `${API_HOST}/v7/weather/now?location=${location.longitude},${location.latitude}&key=${API_KEY}`;

    // 发送API请求
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`天气请求失败: ${response.status}`);
    }

    const data = await response.json();

    // 解析API响应
    weatherData.value = parseWeatherData(data);
    weatherData.value.updateTime = new Date().toLocaleTimeString("zh-CN", {
      hour: "2-digit",
      minute: "2-digit",
    });
  } catch (err) {
    error.value = "获取天气信息失败，请稍后重试";
    console.error("获取天气失败:", err);
  } finally {
    loading.value = false;
  }
};

// 解析天气数据
const parseWeatherData = (apiData) => {
  return {
    city: apiData.location.name || "南京市",
    condition: apiData.now.text || "晴",
    currentTemp: apiData.now.temp || 12,
    minTemp: apiData.daily[0].tempMin || 6,
    maxTemp: apiData.daily[0].tempMax || 19,
    humidity: apiData.now.humidity || 65,
    windPower: apiData.now.windScale || "3级",
    windDirection: apiData.now.windDir || "北风",
    aqi: apiData.air.now.aqi || 55,
    aqiLevel: apiData.air.now.category || "良",
    sunrise: apiData.daily[0].sunrise || "06:52",
    sunset: apiData.daily[0].sunset || "17:00",
    iconCode: apiData.now.icon || "100",
    alert: apiData.warning ? apiData.warning.title : null,
  };
};

// 获取用户位置
const getUserLocation = () => {
  return new Promise((resolve, reject) => {
    if (!navigator.geolocation) {
      reject(new Error("浏览器不支持地理位置功能"));
      return;
    }

    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        resolve({ latitude, longitude });
      },
      (err) => {
        // 获取位置失败时默认使用南京
        console.log("使用默认位置:", err);
        resolve({ latitude: 32.0603, longitude: 118.7969 });
      },
      {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0,
      }
    );
  });
};

// 初始化
onMounted(async () => {
  await fetchWeather();
});

// 暴露方法
defineExpose({
  refresh: fetchWeather,
});
</script>

<style lang="less" scoped>
.weather-section {
  width: 100%;
  margin-bottom: 20px;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);

  .weather-container {
    .weather-overview {
      display: flex;
      align-items: center;
      margin-bottom: 20px;
      padding-bottom: 20px;
      border-bottom: 1px solid rgba(52, 152, 219, 0.3);

      .weather-icon {
        width: 80px;
        height: 80px;
        margin-right: 20px;

        img {
          width: 100%;
          height: 100%;
          object-fit: contain;
        }
      }

      .weather-basic {
        flex: 1;

        .city {
          font-size: 1.8rem;
          font-weight: 600;
          color: #2c3e50;
          margin-bottom: 8px;
        }
        .temperature {
          display: flex;
          align-items: baseline;
          gap: 15px;
          margin-bottom: 8px;

          .current-temp {
            font-size: 2.5rem;
            font-weight: 700;
            color: #e74c3c;
          }
          .temp-range {
            font-size: 1.1rem;
            color: #7f8c8d;
          }
        }

        .condition {
          font-size: 1.2rem;
          color: #3498db;
          font-weight: 500;
        }
      }
    }

    .weather-details {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 15px;
      margin-bottom: 25px;
      padding-bottom: 20px;
      border-bottom: 1px solid rgba(52, 152, 219, 0.3);

      .detail-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 12px;
        background: rgba(255, 255, 255, 0.8);
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);

        .detail-label {
          font-size: 0.9rem;
          color: #7f8c8d;
          margin-bottom: 5px;
        }

        .detail-value {
          font-size: 1.1rem;
          font-weight: 600;
          color: #2c3e50;

          &.aqi-excellent {
            color: #27ae60;
          }
          &.aqi-good {
            color: #2ecc71;
          }
          &.aqi-moderate {
            color: #f39c12;
          }
          &.aqi-poor {
            color: #e74c3c;
          }
          &.aqi-bad {
            color: #c0392b;
          }
        }
      }
    }
    .weather-alert {
      display: flex;
      align-items: center;
      padding: 12px 16px;
      margin-bottom: 20px;
      background: linear-gradient(135deg, #ffeaa7 0%, #fdcb6e 100%);
      border-radius: 8px;
      border-left: 4px solid #e74c3c;

      .alert-icon {
        font-size: 1.5rem;
        margin-right: 12px;
      }
      .alert-content {
        flex: 1;
        color: #d35400;
        font-weight: 500;
      }
    }
  }

  .weather-loading,
  .weather-error {
    text-align: center;
    padding: 40px 20px;
    color: #7f8c8d;

    .retry-btn {
      margin-top: 10px;
      padding: 8px 20px;
      background: #3498db;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.3s;

      &:hover {
        background: #2980b9;
      }
    }
  }

  .weather-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 15px;
    border-top: 1px solid rgba(52, 152, 219, 0.3);

    .update-time {
      font-size: 0.9rem;
      color: #7f8c8d;
    }
    .refresh-btn {
      padding: 8px 20px;
      background: #3498db;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      transition: all 0.3s;
      font-weight: 500;

      &:hover:not(:disabled) {
        background: #2980b9;
        transform: translateY(-1px);
      }
      &:disabled {
        background: #bdc3c7;
        cursor: not-allowed;
        transform: none;
      }
    }
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .weather-section {
    padding: 15px;

    .weather-container {
      .weather-details {
        grid-template-columns: repeat(2, 1fr);
      }
      .living-indices {
        .indices-grid {
          grid-template-columns: 1fr;
        }
      }
    }

    .weather-footer {
      flex-direction: column;
      gap: 10px;

      .refresh-btn {
        width: 100%;
      }
    }
  }
}
</style>
