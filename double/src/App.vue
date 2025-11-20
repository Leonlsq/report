<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue' 
// 👇 请确保路径正确
import FireworksPage from './compoents/FireworksPage.vue'

// --- 0. 预加载状态 ---
const isLoading = ref(true)
const loadProgress = ref(0) // 加载进度 0-100

// --- 1. 数据配置区 ---
const slides = [
  {
    type: 'cover', 
    image: '/photos/cover.jpg', 
    printText: '14岁时的初遇，19岁时的重逢.\n这是我们的故事。', 
  },
  {
    type: 'content', 
    image: '/photos/0927.jpg', 
    date: '2025.09.27 - 🍞 命运的“切片”机',
    text: '都说这是‘德国留子对抗抑郁’的面包，但我没想到，它对抗孤独的效果更好。 我们的重逢，就始于这句关于超市面包机的闲聊。大概是面包机也没想到，它除了能切面包，还能顺便帮我‘切’回了一个女朋友。 如果那天没有那个关于面包机的如常对话，我们可能依然是两条平行的线。 命运有时候很幽默，它不用惊天动地的开场，只用一块超市里的大面包和一台自动切割机，就切开了我生活的裂缝，让光照了进来。 好幸运。',
    backgroundType: 'image', 
    backgroundImage: '/photos/1.jpg', 
  },
  {
    type: 'content',
    image: '/photos/0929.jpg',
    date: '2025.09.29 - 🤒 高烧后的‘冲动’',
    text: '以前的我，总觉得有些话太矫情，说不出口。直到那场突如其来的高烧，烧得我迷迷糊糊，却好像也把那个‘死要面子’的我也一并烧没了。躺在床上那一刻我突然明白，比被拒绝更可怕的，是来不及。于是深夜一点，我借着病后的‘冲动’，按下了那个迟到了几年的发送键。既然不想留遗憾，那就把真心话都告诉你。还好，那晚的勇气，换来了你的‘晚点回你’，也换来了我们的现在。',
    backgroundType: 'image', 
    backgroundImage: '/photos/2.jpg', 
  },
  {
    type: 'content',
    image: '/photos/0929-2.jpg',
    date: '2025.09.29 7:00 - 📲 谢谢你，没听我的话',
    text: '那时的我，别扭又忐忑，发完那一大段真心话就‘怂’了，特意补了一句‘别回我’。其实潜台词是——‘我很怕你真的不回’。无论出于何种原因，你总是会回。或许这就是我们要在一起的注定吧：我试图用‘别回我’来给自己留退路，而你用一句‘晚点回你’，堵住了我所有的胡思乱想，重新为我铺了一条走向你的路。\n你知道吗？看到你说‘没有对不起’，说那是一段‘纯真的画卷’时，我心里那块压了多年的石头终于落地了。谢谢你，没有听我的话；谢谢你，愿意接住那个笨拙的我，把我的‘遗憾’变成了我们共同的‘美好’。',
    backgroundType: 'image', 
    backgroundImage: '/photos/3.jpg', 
  },
  {
    type: 'content',
    // 拼贴照片组
    images: [
      '/photos/b1.jpeg', 
      '/photos/b2.JPG', 
      '/photos/b4.png'
    ],
    date: '2025.10.01 - 2025.10.05 - 🌧️ 慕尼黑的雨，与消失的三天',
    text: '刚刚搬家就和房东闹得一地鸡毛，让我在慕尼黑差点崩溃。有几天，我选择了‘消失’。不是不想找你，而是不敢。我看着满屋的狼藉，心里只有一个念头：‘隔着几千公里，你凭什么要在乎这么狼狈的我？’ 我怕我的负能量会把你吓跑，所以我想一个人扛。那时候的我还不懂，以为爱是只分享光鲜。但其实爱，是敢于把那个破碎的、狼狈的自己也拼凑进来，让你拥有一个完整的我。以后在你面前，笑的那个人是我，哭的那个人，同样也是我。',
    backgroundType: 'image',
    backgroundImage: '/photos/4.webp', 
  },
  {
    type: 'content',
    image: '/photos/c1.png',
    date: '2025.10.04 - 📹 7分30秒，热闹里的“暂停键”',
    text: '搬入新家那天，感觉又被治愈了，说来又是幸运的一次。我迫不及待地拍了这个视频发给你，虽然名字叫‘脱离苦海’，但心里想的其实是‘想和你分享这份安稳’。现在回看，让我感慨和触动的不是当时有多幸运找到这样一个房子，而是你说‘和朋友在一起，视频看了一半’。最后又补了一句‘看完了’。在你那天热闹的生活里，暂停了7分30秒，那是属于我的吗？',
    backgroundType: 'image', 
    backgroundImage: '/photos/c2.jpg', 
  },
  {
    type: 'letter', 
    image: "/photos/信.jpg",
    title: '致我最爱的女孩 (From Leon in Munich)',
    text: '亲爱的，生日快乐。\n\n从14岁到17岁，我们曾紧紧相依；中间走散的那两年，大概是命运为了让我们学会如何更好地相爱。还好，19岁的这年我们在9月27日又抓住了彼此。\n\n虽然刚重逢不久就要面对异地，我们很多时候一个人吃饭、一个人面对压力，我知道会有点辛苦。\n\n但请相信，我在 TUM 的每一次晚睡早起、攻克的每一个难关，都是为了填补那两年的空白，为了让我们未来的画卷不再有缺口。\n\n这是一个迟到了两年的生日礼物，请按下按钮，查收我的心意。',
    buttonText: '点击领取生日礼物😁'
  }
]

