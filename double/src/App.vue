<script setup lang="ts">
import { ref, computed, watch, onMounted, nextTick } from 'vue' 
// 👇 请确保路径正确
import FireworksPage from './compoents/FireworksPage.vue'
// 👇 引入新的解锁组件
import iOSUnlockPage from './compoents/iOSUnlockPage.vue'

// --- 0. 状态管理 ---
const isLoading = ref(true)
const loadProgress = ref(0)
const showDeviceSelector = ref(false) 
const deviceMode = ref('') 
const isLocked = ref(true) 

// --- 1. 数据配置区 ---
const slides = [
  {
    type: 'cover', 
    image: '/photos/cover.jpg', 
    printText: '14岁时的初遇，19岁时的重逢.\n这是我们的故事。', 
  },
  {
    type: 'content', 
    title: '🍞 命运的“切片”机',
    image: '/photos/0927.jpg', 
    date: '2025.09.27',
    text: '都说这是‘德国留子对抗抑郁’的面包，但我没想到，它对抗孤独的效果更好。 我们的重逢，就始于这句关于超市面包机的闲聊。大概是面包机也没想到，它除了能切面包，还能顺便帮我‘切’回了一个女朋友。 如果那天没有那个关于面包机的如常对话，我们可能依然是两条平行的线。 命运有时候很幽默，它不用惊天动地的开场，只用一块超市里的大面包和一台自动切割机，就切开了我生活的裂缝，让光照了进来。 好幸运。',
    backgroundType: 'image', 
    backgroundImage: '/photos/1.jpg', 
  },
  {
    type: 'content',
    title: '🤒 高烧后的‘冲动’',
    image: '/photos/0929.jpg',
    date: '2025.09.29',
    text: '以前的我，总觉得有些话太矫情，说不出口。直到那场突如其来的高烧，烧得我迷迷糊糊，却好像也把那个‘死要面子’的我也一并烧没了。躺在床上那一刻我突然明白，比被拒绝更可怕的，是来不及。于是深夜一点，我借着病后的‘冲动’，按下了那个迟到了几年的发送键。既然不想留遗憾，那就把真心话都告诉你。还好，那晚的勇气，换来了你的‘晚点回你’，也换来了我们的现在。',
    backgroundType: 'image', 
    backgroundImage: '/photos/2.jpg', 
  },
  {
    type: 'content',
    title: '📲 谢谢你，没听我的话',
    image: '/photos/0929-2.jpg',
    date: '2025.09.29 7:00',
    text: '那时的我，别扭又忐忑，发完那一大段真心话就‘怂’了，特意补了一句‘别回我’。其实潜台词是——‘我很怕你真的不回’。但好像，无论出于何种原因，你总是会回。或许这就是我们要在一起的注定吧：我试图用‘别回我’来给自己留退路，而你用一句‘晚点回你’，堵住了我所有的胡思乱想，重新为我铺了一条走向你的路。\n你知道吗？看到你说‘没有对不起’，说那是一段‘纯真的画卷’时，我心里那块压了多年的石头终于落地了。谢谢你，没有听我的话；谢谢你，愿意接住那个笨拙的我，把我的‘遗憾’变成了我们共同的‘美好’。',
    backgroundType: 'image', 
    backgroundImage: '/photos/3.jpg', 
  },
  {
    type: 'content',
    title: '🌧️ 慕尼黑的雨，与消失的三天',
    images: [
      '/photos/b1.jpeg', 
      '/photos/b2.JPG', 
      '/photos/b4.png'
    ],
    date: '2025.10.01 - 2025.10.05',
    text: '刚重逢的喜悦还没散去，现实就给我上了一课。遇到的奇葩房东和搬家的一地鸡毛，让我在慕尼黑差点崩溃。\n\n那几天，我选择了‘消失’。不是不想找你，而是胆怯和纠结。我看着满屋的狼藉，心里只有一个念头：‘隔着几千公里，你凭什么要在乎如此狼狈的我？’ 我怕我的负能量会把你吓跑，所以我想一个人扛。\n\n那时候的我以为，爱是只分享光鲜。但其实爱，是敢于把那个破碎的、狼狈的自己也拼凑进来，让你拥有一个完整的我。\n\n从此以后，笑的那个人是我，哭的那个人，同样也是我。',
    backgroundType: 'image',
    backgroundImage: '/photos/4.webp', 
  },
  {
    type: 'content',
    title: '📹 7分30秒，热闹里的“暂停键”',
    image: '/photos/c1.png',
    date: '2025.10.04 - 脱离苦海',
    text: '搬进新家那天，感觉又被治愈了，说来又是幸运的一次。我迫不及待地拍了这个视频发给你，虽然名字叫‘脱离苦海’，但心里想的其实是‘想和你分享这份安稳’。\n\n现在回看，让我感慨和触动的不是当时有多幸运找到这样一个房子，而是你说‘和朋友在一起，视频看了一半’，最后又补了一句‘看完了’。\n\n热闹的生活里被暂停的7分30秒，是属于我的吗。这份‘在意’，比安逸的新房子更让我心安。',
    backgroundType: 'image', 
    backgroundImage: '/photos/c2.jpg', 
  },
  // 📸 10.10 Gallery
  {
    type: 'gallery',
    title: '被分享欲填满的10月10日',
    date: '2025.10.10',
    backgroundType: 'image',
    backgroundImage: '/photos/记录/1010/5.jpg', 
    gallery: [
      { img: '/photos/记录/1010/1.jpg', text: '早起看见忘给小孩改作业的你' },
      { img: '/photos/记录/1010/2.jpg', text: '接着就是早上还没下班的月亮(然后我就去市政厅办事了😣)' },
      { img: '/photos/记录/1010/m.JPG', text: '好可爱的寻猫启示。我也喜欢猫🐱' },
      { img: '/photos/记录/1010/4.jpg', text: '刚给你看完欧洲超市的可颂就自己没忍住买一个吃🥐' },
      { img: '/photos/记录/1010/66.jpg', text: '你也在逛超市。聊完咖啡我决定给你买点咖啡粉试试☕️' },
      { img: '/photos/记录/1010/6.jpg', text: '国内油油的面包（除了山姆！）' },
      { img: '/photos/记录/1010/7.jpg', text: '回到家吃刚刚买回来的速冻披萨(留子经典出装)' },
      { img: '/photos/记录/1010/33.jpg', text: '看起来好好吃的面包😋，我说放心吃热量算我的，你说我害人[捂嘴笑]。' },
      { img: '/photos/记录/1010/8.jpg', text: '最后还是买回去吃了...好好吃' },
      { img: '/photos/记录/1010/9.png', text: '要我倒掉的蘑菇意面(不吃蘑菇！)' }
    ],
    text: '从我这边的清晨七点，一直聊到你那边的日落黄昏。以前觉得“永远有话说”是一种能力，现在才发现，那是因为遇到对的人。虽然相隔万里，但感觉就像在互相的耳边碎碎念。隔着几千公里和六个小时的时差，我们的生活依然能严丝合缝地拼在一起，真好。',
  },

  {
    type: 'gallery',
    title: '记录生活的我',
    date: '2025.10',
    backgroundType: 'image',
    backgroundImage: '/photos/记录/back1.jpeg', 
    gallery: [
      { img: '/photos/记录/12.png', text: '被剪发哥剪出一个秃顶的我' },
      { img: '/photos/记录/13.JPG', text: '😋早上煮出了溏心蛋🥚' },
      { img: '/photos/记录/14.png', text: '来自中国的锁' },
      { img: '/photos/记录/15.JPG', text: '弯弯的房东送我的辣椒酱拉🌶️' },
      { img: '/photos/记录/8.jpeg', text: '健康的辣椒炒鸡腿肉🍗' },
      { img: '/photos/记录/back.JPG', text: '临近秋天学校里好漂亮的秋景🍂' },
      { img: '/photos/记录/4.jpeg', text: '小花园里的野猫，黑黑的🐈‍⬛' },
      { img: '/photos/记录/2.jpeg', text: '好久没吃月饼🥮了😭' },
      { img: '/photos/记录/3.jpeg', text: '公园吃cheese cake和寿司(后面被蜜蜂🐝追着跑)' },
      { img: '/photos/记录/16.JPG', text: '准备辣椒炒肉中（腊肉怎么是红红的）' },
      { img: '/photos/记录/25.jpeg', text: '咖啡哥带我在市中心喝☕️（又是被咖啡哥洗礼的一天）' },
      { img: '/photos/记录/28.jpeg', text: '好漂亮的公园⛲️（ipad壁纸）' },
      { img: '/photos/记录/26.JPG', text: '用同一个杯子的同一级的计算机土耳其老哥' },
      { img: '/photos/记录/27.jpeg', text: '第一次参加学校的课' },
      { img: '/photos/记录/30.JPG', text: '第一次吃学校饭堂的白人餐(😣)' },
      { img: '/photos/记录/29.JPG', text: '开学典礼（TUM🧢）' },
      { img: '/photos/记录/31.JPG', text: '我当时说要留这样的胡子🧔哈哈哈' },
      { img: '/photos/记录/33.jpeg', text: '城墙猪排难吃哦' },
      { img: '/photos/记录/5.jpg', text: '回家路上你说好看的树' }
    ],
    text: '这里有深秋的树、有来自中国的锁、还有我努力照顾好自己的证据（虽然偶尔也有翻车的时候）。从被剪坏的头发到TUM的开学典礼，从第一次尝试的学校‘白人餐’到路边偶遇的黑猫。这些看似毫无关联的碎片，拼凑出了我在慕尼黑的十月。其实生活里哪有那么多惊天动地的大事，大多都是这些鸡毛蒜皮、无关紧要。但我还是想把这些微不足道的瞬间都打包寄给你。因为在这个世界上，我所有的分享欲，所有的碎碎念，还有那些想说未说的话，都只想留给你。',
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
const currentIndex = ref(5) 
const isAnimate = ref(false)
const displayedText = ref('') 
const cursorVisible = ref(true) 
const showFireworksPage = ref(false)
const audioRef = ref<HTMLAudioElement | null>(null)
const isMusicPlaying = ref(false)
const isTypingFinished = ref(false) 
const contentStep = ref(1)

const currentSlide = computed(() => slides[currentIndex.value])

// 修改：允许 Content 和 Gallery 都能处理分句逻辑
const currentSlideSentences = computed(() => {
  const slide = currentSlide.value
  if ((slide.type !== 'content' && slide.type !== 'gallery') || !slide.text) return []
  return slide.text.replace(/。/g, '。|').split('|').map(s => s.trim()).filter(s => s)
})

// 修改：根据页面类型计算可见的文字数量
const visibleSentences = computed(() => {
  const slide = currentSlide.value

  // 1. Gallery 模式逻辑：步数减去图片数量
  if (slide.type === 'gallery') {
    const galleryCount = slide.gallery?.length || 0
    const textStep = Math.max(0, contentStep.value - galleryCount)
    return currentSlideSentences.value.slice(0, textStep)
  }

  // 2. 第6页(Index 5)特殊逻辑：步数减去图片数量
  if (currentIndex.value === 5 && slide.images) {
    const imageCount = slide.images.length
    const textStep = Math.max(0, contentStep.value - imageCount)
    return currentSlideSentences.value.slice(0, textStep)
  }
  
  // 3. 普通 Content 逻辑
  return currentSlideSentences.value.slice(0, contentStep.value)
})

const toggleMusic = () => {
  if (audioRef.value) {
    if (isMusicPlaying.value) audioRef.value.pause()
    else audioRef.value.play()
    isMusicPlaying.value = !isMusicPlaying.value
  }
}

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

// 监听解锁
watch([currentIndex, isLocked], ([newIndex, newLockedState]) => {
  if (newLockedState) return 

  contentStep.value = 1 
  if (currentSlide.value.type === 'cover' && currentSlide.value.printText) {
    typewriterEffect(currentSlide.value.printText)
  }
})

const nextSlide = () => {
  if (isLoading.value || showDeviceSelector.value || isLocked.value) return
  if (showFireworksPage.value) return

  if (currentSlide.value.type === 'cover' && !isTypingFinished.value) {
    return 
  }

  if (audioRef.value && audioRef.value.paused && !isMusicPlaying.value && currentIndex.value < slides.length - 1) {
    audioRef.value.play()
      .then(() => { isMusicPlaying.value = true })
      .catch((e) => console.log('等待交互播放', e))
  }

  // ⭐⭐⭐ 自动滚动与分步逻辑 (兼容 Content 和 Gallery) ⭐⭐⭐
  const isContent = currentSlide.value.type === 'content'
  const isGallery = currentSlide.value.type === 'gallery'

  if (isContent || isGallery) {
    let totalSteps = 0

    if (isGallery) {
      // Gallery页：总步数 = 照片数 + 句子数
      totalSteps = (currentSlide.value.gallery?.length || 0) + currentSlideSentences.value.length
    } else {
      // 第6页(Index 5)：总步数 = 照片数 + 句子数
      if (currentIndex.value === 5 && currentSlide.value.images) {
        totalSteps = currentSlide.value.images.length + currentSlideSentences.value.length
      } else {
        // 其他Content页：总步数 = 句子数
        totalSteps = currentSlideSentences.value.length
      }
    }

    if (contentStep.value < totalSteps) {
      contentStep.value++
      
      // 使用 nextTick + 延时，确保手机端能滚到底
      nextTick(() => {
        setTimeout(() => {
          const container = document.querySelector('.mode-mobile .content-main') || document.querySelector('.gallery-container')
          if (container) {
            container.scrollTo({
              top: container.scrollHeight + 2000, 
              behavior: 'smooth'
            })
          }
        }, 100) 
      })

      return // 步数没走完，不翻页
    }
  }

  if (currentIndex.value === slides.length - 1) {
    if (audioRef.value) {
      audioRef.value.pause()
      audioRef.value.src = '/music/你是我的风景.mp3'
      audioRef.value.load()
      audioRef.value.play()
        .then(() => { isMusicPlaying.value = true })
        .catch((e) => console.error('切歌失败', e))
    }
    showFireworksPage.value = true
  } else if (currentIndex.value < slides.length - 1) {
    isAnimate.value = true
    setTimeout(() => {
      currentIndex.value++
      isAnimate.value = false
      // 翻页后重置滚动位置
      nextTick(() => {
        const container = document.querySelector('.mode-mobile .content-main')
        if (container) container.scrollTop = 0
      })
    }, 500) 
  }
}

// --- 设备选择 ---
const selectDevice = (mode: string) => {
  deviceMode.value = mode
  showDeviceSelector.value = false
}

// --- 解锁 ---
const handleUnlock = () => {
  isLocked.value = false 
  if (audioRef.value) {
    audioRef.value.play().then(() => { isMusicPlaying.value = true }).catch(() => { })
  }
}

// --- 预加载 ---
const preloadImages = async () => {
  const imageUrls: string[] = []
  slides.forEach(slide => {
    if (slide.image) imageUrls.push(slide.image)
    if (slide.backgroundImage) imageUrls.push(slide.backgroundImage)
    if (slide.images && slide.images.length > 0) {
      imageUrls.push(...slide.images)
    }
    // 预加载 gallery 里的图
    if (slide.gallery && slide.gallery.length > 0) {
      slide.gallery.forEach(item => imageUrls.push(item.img))
    }
  })
  imageUrls.push('/photos/cover.jpg')

  const uniqueUrls = [...new Set(imageUrls)]
  let loadedCount = 0

  const loadSingleImage = (url: string) => {
    return new Promise((resolve) => {
      const img = new Image()
      img.src = url
      img.onload = () => {
        loadedCount++
        loadProgress.value = Math.floor((loadedCount / uniqueUrls.length) * 100)
        resolve(true)
      }
      img.onerror = () => {
        loadedCount++
        loadProgress.value = Math.floor((loadedCount / uniqueUrls.length) * 100)
        resolve(false)
      }
    })
  }

  await Promise.all(uniqueUrls.map(url => loadSingleImage(url)))
  setTimeout(() => {
    isLoading.value = false
    showDeviceSelector.value = true 
  }, 500)
}

onMounted(() => {
  preloadImages()
})
</script>

<template>
  <div class="app-container" :class="[`mode-${deviceMode}`]" @click="nextSlide">
    
    <transition name="fade">
      <div v-if="isLoading" class="loading-overlay">
        <div class="loading-content">
          <div class="spinner"></div>
          <p>双祺正在整理回忆... {{ loadProgress }}%</p>
        </div>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="!isLoading && showDeviceSelector" class="device-selector-overlay">
        <div class="selector-box">
          <h2>亲爱的Ty你是在用哪个设备打开我们的网站的吖😊</h2>
          <p>✨选择你的设备✨</p>
          <div class="btn-group">
            <button @click.stop="selectDevice('mobile')">📱 手机 iPhone</button>
            <button @click.stop="selectDevice('tablet')">📟 平板 iPad</button>
            <button @click.stop="selectDevice('desktop')">💻 电脑 Mac/PC(体验最佳)</button>
          </div>
        </div>
      </div>
    </transition>

    <transition name="fade">
      <iOSUnlockPage 
        v-if="!isLoading && !showDeviceSelector && isLocked" 
        @unlocked="handleUnlock" 
      />
    </transition>

    <audio ref="audioRef" loop src="/music/伴奏.mp3"></audio>

    <div v-if="!isLoading && !showDeviceSelector && !isLocked" class="music-btn" @click.stop="toggleMusic" :class="{ 'playing': isMusicPlaying }">
      <div class="music-icon" :class="{ 'spinning': isMusicPlaying }">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
          <path d="M12 3v10.55c-.59-.34-1.27-.55-2-.55-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4V7h4V3h-6z"/>
        </svg>
      </div>
    </div>

    <div v-if="currentSlide.type !== 'content' || !currentSlide.backgroundType || currentSlide.backgroundType !== 'image'" class="bg-blob blob-1"></div>
    <div v-if="currentSlide.type !== 'content' || !currentSlide.backgroundType || currentSlide.backgroundType !== 'image'" class="bg-blob blob-2"></div>

    <transition name="fade" mode="out-in">
      <template v-if="!showFireworksPage && !isLoading && !showDeviceSelector && !isLocked">
        
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
                class="polaroid-mini pop-in-effect"
                :class="[`collage-${index + 1}`, { 'delayed-show': currentIndex === 5 && contentStep <= index }]"
              >
                <img :src="imgSrc" alt="Memory" />
              </div>
            </div>

            <div v-else class="polaroid">
              <img :src="currentSlide.image" alt="Memory" />
            </div>
            
            <div class="text-area">
              <h3 v-if="currentSlide.title" class="slide-title">{{ currentSlide.title }}</h3>
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
          v-else-if="currentSlide.type === 'gallery'" 
          class="slide-section content gallery-mode" 
          :key="currentIndex"
          :style="{ backgroundImage: `url(${currentSlide.backgroundImage})` }"
        >
          <div class="background-overlay"></div>
          
          <div class="content-main gallery-container">
            <h3 class="gallery-title">{{ currentSlide.title }}</h3>
            
            <div class="gallery-grid">
              <div 
                v-for="(item, index) in currentSlide.gallery" 
                :key="index" 
                class="gallery-item"
                v-show="index < contentStep"
              >
                <div class="polaroid-mini-card">
                  <img :src="item.img" loading="lazy" />
                </div>
                <p class="gallery-text">{{ item.text }}</p>
              </div>
            </div>
            
            <div class="gallery-text-area" v-if="currentSlide.text">
              <p 
                v-for="(sentence, index) in visibleSentences" 
                :key="index"
                class="sentence-item gallery-sentence"
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
      <FireworksPage v-else-if="showFireworksPage" />
    </transition>

    <div v-if="!showFireworksPage && !isLoading && !showDeviceSelector && !isLocked" class="progress-bar">
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

/* --- 设备适配逻辑 --- */
.app-container.mode-tablet .content-main {
  transform: scale(0.85); 
  width: 95%;
}

/* Mobile (Phone) */
.app-container.mode-mobile .content-main {
  display: flex !important;
  flex-direction: column !important;
  align-items: center !important;
  justify-content: flex-start !important;
  width: 85vw !important; 
  max-width: 380px !important;
  height: auto;
  max-height: 80vh;
  padding: 30px 20px 100px 20px !important;
  margin: 20px auto !important; 
  left: auto !important;
  right: auto !important;
  transform: none !important;
  gap: 20px;
  overflow-y: auto !important; 
  overflow-x: hidden !important;
  -webkit-overflow-scrolling: touch;
  box-sizing: border-box !important;
}

.app-container.mode-mobile .polaroid {
  margin: 0 !important; 
  width: 200px !important;
  padding: 10px 10px 35px 10px !important;
  align-self: center !important;
  transform: rotate(-2deg) !important;
  position: relative !important;
  left: auto !important;
  top: auto !important;
  flex-shrink: 0 !important;
}

.app-container.mode-mobile .photo-collage {
  margin: 0 !important;
  width: 260px !important;
  height: 230px !important;
  align-self: center !important;
  transform-origin: center center !important;
  transform: scale(0.95) !important;
  position: relative !important;
  left: auto !important;
  top: auto !important;
  flex-shrink: 0 !important;
}

.app-container.mode-mobile .collage-1, 
.app-container.mode-mobile .collage-2, 
.app-container.mode-mobile .collage-3, 
.app-container.mode-mobile .collage-4 {
  width: 120px !important; 
}
.app-container.mode-mobile .collage-1 { left: 10px !important; top: 0 !important; }
.app-container.mode-mobile .collage-2 { right: 10px !important; top: 10px !important; }
.app-container.mode-mobile .collage-3 { left: 20px !important; bottom: 10px !important; }
.app-container.mode-mobile .collage-4 { right: 20px !important; bottom: 0 !important; }

.app-container.mode-mobile .text-area {
  width: 100% !important;
  text-align: center !important;
  padding: 0 !important; 
  margin: 0 !important;
}

.app-container.mode-mobile .slide-title {
  font-size: 1.25rem !important;
  margin: 5px 0 10px 0 !important;
  text-align: center !important;
  display: block !important;
  width: 100% !important;
}

.app-container.mode-mobile .date-tag {
  font-size: 0.85rem !important;
  padding: 4px 14px !important;
  margin: 0 auto 15px auto !important;
  display: inline-block !important;
}

.app-container.mode-mobile .text-area p.sentence-item {
  font-size: 1rem !important;
  line-height: 1.7 !important;
  margin: 6px 0 !important;
  text-align: center !important;
}

/* --- 设备选择遮罩样式 --- */
.device-selector-overlay {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px);
  z-index: 10000; display: flex; justify-content: center; align-items: center;
}
.selector-box {
  text-align: center; background: white; padding: 40px; border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.1); border: 1px solid rgba(0,0,0,0.05);
}
.selector-box h2 { color: var(--text-main); margin-bottom: 10px; }
.selector-box p { color: var(--text-light); margin-bottom: 30px; }
.btn-group { display: flex; flex-direction: column; gap: 15px; }
.btn-group button {
  padding: 15px 30px; border: 1px solid #eee; background: white;
  border-radius: 12px; font-size: 1rem; cursor: pointer;
  transition: all 0.3s; color: var(--text-main); font-weight: 500;
}
.btn-group button:hover {
  background: var(--primary); color: white; border-color: var(--primary);
  transform: translateY(-2px); box-shadow: 0 5px 15px rgba(228, 177, 171, 0.4);
}

