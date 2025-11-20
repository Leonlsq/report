<template>
  <div 
    class="ios-unlock-container" 
    :style="{ backgroundImage: `url(${backgroundImage})` }"
    @click="handleContainerClick"
  >
    <div class="overlay" :class="{ 'blur-bg': showPasscode }"></div>

    <div class="status-bar">
      <span class="time">{{ currentTime }}</span>
      <div class="icons">
        <svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M19.6 3C19.4 3 19.2 3.05 19 3.15L3.85 12C3.45 12.25 3.35 12.8 3.6 13.2C3.85 13.6 4.4 13.7 4.8 13.45L19 4.65C19.2 4.55 19.4 4.5 19.6 4.5C20.25 4.5 20.75 5 20.75 5.65C20.75 6.3 20.25 6.8 19.6 6.8L9.8 12.85C9.4 13.1 9.3 13.65 9.55 14.05C9.8 14.45 10.35 14.55 10.75 14.3L20.55 8.25C21.55 7.65 22.25 6.5 22.25 5.25C22.25 3.4 20.75 1.9 18.9 1.9C18.35 1.9 17.8 2.05 17.35 2.3L16.1 3.05C16.35 3 16.65 3 16.9 3C17.55 3 18.05 3.5 18.05 4.15C18.05 4.8 17.55 5.3 16.9 5.3C16.65 5.3 16.35 5.2 16.1 5.05L19.6 3Z"/></svg>
        <svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M12.01 21.49L23.64 7C23.19 6.6 18.71 3 12 3C5.27 3 0.81 6.6 0.36 7L12 21.49V21.49H12.01Z"/></svg>
        <svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M15.67 4H14V2h-4v2H8.33C7.6 4 7 4.6 7 5.33v15.33C7 21.4 7.6 22 8.33 22h7.33c.74 0 1.34-.6 1.34-1.33V5.33C17 4.6 16.4 4 15.67 4z"/></svg>
      </div>
    </div>

    <transition name="fade-up">
      <div v-if="!showPasscode" class="lock-screen-content">
        <div class="lock-icon">
          <svg viewBox="0 0 24 24" fill="currentColor" width="30" height="30"><path d="M12 17a2 2 0 1 0 0-4 2 2 0 0 0 0 4zm6-9a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V10a2 2 0 0 1 2-2h1V6a5 5 0 0 1 10 0v2h1zm-6-5a3 3 0 0 0-3 3v2h6V6a3 3 0 0 0-3-3z"/></svg>
        </div>
        <h1 class="big-time">{{ currentTime }}</h1>
        <p class="date">{{ currentDate }}</p>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="!showPasscode" class="swipe-hint">
        <p>向上轻扫以解锁</p>
        <div class="home-bar"></div>
      </div>
    </transition>

    <transition name="slide-up">
      <div v-if="showPasscode" class="passcode-screen">
        <p class="enter-passcode-text">输入密码</p>
        
        <div class="passcode-dots" :class="{ 'shake': isError }">
          <div 
            v-for="i in 6" 
            :key="i" 
            class="dot" 
            :class="{ 'filled': passcode.length >= i }"
          ></div>
        </div>

        <div class="keypad">
          <div class="key-row" v-for="(row, rowIndex) in keys" :key="rowIndex">
            <button 
              v-for="(key, kIndex) in row" 
              :key="kIndex" 
              class="key-btn" 
              :class="{ 'empty-key': !key.value && key.value !== 0 && key.action !== 'delete' }"
              @click.stop="handleKeyClick(key)"
            >
              <span class="key-num" v-if="key.value !== undefined && key.value !== null">{{ key.value }}</span>
              <span class="key-letters" v-if="key.letters">{{ key.letters }}</span>
              <span v-if="key.action === 'delete'" class="delete-icon">
                <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"><path d="M21 4H8l-7 8 7 8h13a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2z"></path><line x1="18" y1="9" x2="12" y2="15"></line><line x1="12" y1="9" x2="18" y2="15"></line></svg>
              </span>
            </button>
          </div>
        </div>
        
        <button class="cancel-btn" @click.stop="showPasscode = false">取消</button>
      </div>
    </transition>

  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// --- 预留背景图片路径 ---
// ⭐请在这里替换为你想要的背景图片地址⭐
const backgroundImage = '/photos/1.jpg' 

const emit = defineEmits(['unlocked'])

const CORRECT_PASSCODE = '061128'
const passcode = ref('')
const showPasscode = ref(false)
const isError = ref(false)

// 时间日期相关
const now = ref(new Date())

// ⭐ 修复点1：类型定义为 any，避免 build 时环境差异导致的类型报错
let timer: any = null

const currentTime = computed(() => {
  return now.value.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', hour12: false })
})
const currentDate = computed(() => {
  const options: Intl.DateTimeFormatOptions = { weekday: 'long', month: 'long', day: 'numeric' }
  return now.value.toLocaleDateString('zh-CN', options)
})

onMounted(() => {
  timer = setInterval(() => { now.value = new Date() }, 1000)
})
onUnmounted(() => {
  if (timer) clearInterval(timer)
})

// 点击屏幕显示密码界面
const handleContainerClick = () => {
  if (!showPasscode.value) {
    showPasscode.value = true
  }
}

// ⭐ 修复点2：定义接口，解决 implicitly has an 'any' type 错误
interface KeyItem {
  value?: number | null;
  letters?: string;
  action?: string;
}