// --- 2. 逻辑控制区 ---
const currentIndex = ref(0)
const isAnimate = ref(false)
const displayedText = ref('') 
const cursorVisible = ref(true) 
const showFireworksPage = ref(false)
const audioRef = ref<HTMLAudioElement | null>(null)
const isMusicPlaying = ref(false)
const isTypingFinished = ref(false) 
const contentStep = ref(1)

const currentSlide = computed(() => slides[currentIndex.value])

// 🛠️ 自动拆分句子逻辑
const currentSlideSentences = computed(() => {
  const slide = currentSlide.value
  if (slide.type !== 'content' || !slide.text) return []
  return slide.text.replace(/。/g, '。|').split('|').map(s => s.trim()).filter(s => s)
})

const visibleSentences = computed(() => {
  return currentSlideSentences.value.slice(0, contentStep.value)
})

// 音乐控制
const toggleMusic = () => {
  if (audioRef.value) {
    if (isMusicPlaying.value) audioRef.value.pause()
    else audioRef.value.play()
    isMusicPlaying.value = !isMusicPlaying.value
  }
}

// 打字机效果
let typeInterval: number | null = null
const typewriterEffect = (text: string, delay = 100) => {
  isTypingFinished.value = false 
  displayedText.value = ''
  cursorVisible.value = true
  if (typeInterval) clearInterval(typeInterval)
  let i = 0
  typeInterval = setInterval(() => {
    if (i < text.length) {
      displayedText.value += text.charAt(i)
      i++
    } else {
      if (typeInterval) clearInterval(typeInterval)
      cursorVisible.value = false 
      isTypingFinished.value = true
    }
  }, delay)
}

// 监听翻页
watch(currentIndex, () => {
  contentStep.value = 1 
  if (currentSlide.value.type === 'cover' && currentSlide.value.printText) {
    typewriterEffect(currentSlide.value.printText)
  }
}, { immediate: true })

// 核心翻页交互逻辑
const nextSlide = () => {
  // 0. 如果还在加载中，禁止任何操作
  if (isLoading.value) return

  // ⭐⭐⭐ 修复点：如果已经是烟花页，不再执行任何翻页/音乐逻辑，避免重复加载音乐 ⭐⭐⭐
  if (showFireworksPage.value) {
    return
  }

  // 1. 封面打字未完成，禁止操作
  if (currentSlide.value.type === 'cover' && !isTypingFinished.value) {
    return 
  }

  // 2. 播放音乐逻辑 (如果没在播放，且不是最后一步，则尝试播放默认BGM)
  if (audioRef.value && audioRef.value.paused && !isMusicPlaying.value && currentIndex.value < slides.length - 1) {
    audioRef.value.play()
      .then(() => { isMusicPlaying.value = true })
      .catch((e) => console.log('等待交互播放', e))
  }

  // 3. 内容页逐句显示逻辑
  if (currentSlide.value.type === 'content') {
    if (contentStep.value < currentSlideSentences.value.length) {
      contentStep.value++
      return 
    }
  }

  // 4. 翻页和结束逻辑
  if (currentIndex.value === slides.length - 1) {
    // 在点击领取礼物时，强制切歌
    if (audioRef.value) {
      audioRef.value.pause()
      audioRef.value.src = '/music/你是我的风景.mp3'
      audioRef.value.load()
      audioRef.value.play()
        .then(() => { isMusicPlaying.value = true })
        .catch((e) => console.error('切歌失败', e))
    }
    
    // 显示烟花页
    showFireworksPage.value = true

  } else if (currentIndex.value < slides.length - 1) {
    // 普通翻页动画
    isAnimate.value = true
    setTimeout(() => {
      currentIndex.value++
      isAnimate.value = false
    }, 500) 
  }
}