/* --- 其他通用样式 --- */
.music-btn {
  position: absolute; top: 20px; right: 20px; z-index: 100; width: 40px; height: 40px;
  background: rgba(255, 255, 255, 0.2); backdrop-filter: blur(5px); border-radius: 50%;
  display: flex; justify-content: center; align-items: center; cursor: pointer;
  box-shadow: 0 4px 10px rgba(0,0,0,0.1); border: 1px solid rgba(255,255,255,0.5);
  transition: all 0.3s ease;
}
.music-btn:hover, .music-btn.playing { background: rgba(255, 255, 255, 0.8); transform: scale(1.1); }
.music-icon { color: var(--primary); display: flex; align-items: center; justify-content: center; }
.spinning { animation: spin 3s linear infinite; }
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

.bg-blob { position: absolute; border-radius: 50%; filter: blur(60px); opacity: 0.6; z-index: 0; animation: float 10s infinite ease-in-out; }
.blob-1 { width: 300px; height: 300px; background: #ffe4e1; top: -50px; left: -50px; }
.blob-2 { width: 400px; height: 400px; background: #e6e6fa; bottom: -100px; right: -100px; animation-delay: -5s; }
@keyframes float { 0%, 100% { transform: translate(0, 0); } 50% { transform: translate(30px, 50px); } }

.slide-section { z-index: 10; width: 100%; height: 100%; position: absolute; top: 0; left: 0; background-size: cover; background-position: center; background-repeat: no-repeat; transition: background-image 0.5s ease; }
.slide-section.cover .overlay, .slide-section.letter .overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.4); backdrop-filter: blur(5px); z-index: 1; }
.slide-section.letter .overlay { background: rgba(255, 255, 255, 0.3); backdrop-filter: blur(3px); }
.slide-section.cover { display: flex; flex-direction: column; align-items: center; justify-content: center; }
.slide-section.cover .content-box { z-index: 2; max-width: 85%; padding: 20px; width: 800px; text-align: center; }
.cover h1, .cover .typewriter-text { font-size: clamp(1.5rem, 5vw, 2.5rem); color: white; margin-bottom: 20px; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); white-space: pre-wrap; line-height: 1.4; }
.typewriter-text .cursor { display: inline-block; background-color: white; width: 3px; height: 1.1em; vertical-align: middle; margin-left: 5px; animation: blink-cursor 0.8s infinite step-end; }
@keyframes blink-cursor { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
.hint { margin-top: 50px; font-size: 0.9rem; color: #f0f0f0; animation: pulse 2s infinite; text-align: center; }

.slide-section.content { display: flex; align-items: center; justify-content: center; padding: 20px; }
.slide-section.content .background-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.3); backdrop-filter: blur(8px); z-index: 1; }
.content-main { display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 40px; max-width: 1100px; width: 90%; z-index: 2; position: relative; background: rgba(255, 255, 255, 0.6); border-radius: 20px; padding: 15px 40px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); backdrop-filter: blur(5px); overflow: visible !important; }