// 键盘配置 (应用接口)
const keys: KeyItem[][] = [
  [{ value: 1, letters: '' }, { value: 2, letters: 'ABC' }, { value: 3, letters: 'DEF' }],
  [{ value: 4, letters: 'GHI' }, { value: 5, letters: 'JKL' }, { value: 6, letters: 'MNO' }],
  [{ value: 7, letters: 'PQRS' }, { value: 8, letters: 'TUV' }, { value: 9, letters: 'WXYZ' }],
  [{ value: null }, { value: 0, letters: '' }, { action: 'delete' }]
]

// ⭐ 修复点3：参数显式指定类型，且进行类型安全的字符串转换
const handleKeyClick = (key: KeyItem) => {
  if (isError.value) return

  if (key.action === 'delete') {
    passcode.value = passcode.value.slice(0, -1)
  } else if (key.value !== undefined && key.value !== null) {
    if (passcode.value.length < 6) {
      // 显式转为 String，防止 TS 报错
      passcode.value += String(key.value)
      
      // 密码输满6位，校验
      if (passcode.value.length === 6) {
        if (passcode.value === CORRECT_PASSCODE) {
          // 解锁成功
          setTimeout(() => {
            emit('unlocked')
          }, 300) 
        } else {
          // 解锁失败
          isError.value = true
          setTimeout(() => {
            passcode.value = ''
            isError.value = false
          }, 500) 
        }
      }
    }
  }
}
</script>

<style scoped>
/* iOS 字体 */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500&display=swap');

.ios-unlock-container {
  position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
  background-size: cover; background-position: center;
  color: white; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  z-index: 20000; /* 确保在最上层 */
  overflow: hidden;
}

.overlay {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0, 0, 0, 0.2);
  transition: backdrop-filter 0.3s ease;
}
.overlay.blur-bg { backdrop-filter: blur(15px); background: rgba(0, 0, 0, 0.4); }

/* 状态栏 */
.status-bar {
  position: absolute; top: 0; left: 0; width: 100%; padding: 10px 20px;
  display: flex; justify-content: space-between; align-items: center;
  font-size: 14px; font-weight: 500; z-index: 2;
}
.icons .icon { width: 18px; height: 18px; margin-left: 5px; }

/* 锁屏主内容 */
.lock-screen-content {
  position: absolute; top: 15%; width: 100%;
  text-align: center; z-index: 1;
}
.lock-icon { margin-bottom: 10px; }
.big-time { font-size: 80px; font-weight: 300; margin: 0; line-height: 1; }
.date { font-size: 20px; font-weight: 400; margin-top: 10px; }

/* 底部提示 */
.swipe-hint {
  position: absolute; bottom: 30px; width: 100%;
  text-align: center; font-size: 14px; z-index: 1;
  animation: hint-bounce 2s infinite ease-in-out;
}
.home-bar {
  width: 130px; height: 5px; background: white; border-radius: 10px;
  margin: 15px auto 0;
}

/* 密码界面 */
.passcode-screen {
  position: absolute; top: 0; left: 0; width: 100%; height: 100%;
  display: flex; flex-direction: column; align-items: center;
  padding-top: 100px; z-index: 2;
}
.enter-passcode-text { font-size: 20px; margin-bottom: 30px; }

/* 密码点 */
.passcode-dots { display: flex; gap: 15px; margin-bottom: 50px; }
.dot {
  width: 14px; height: 14px; border-radius: 50%;
  border: 1.5px solid white; transition: all 0.2s;
}
.dot.filled { background: white; }
.shake { animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both; }

/* 数字键盘 */
.keypad { display: flex; flex-direction: column; gap: 20px; margin-bottom: 50px; }
.key-row { display: flex; gap: 25px; }
.key-btn {
  width: 75px; height: 75px; border-radius: 50%;
  background: rgba(255, 255, 255, 0.15);
  border: none; color: white;
  display: flex; flex-direction: column;
  justify-content: center; align-items: center;
  cursor: pointer; transition: background 0.2s;
  backdrop-filter: blur(5px); -webkit-tap-highlight-color: transparent;
}
.key-btn:active:not(.empty-key) { background: rgba(255, 255, 255, 0.4); }
.empty-key { background: transparent; cursor: default; }
.key-num { font-size: 32px; font-weight: 400; line-height: 1; }
.key-letters { font-size: 10px; letter-spacing: 1px; margin-top: 2px; font-weight: 600; }
.delete-icon { display: flex; align-items: center; justify-content: center; height: 100%; }

.cancel-btn {
  background: transparent; border: none; color: white;
  font-size: 16px; font-weight: 500; cursor: pointer;
  position: absolute; bottom: 50px; right: 40px;
}

/* 动画 */
@keyframes hint-bounce { 0%, 20%, 50%, 80%, 100% { transform: translateY(0); } 40% { transform: translateY(-10px); } 60% { transform: translateY(-5px); } }
@keyframes shake { 10%, 90% { transform: translate3d(-1px, 0, 0); } 20%, 80% { transform: translate3d(2px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-4px, 0, 0); } 40%, 60% { transform: translate3d(4px, 0, 0); } }

.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.fade-up-enter-active, .fade-up-leave-active { transition: all 0.4s ease; }
.fade-up-enter-from, .fade-up-leave-to { opacity: 0; transform: translateY(-20px); }

.slide-up-enter-active, .slide-up-leave-active { transition: all 0.5s cubic-bezier(0.19, 1, 0.22, 1); }
.slide-up-enter-from, .slide-up-leave-to { transform: translateY(100%); }
</style>