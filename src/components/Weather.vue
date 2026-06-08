```vue
<template>
  <div
    class="weather"
    v-if="weatherData.adCode.city && weatherData.weather.weather"
  >
    <span>{{ weatherData.adCode.city }}&nbsp;</span>

    <span>{{ weatherData.weather.weather }}&nbsp;</span>

    <span>{{ weatherData.weather.temperature }}℃</span>

    <span class="sm-hidden">
      &nbsp;
      {{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}
      &nbsp;
    </span>

    <span class="sm-hidden">
      {{ weatherData.weather.windpower }}&nbsp;级
    </span>
  </div>

  <div class="weather" v-else>
    <span>天气坏了,不考虑修😅</span>
  </div>
</template>

<script setup>
import { reactive, onMounted, h } from "vue";

import { getAdcode, getWeather, getOtherWeather } from "@/api";

// ⚠️ 这里重命名，避免覆盖 JS 内置 Error
import { Error as ErrorIcon } from "@icon-park/vue-next";

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  adCode: {
    city: null,
    adcode: null,
  },

  weather: {
    weather: null,
    temperature: null,
    winddirection: null,
    windpower: null,
  },
});

// 计算平均温度
const getTemperature = (min, max) =>
{
  try
  {
    const average = (Number(min) + Number(max)) / 2;

    return Math.round(average);
  }
  catch (error)
  {
    console.error("计算温度出现错误：", error);

    return "NaN";
  }
};

// 获取天气数据
const getWeatherData = async () =>
{
  try
  {
    // =========================
    // 未配置高德 Key
    // =========================
    if (!mainKey)
    {
      console.log("未配置天气 Key，使用备用天气接口");

      const result = await getOtherWeather();

      console.log("备用天气接口返回：", result);

      const data = result?.result;

      if (!data)
      {
        throw new Error("备用天气数据为空");
      }

      weatherData.adCode =
      {
        city: data.city?.City || "未知地区",
      };

      weatherData.weather =
      {
        weather: data.condition?.day_weather || "未知",

        temperature: getTemperature(
          data.condition?.min_degree,
          data.condition?.max_degree
        ),

        winddirection:
          data.condition?.day_wind_direction || "未知",

        windpower:
          data.condition?.day_wind_power || "未知",
      };
    }

    // =========================
    // 使用高德天气 API
    // =========================
    else
    {
      console.log("当前天气 Key：", mainKey);

      // 获取地区信息
      const adCode = await getAdcode(mainKey);

      console.log("AdCode 数据：", adCode);

      if (!adCode || adCode.infocode !== "10000")
      {
        throw new Error("地区查询失败");
      }

      weatherData.adCode =
      {
        city: adCode.city,
        adcode: adCode.adcode,
      };

      // 获取天气信息
      const result = await getWeather(
        mainKey,
        weatherData.adCode.adcode
      );

      console.log("天气接口返回：", result);

      // ⚠️ 兼容 axios data 层
      const live =
        result?.data?.lives?.[0] ||
        result?.lives?.[0];

      if (!live)
      {
        throw new Error("天气数据为空");
      }

      weatherData.weather =
      {
        weather: live.weather,

        temperature: live.temperature,

        winddirection: live.winddirection,

        windpower: live.windpower,
      };

      console.log("最终天气数据：", weatherData);
    }
  }
  catch (error)
  {
    console.error("天气信息获取失败：", error);

    onError("天气信息获取失败");
  }
};

// 错误提示
const onError = (message) =>
{
  ElMessage({
    message,

    icon: h(ErrorIcon, {
      theme: "filled",
      fill: "#efefef",
    }),
  });

  console.error(message);
};

// 页面加载时执行
onMounted(() =>
{
  getWeatherData();
});
</script>

<style scoped lang="scss">
.weather
{
  animation: fadeInUp 0.8s ease;

  transition: all 0.3s ease;

  &:hover
  {
    transform: translateY(-3px);
  }
}

// 淡入上浮动画
@keyframes fadeInUp
{
  from
  {
    opacity: 0;

    transform: translateY(20px);
  }

  to
  {
    opacity: 1;

    transform: translateY(0);
  }
}
</style>
```