// --- ⭐⭐⭐ 新增：图片预加载系统 ⭐⭐⭐ ---
const preloadImages = async () => {
  // 1. 提取所有需要加载的图片 URL
  const imageUrls: string[] = []
  
  slides.forEach(slide => {
    if (slide.image) imageUrls.push(slide.image)
    if (slide.backgroundImage) imageUrls.push(slide.backgroundImage)
    if (slide.images && slide.images.length > 0) {
      imageUrls.push(...slide.images)
    }
  })

  // 去重
  const uniqueUrls = [...new Set(imageUrls)]
  let loadedCount = 0

  // 2. 并行加载函数
  const loadSingleImage = (url: string) => {
    return new Promise((resolve) => {
      const img = new Image()
      img.src = url
      // 关键：图片解码完成后再 resolve，防止切页卡顿
      img.onload = () => {
        loadedCount++
        loadProgress.value = Math.floor((loadedCount / uniqueUrls.length) * 100)
        resolve(true)
      }
      img.onerror = () => {
        // 即使失败也要 resolve，防止卡死在 Loading 界面
        console.error(`Failed to load: ${url}`)
        loadedCount++
        loadProgress.value = Math.floor((loadedCount / uniqueUrls.length) * 100)
        resolve(false)
      }
    })
  }

  // 3. 开始加载
  await Promise.all(uniqueUrls.map(url => loadSingleImage(url)))
  
  // 4. 加载完毕，为了体验稍微延迟一下消失
  setTimeout(() => {
    isLoading.value = false
  }, 500)
}

// 在组件挂载时启动预加载
onMounted(() => {
  preloadImages()
})
</script>

<template>
  <div class="app-container" @click="nextSlide">
    
    <transition name="fade">
      <div v-if="isLoading" class="loading-overlay">
        <div class="loading-content">
          <div class="spinner"></div>
          <p>双祺正在整理回忆... {{ loadProgress }}%</p>
        </div>
      </div>
    </transition>

    <audio ref="audioRef" loop src="/music/伴奏.mp3"></audio>

    <div class="music-btn" @click.stop="toggleMusic" :class="{ 'playing': isMusicPlaying }">
      <div class="music-icon" :class="{ 'spinning': isMusicPlaying }">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
          <path d="M12 3v10.55c-.59-.34-1.27-.55-2-.55-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4V7h4V3h-6z"/>
        </svg>
      </div>
    </div>

    <div v-if="currentSlide.type !== 'content' || !currentSlide.backgroundType || currentSlide.backgroundType !== 'image'" class="bg-blob blob-1"></div>
    <div v-if="currentSlide.type !== 'content' || !currentSlide.backgroundType || currentSlide.backgroundType !== 'image'" class="bg-blob blob-2"></div>

    <transition name="fade" mode="out-in">
      <template v-if="!showFireworksPage">
        
        <div 
          v-if="currentSlide.type === 'cover'" 
          class="slide-section cover" 
          :key="currentIndex"
          :style="{ backgroundImage: `url(${currentSlide.image})` }"
        >
          <div class="overlay"></div> 
          <div class="content-box">
            <h1 class="typewriter-text">
              {{ displayedText }}
              <span v-if="cursorVisible" class="cursor">|</span>
            </h1>
            <p class="hint">✨ 点击屏幕开启回忆 ✨</p>
          </div>
        </div>

        <div 
          v-else-if="currentSlide.type === 'content'" 
          class="slide-section content" 
          :key="currentIndex"
          :style="currentSlide.backgroundType === 'image' ? { backgroundImage: `url(${currentSlide.backgroundImage})`, backgroundSize: 'cover', backgroundPosition: 'center' } : {}"
        >
          <div v-if="currentSlide.backgroundType === 'image'" class="background-overlay"></div> 

          <div class="content-main">
            
            <div v-if="currentSlide.images && currentSlide.images.length > 0" class="photo-collage">
              <div
                v-for="(imgSrc, index) in currentSlide.images"
                :key="index"
                class="polaroid-mini"
                :class="`collage-${index + 1}`"
              >
                <img :src="imgSrc" alt="Memory" />
              </div>
            </div>

            <div v-else class="polaroid">
              <img :src="currentSlide.image" alt="Memory" />
            </div>
            
            <div class="text-area">
              <span class="date-tag">{{ currentSlide.date }}</span>
              <p 
                v-for="(sentence, index) in visibleSentences" 
                :key="index"
                class="sentence-item"
              >
                {{ sentence }}
              </p>
            </div>
          </div>
        </div>

        <div 
          v-else-if="currentSlide.type === 'letter'" 
          class="slide-section letter" 
          :key="currentIndex"
          :style="{ backgroundImage: `url(${currentSlide.image})` }"
        >
          <div class="overlay"></div>
          <div class="letter-paper">
            <h2>{{ currentSlide.title }}</h2>
            <p style="white-space: pre-line;">{{ currentSlide.text }}</p>
            <button class="gift-btn" @click.stop="nextSlide">{{ currentSlide.buttonText }}</button>
          </div>
        </div>

      </template>
      <FireworksPage v-else />
    </transition>

    <div v-if="!showFireworksPage" class="progress-bar">
      <div class="progress-inner" :style="{ width: ((currentIndex + 1) / slides.length) * 100 + '%' }"></div>
    </div>
  </div>