.polaroid { flex-shrink: 0; width: 450px; background: white; padding: 15px 15px 60px 15px; box-shadow: 0 25px 50px rgba(0,0,0,0.2), 0 0 0 2px rgba(0,0,0,0.02) inset; margin-left: -140px; margin-top: -60px; margin-bottom: -60px; transform: rotate(-5deg); z-index: 10; border-radius: 4px; transition: transform 0.3s; }
.polaroid img { width: 100%; height: auto; object-fit: cover; border-radius: 2px; display: block; }
.polaroid:hover { transform: rotate(0deg) scale(1.05); box-shadow: 0 30px 70px rgba(0,0,0,0.3); z-index: 20; }

.text-area { flex-grow: 1; text-align: left; min-width: 0; padding-left: 10px; }
.slide-title { font-size: 1.5rem; color: var(--primary); margin-bottom: 10px; font-weight: bold; text-shadow: 1px 1px 2px rgba(0,0,0,0.05); }
.date-tag { background: var(--primary); color: white; padding: 6px 16px; border-radius: 20px; font-size: 0.9rem; font-weight: bold; display: inline-block; margin-bottom: 20px; box-shadow: 0 4px 10px rgba(228, 177, 171, 0.4); }
@keyframes soft-float-up { 0% { opacity: 0; transform: translateY(15px); } 100% { opacity: 1; transform: translateY(0); } }
.text-area p.sentence-item { margin: 8px 0; color: var(--text-main); line-height: 1.8; white-space: pre-line; font-family: 'ZCOOL KuaiLe', cursive, "Microsoft YaHei", sans-serif; font-size: 1.4rem; animation: soft-float-up 4.0s cubic-bezier(0.22, 1, 0.36, 1) forwards; }

