<template>
  <!-- 社交链接 -->
  <div class="social">
    <div class="link">
      <div
        class="link-item"
        v-for="item in socialLinks"
        :key="item.name"
        @mouseenter="
          socialTip = item.tip;
          activeQr = item.qrcode || null;
        "
        @mouseleave="
          socialTip = '通过这里联系我吧';
          activeQr = null;
        "
      >
        <!-- 普通链接 -->
        <a
          v-if="item.url"
          :href="item.url"
          target="_blank"
        >
          <img class="icon" :src="item.icon" height="24" />
        </a>

        <!-- 二维码类型 -->
        <div v-else class="qr-trigger">
          <img class="icon" :src="item.icon" height="24" />
        </div>

        <!-- 二维码弹窗 -->
        <transition name="qr-fade">
          <div
            v-if="activeQr === item.qrcode"
            class="qrcode-popup"
          >
            <img
              class="qrcode-img"
              :src="item.qrcode"
              alt="二维码"
            />
            <p>嘻嘻🙂</p>
          </div>
        </transition>
      </div>
    </div>

    <span class="tip">{{ socialTip }}</span>
  </div>
</template>

<script setup>
import { ref } from "vue";
import socialLinks from "@/assets/socialLinks.json";

// 提示文字
const socialTip = ref("通过这里联系我吧");

// 当前显示的二维码
const activeQr = ref(null);
</script>

<style lang="scss" scoped>
.social
{
  margin-top: 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 460px;
  width: 100%;
  height: 42px;
  background-color: transparent;
  border-radius: 6px;
  backdrop-filter: blur(0);
  animation: fade 0.5s;

  transition:
    background-color 0.3s,
    backdrop-filter 0.3s;

  @media (max-width: 840px)
  {
    max-width: 100%;
    justify-content: center;

    .link
    {
      justify-content: space-evenly !important;
      width: 90%;
    }

    .tip
    {
      display: none !important;
    }
  }

  .link
  {
    display: flex;
    align-items: center;
    justify-content: center;

    .link-item
    {
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    a,
    .qr-trigger
    {
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .icon
    {
      margin: 0 12px;
      transition: transform 0.3s;
      cursor: pointer;

      &:hover
      {
        transform: scale(1.1);
      }

      &:active
      {
        transform: scale(1);
      }
    }
  }

  .tip
  {
    display: none;
    margin-right: 12px;
    animation: fade 0.5s;
  }

  @media (min-width: 768px)
  {
    &:hover
    {
      background-color: #00000040;
      backdrop-filter: blur(5px);

      .tip
      {
        display: block;
      }
    }
  }
}

/* 二维码弹窗 */

.qrcode-popup
{
  position: absolute;

  bottom: 60px;
  left: 50%;

  transform: translateX(-50%);

  width: 180px;

  padding: 12px;

  border-radius: 18px;

  background: rgba(20, 20, 20, 0.92);

  backdrop-filter: blur(20px);

  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.35);

  z-index: 999;

  display: flex;
  flex-direction: column;
  align-items: center;
}

.qrcode-img
{
  width: 180px;
  height: auto;

  border-radius: 12px;

  display: block;
}
.qrcode-popup p
{
  margin-top: 10px;

  font-size: 13px;

  color: #fff;
}

/* 动画 */

.qr-fade-enter-active,
.qr-fade-leave-active
{
  transition: all 0.25s ease;
}

.qr-fade-enter-from,
.qr-fade-leave-to
{
  opacity: 0;

  transform:
    translateX(-50%)
    translateY(10px);
}
</style>