</template>

<style>
/* --- 3. 样式区 --- */
:root {
  --bg-color: #fdfcf8;
  --primary: #e4b1ab; 
  --text-main: #5d5d5d;
  --text-light: #8a8a8a;
}

body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Microsoft YaHei", Arial, sans-serif;
  background-color: var(--bg-color);
  overflow: hidden; 
}

.app-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  background: linear-gradient(135deg, #fefdfb 0%, #fcebeb 100%); 
}

/* 🎵 音乐按钮 */
.music-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 100; 
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(5px);
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1);
  border: 1px solid rgba(255,255,255,0.5);
  transition: all 0.3s ease;
}
.music-btn:hover, .music-btn.playing {
  background: rgba(255, 255, 255, 0.8);
  transform: scale(1.1);
}
.music-icon { color: var(--primary); display: flex; align-items: center; justify-content: center; }
.spinning { animation: spin 3s linear infinite; }
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

/* 背景光斑 */
.bg-blob {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  opacity: 0.6;
  z-index: 0;
  animation: float 10s infinite ease-in-out;
}
.blob-1 { width: 300px; height: 300px; background: #ffe4e1; top: -50px; left: -50px; }
.blob-2 { width: 400px; height: 400px; background: #e6e6fa; bottom: -100px; right: -100px; animation-delay: -5s; }
@keyframes float { 0%, 100% { transform: translate(0, 0); } 50% { transform: translate(30px, 50px); } }

/* 通用 Slide 布局 */
.slide-section {
  z-index: 10;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  transition: background-image 0.5s ease;
}

/* 遮罩层 (Cover & Letter) */
.slide-section.cover .overlay, .slide-section.letter .overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4); 
  backdrop-filter: blur(5px);
  z-index: 1;
}
.slide-section.letter .overlay {
  background: rgba(255, 255, 255, 0.3); 
  backdrop-filter: blur(3px);
}

/* Cover 内容 */
.slide-section.cover {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.slide-section.cover .content-box {
  z-index: 2;
  max-width: 85%; 
  padding: 20px;
  width: 800px; 
  text-align: center;
}
.cover h1, .cover .typewriter-text {
  font-size: clamp(1.5rem, 5vw, 2.5rem); 
  color: white;
  margin-bottom: 20px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  white-space: pre-wrap; 
  line-height: 1.4;
}
.typewriter-text .cursor {
  display: inline-block;
  background-color: white;
  width: 3px;
  height: 1.1em;
  vertical-align: middle;
  margin-left: 5px;
  animation: blink-cursor 0.8s infinite step-end;
}
@keyframes blink-cursor { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
.hint {
  margin-top: 50px;
  font-size: 0.9rem;
  color: #f0f0f0;
  animation: pulse 2s infinite;
  text-align: center;
}

/* --- Content 页布局 --- */
.slide-section.content {
  display: flex; 
  align-items: center; 
  justify-content: center; 
  padding: 20px; 
}

.slide-section.content .background-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0.3); 
  backdrop-filter: blur(8px); 
  z-index: 1; 
}