.slide-section.letter { display: flex; align-items: center; justify-content: center; }
.letter-paper { background: rgba(255, 255, 255, 0.95); padding: 40px; border-radius: 10px; box-shadow: 0 10px 40px rgba(0,0,0,0.15); text-align: left; width: 85%; max-width: 600px; z-index: 2; position: relative; max-height: 80vh; overflow-y: auto; }
.letter-paper h2 { color: var(--primary); border-bottom: 1px solid #e0e0e0; padding-bottom: 15px; margin-bottom: 20px; font-weight: normal; }
.letter-paper p { font-family: "Microsoft YaHei", sans-serif; font-size: 1.1rem; line-height: 2; color: var(--text-main); white-space: pre-line; }
.gift-btn { margin-top: 30px; width: 100%; padding: 15px; background: var(--primary); color: white; border: none; border-radius: 8px; font-size: 1rem; font-weight: bold; cursor: pointer; transition: 0.3s; }
.gift-btn:hover { opacity: 0.9; transform: translateY(-2px); box-shadow: 0 5px 15px rgba(228, 177, 171, 0.4); }

.progress-bar { position: absolute; bottom: 0; left: 0; width: 100%; height: 4px; background: rgba(255,255,255,0.3); z-index: 20; }
.progress-inner { height: 100%; background: var(--primary); transition: width 0.5s ease; }

.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease, transform 0.5s ease; }
.fade-enter-from { opacity: 0; transform: translateY(20px); }
.fade-leave-to { opacity: 0; transform: translateY(-20px); }
@keyframes pulse { 0% { opacity: 0.5; } 50% { opacity: 1; } 100% { opacity: 0.5; } }