/* 内容主容器 */
.content-main {
  display: flex; 
  flex-direction: row; 
  align-items: center; 
  justify-content: center;
  gap: 40px; 
  max-width: 1100px; 
  width: 90%; 
  z-index: 2; 
  position: relative; 
  background: rgba(255, 255, 255, 0.6); 
  border-radius: 20px;
  padding: 15px 40px; 
  box-shadow: 0 10px 30px rgba(0,0,0,0.05);
  backdrop-filter: blur(5px); 
  overflow: visible !important; 
}

/* 左侧拍立得 (单张) */
.polaroid {
  flex-shrink: 0; 
  width: 450px;  
  background: white;
  padding: 15px 15px 60px 15px;
  box-shadow: 0 25px 50px rgba(0,0,0,0.2), 
              0 0 0 2px rgba(0,0,0,0.02) inset; 
  margin-left: -140px; 
  margin-top: -60px; 
  margin-bottom: -60px; 
  transform: rotate(-5deg); 
  z-index: 10; 
  border-radius: 4px; 
  transition: transform 0.3s;
}
.polaroid img {
  width: 100%;
  height: auto;
  object-fit: cover;
  border-radius: 2px;
  display: block;
}
.polaroid:hover {
  transform: rotate(0deg) scale(1.05); 
  box-shadow: 0 30px 70px rgba(0,0,0,0.3);
  z-index: 20;
}

/* 右侧文字区 */
.text-area {
  flex-grow: 1; 
  text-align: left; 
  min-width: 0; 
  padding-left: 10px;
}
.date-tag {
  background: var(--primary);
  color: white;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: bold;
  display: inline-block;
  margin-bottom: 20px;
  box-shadow: 0 4px 10px rgba(228, 177, 171, 0.4);
}
@keyframes soft-float-up {
  0% { opacity: 0; transform: translateY(15px); }
  100% { opacity: 1; transform: translateY(0); }
}
.text-area p.sentence-item {
  margin: 8px 0;
  color: var(--text-main);
  line-height: 1.8;
  white-space: pre-line;
  font-family: 'ZCOOL KuaiLe', cursive, "Microsoft YaHei", sans-serif;
  font-size: 1.4rem;
  animation: soft-float-up 4.0s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

/* --- 📱 移动端适配 --- */
@media (max-width: 768px) {
  .content-main {
    flex-direction: column;
    gap: 30px; 
    padding: 25px; 
    width: 95%;
    margin-top: 20px;
    overflow: hidden !important; 
  }
  .polaroid {
    width: 280px; 
    margin-left: 0; 
    margin-top: 0;
    margin-bottom: 0;
    padding-bottom: 40px;
    transform: rotate(-2deg);
  }
  .text-area {
    text-align: center; 
    width: 100%;
    padding-left: 0;
  }
  .text-area p.sentence-item {
    font-size: 1.1rem;
    margin: 8px 0;
  }
}

/* --- 信件页 (Letter) 样式 --- */
.slide-section.letter {
  display: flex;
  align-items: center;
  justify-content: center;
}
.letter-paper {
  background: rgba(255, 255, 255, 0.95); 
  padding: 40px;
  border-radius: 10px;
  box-shadow: 0 10px 40px rgba(0,0,0,0.15);
  text-align: left;
  width: 85%;
  max-width: 600px;
  z-index: 2; 
  position: relative;
  max-height: 80vh;
  overflow-y: auto; 
}
.letter-paper h2 {
  color: var(--primary);
  border-bottom: 1px solid #e0e0e0;
  padding-bottom: 15px;
  margin-bottom: 20px;
  font-weight: normal;
}
.letter-paper p {
  font-family: "Microsoft YaHei", sans-serif; 
  font-size: 1.1rem;
  line-height: 2;
  color: var(--text-main);
  white-space: pre-line;
}
.gift-btn {
  margin-top: 30px;
  width: 100%;
  padding: 15px;
  background: var(--primary);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}
.gift-btn:hover { opacity: 0.9; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(228, 177, 171, 0.4); }

.progress-bar {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 4px;
  background: rgba(255,255,255,0.3);
  z-index: 20;
}
.progress-inner { height: 100%; background: var(--primary); transition: width 0.5s ease; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease, transform 0.5s ease; }
.fade-enter-from { opacity: 0; transform: translateY(20px); }
.fade-leave-to { opacity: 0; transform: translateY(-20px); }
@keyframes pulse { 0% { opacity: 0.5; } 50% { opacity: 1; } 100% { opacity: 0.5; } }

/* --- 照片拼贴容器 (四角分散版) --- */
.photo-collage {
  flex-shrink: 0;
  width: 550px;  /* 宽度加大，足够放下两列 */
  height: 500px; /* 高度加大，足够放下两行 */
  position: relative;
  /* 保持破框效果，稍微调整边距 */
  margin-left: -120px; 
  margin-top: -50px; 
  margin-bottom: -50px;
  z-index: 10;
}

/* 迷你拍立得通用样式 */
.polaroid-mini {
  position: absolute;
  background: white;
  padding: 8px 8px 35px 8px; /* 底部留白给手写字的感觉 */
  box-shadow: 0 10px 25px rgba(0,0,0,0.15),
              0 0 0 1px rgba(0,0,0,0.02) inset;
  border-radius: 4px;
  transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1); 
}
.polaroid-mini img {
  width: 100%;
  height: auto;
  object-fit: cover;
  border-radius: 2px;
  display: block;
}