.photo-collage { flex-shrink: 0; width: 550px; height: 500px; position: relative; margin-left: -120px; margin-top: -50px; margin-bottom: -50px; z-index: 10; }
.polaroid-mini { position: absolute; background: white; padding: 8px 8px 35px 8px; box-shadow: 0 10px 25px rgba(0,0,0,0.15), 0 0 0 1px rgba(0,0,0,0.02) inset; border-radius: 4px; transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1); }
.polaroid-mini img { width: 100%; height: auto; object-fit: cover; border-radius: 2px; display: block; }
.polaroid-mini:hover { z-index: 100 !important; transform: scale(1.2) rotate(0deg) !important; box-shadow: 0 30px 60px rgba(0,0,0,0.3); }
.collage-1 { width: 210px; top: 10px; left: 10px; transform: rotate(-6deg); z-index: 11; }
.collage-2 { width: 200px; top: 30px; right: 10px; transform: rotate(5deg); z-index: 12; }
.collage-3 { width: 210px; bottom: 20px; left: 20px; transform: rotate(3deg); z-index: 13; }
.collage-4 { width: 200px; bottom: 10px; right: 10px; transform: rotate(-4deg); z-index: 14; }

.loading-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: #fdfcf8; z-index: 9999; display: flex; justify-content: center; align-items: center; }
.loading-content { text-align: center; color: var(--primary); }
.loading-content p { margin-top: 20px; font-size: 1.2rem; letter-spacing: 2px; font-family: "Microsoft YaHei", sans-serif; }
.spinner { width: 50px; height: 50px; border: 3px solid rgba(228, 177, 171, 0.3); border-radius: 50%; border-top-color: var(--primary); animation: spin-loading 1s ease-in-out infinite; margin: 0 auto; }
@keyframes spin-loading { to { transform: rotate(360deg); } }