/* 🖱️ 悬停效果：放大并置顶 */
.polaroid-mini:hover {
  z-index: 100 !important; /* 确保绝对置顶 */
  transform: scale(1.2) rotate(0deg) !important; /* 放大倍数增加，方便查看细节 */
  box-shadow: 0 30px 60px rgba(0,0,0,0.3);
}

/* --- 四张照片的具体布局 (田字格分布) --- */

/* 1. 左上 */
.collage-1 {
  width: 210px;
  top: 10px;
  left: 10px;
  transform: rotate(-6deg);
  z-index: 11;
}

/* 2. 右上 */
.collage-2 {
  width: 200px;
  top: 30px;
  right: 10px;
  transform: rotate(5deg);
  z-index: 12;
}

/* 3. 左下 */
.collage-3 {
  width: 210px;
  bottom: 20px;
  left: 20px;
  transform: rotate(3deg);
  z-index: 13;
}

/* 4. 右下 */
.collage-4 {
  width: 200px;
  bottom: 10px;
  right: 10px;
  transform: rotate(-4deg);
  z-index: 14;
}


/* --- 📱 移动端适配 (缩小版田字格) --- */
@media (max-width: 768px) {
  .photo-collage {
    width: 340px;
    height: 320px;
    margin-left: 0;
    margin-top: 10px;
    margin-bottom: 0;
  }

  /* 统一缩小尺寸 */
  .collage-1, .collage-2, .collage-3, .collage-4 {
    width: 145px;
  }

  /* 移动端位置微调 */
  .collage-1 { top: 0; left: 0; transform: rotate(-5deg); }
  .collage-2 { top: 10px; right: 0; transform: rotate(4deg); }
  .collage-3 { bottom: 10px; left: 5px; transform: rotate(3deg); }
  .collage-4 { bottom: 0; right: 5px; transform: rotate(-3deg); }
}

/* --- Loading 样式 --- */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #fdfcf8;
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
}
.loading-content {
  text-align: center;
  color: var(--primary);
}
.loading-content p {
  margin-top: 20px;
  font-size: 1.2rem;
  letter-spacing: 2px;
  font-family: "Microsoft YaHei", sans-serif;
}
.spinner {
  width: 50px;
  height: 50px;
  border: 3px solid rgba(228, 177, 171, 0.3);
  border-radius: 50%;
  border-top-color: var(--primary);
  animation: spin-loading 1s ease-in-out infinite;
  margin: 0 auto;
}
@keyframes spin-loading {
  to { transform: rotate(360deg); }
}
</style>