/* --- Gallery 画廊模式样式 --- */
.gallery-container {
  flex-direction: column !important;
  align-items: center;
  height: 80vh; 
  overflow-y: auto !important;
  padding-bottom: 100px !important;
}

.gallery-title {
  font-size: 1.8rem;
  color: var(--primary);
  margin-bottom: 15px;
  text-shadow: 2px 2px 0px white;
  flex-shrink: 0;
  text-align: center;
}

.gallery-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  width: 100%;
  padding: 10px;
}

/* 单个卡片容器 */
.gallery-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 200px; 
  opacity: 0;
  transform: translateY(20px);
  animation: pop-in 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}

@keyframes pop-in {
  to { opacity: 1; transform: translateY(0); }
}

/* 迷你拍立得效果 */
.polaroid-mini-card {
  background: white;
  padding: 8px 8px 25px 8px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
  transform: rotate(-2deg);
  transition: transform 0.3s;
  width: 100%;
}

.gallery-item:nth-child(even) .polaroid-mini-card {
  transform: rotate(2deg); 
}

.polaroid-mini-card img {
  width: 100%;
  height: 150px; 
  object-fit: cover;
  border-radius: 2px;
}

.gallery-text {
  /* background: rgba(255,255,255,0.8);  <-- 保持移除状态 */
  margin-top: 12px;
  font-family: 'ZCOOL KuaiLe', cursive;
  color: var(--text-main);
  font-size: 1.25rem; /* <-- 字体调大 */
  text-align: center;
  font-weight: bold; 
}

/* 手机端画廊适配 */
.mode-mobile .gallery-item {
  width: 100% !important; 
  flex-direction: row; 
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
}

.mode-mobile .polaroid-mini-card {
  width: 120px; 
  flex-shrink: 0;
  transform: rotate(-3deg) !important;
}

.mode-mobile .gallery-text {
  flex-grow: 1;
  text-align: left;
  font-size: 1.15rem; /* <-- 手机端也调大 */
  background: none;
  box-shadow: none;
  padding: 0;
  margin: 0;
}

/* --- 新增样式：第6页照片控制 & Gallery文本区 --- */

/* 第6页照片的弹出效果 */
.pop-in-effect {
  opacity: 1;
  transform: scale(1) rotate(var(--rotation, 0deg)); 
  transition: all 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

/* 第6页照片的隐藏状态 */
.delayed-show {
  opacity: 0;
  transform: scale(0.5) !important; 
  pointer-events: none;
}

/* Gallery 文字区域容器 - 修改处 */
.gallery-text-area {
  width: 90%;
  max-width: 800px;
  margin-top: 30px;
  margin-bottom: 50px; 
  text-align: center;
  /* background: rgba(255,255,255,0.6); <-- 已彻底删除背景色 */
  background: transparent; 
  padding: 10px; /* 减少内边距 */
  /* border-radius: 15px; <-- 已删除圆角 */
  flex-shrink: 0; /* 防止压缩 */
}

/* Gallery 单句文字样式 - 修改处 */
.gallery-sentence {
  text-align: center !important;
  margin-bottom: 10px;
  font-size: 1.3rem !important; /* <-- 字体再次调大 (原 1.1rem) */
  color: var(--text-main);
  font-family: 'ZCOOL KuaiLe', cursive, sans-serif;
  animation: soft-float-up 4.0s cubic-bezier(0.22, 1, 0.36, 1) forwards;
  line-height: 1.8;
  font-weight: bold; /* 稍微加粗以防背景干扰 */
}
</style>