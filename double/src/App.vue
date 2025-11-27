<script setup lang="ts">
import { ref, computed, watch, onMounted, nextTick } from 'vue' 
// 👇 请确保路径正确
import FireworksPage from './compoents/FireworksPage.vue'
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
    printText: '2025.9.27 - 2025.11.28\n这是一段属于我们，短而有幸的时光。', 
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
    title: '按下快门记录的我',
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
      { img: '/photos/ty/18.jpg', text: '帮我也把把脉' },
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
    type: 'gallery',
    title: '按下快门记录的你',
    date: '2025.10',
    backgroundType: 'image',
    backgroundImage: '/photos/s.jpg', 
    gallery: [
      { img: '/photos/ty/41.jpg', text: '家教教小孩🧒' },
      { img: '/photos/ty/39.jpg', text: '人挤人的音乐节（担心死我了）' },
      { img: '/photos/ty/34.jpg', text: '这沙发，100%好货' },
      { img: '/photos/ty/32.jpg', text: '爬坡爬坡！' },
      { img: '/photos/ty/29.jpg', text: '便利店里指点江山' },
      { img: '/photos/ty/27.jpg', text: '找到共享单车拉✌️ '},
      { img: '/photos/ty/25.jpg', text: '投送咖啡粉，爱你' },
      { img: '/photos/ty/23.jpg', text: '看出来了，拉花是Ty' },
      { img: '/photos/ty/21.jpg', text: '这个架子在防舍友[捂嘴笑]' },
      { img: '/photos/ty/19.jpg', text: '天气好好' },
      { img: '/photos/ty/18.jpg', text: '帮我也把把脉' },
      { img: '/photos/ty/15.jpg', text: '健身成果展示ing' },
      { img: '/photos/ty/14.jpg', text: '这个拼豆（早该想到的）' },
      { img: '/photos/ty/13.jpg', text: '热心舍友Ty（不要学打篮球了😨）' },
      { img: '/photos/ty/11.jpg', text: '拼豆作品集' },
      { img: '/photos/ty/6.jpg', text: '可爱的勺子碎了😭' },
      { img: '/photos/ty/66.jpg', text: '跟我说了这件事之后就下单啦。宠你' },
      { img: '/photos/ty/5.jpg', text: 'Leon摄影有限公司出品' },
      { img: '/photos/ty/3.jpg', text: '最讨厌的化学' },
      { img: '/photos/ty/1.jpg', text: '现在我才知道这个叫乌萨琪' }
    ],
    text: '这里有让我担心的音乐节人海、有让人头大的化学公式、还有你用拼豆一点点拼出来的可爱世界（送我的拼得好好！）。从便利店的指点江山到健身房的汗水，从那罐漂洋过海的咖啡粉到摔碎的小勺子。如果不看这些照片，我大概无法想象你在我看不到的地方，把生活过得这么热气腾腾。虽然隔着时差，但我好像能透过屏幕，摸到你生活的脉搏。你负责在镜头那边闪闪发光，我负责在镜头这边，把你的每一个瞬间都好好收藏。',
  },

  // ✨✨✨ 新增：过渡页 1 ✨✨✨
  {
    type: 'transition',
    text: '还有就是......'
  },

  // ✨✨✨ 新增：过渡页 2 ✨✨✨
  {
    type: 'transition',
    text: '关于吃吃吃吃😋：我们开启了疯狂的“云投喂”模式。\n在这几千公里的距离里，\n分享一日三餐，成了我们最独特的拥抱方式。'
  },

  // ✨✨✨ Ty 的美食篇 ✨✨✨
  {
    type: 'gallery',
    title: 'Ty的“云投喂”日记',
    date: '2025.10 - 2025.11',
    backgroundType: 'image',
    backgroundImage: '/photos/ty/77.png', 
    gallery: [
      { img: '/photos/ty/ty吃/55.jpg', text: '红豆面包🫘' },
      { img: '/photos/ty/ty吃/66.jpg', text: '恰巴塔' },
      { img: '/photos/ty/ty吃/7.jpg', text: '低卡牛肉干' },
      { img: '/photos/ty/ty吃/37.jpg', text: '葡萄雪糕（满满蛋白质）' },
      { img: '/photos/ty/ty吃/778.jpg', text: '杂粮饭+鸡胸肉' },
      { img: '/photos/ty/ty吃/46.jpg', text: '美味的米面包' },
      { img: '/photos/ty/ty吃/14.jpg', text: '福袋（麻酱好好吃🤤）' },
      { img: '/photos/ty/ty吃/43.png', text: '芋泥饼，吃美了' },
      { img: '/photos/ty/ty吃/21.png', text: '蛋糕投喂，评鉴ing...' },
      { img: '/photos/ty/ty吃/54.jpg', text: '鱼蛋，我吃' },
      { img: '/photos/ty/ty吃/85.jpg', text: '这个练后餐蛋白质达标！' },
      { img: '/photos/ty/ty吃/86.jpg', text: '培根蔬萃双层牛堡！' },
      { img: '/photos/ty/ty吃/19.jpg', text: '健康！' },
      { img: '/photos/ty/ty吃/73.jpg', text: '替我品尝的三明治🥹' },
      { img: '/photos/ty/ty吃/17.jpg', text: '0卡😁' },
      { img: '/photos/ty/ty吃/16.jpg', text: '减脂吃得好平淡（已经很瘦拉！）' },
      { img: '/photos/ty/ty吃/18.jpg', text: '又是Sandwich！' },
      { img: '/photos/ty/ty吃/13.jpg', text: '突然感觉舍友好好，嘻嘻' },
      { img: '/photos/ty/ty吃/3.jpg', text: '健身餐里最馋我的一次' },
      { img: '/photos/ty/ty吃/4.jpg', text: '卡朋的这个轻食好好吃' },
      { img: '/photos/ty/ty吃/5.jpg', text: '珠海玩完回来被老哥”强“拉着去吃的宵夜' },
      { img: '/photos/ty/ty吃/6.jpg', text: '看着就健康好吃的粉' },
      { img: '/photos/ty/ty吃/35.jpg', text: '巨无敌无敌好吃的鱿鱼面！' },
      { img: '/photos/ty/ty吃/77.jpg', text: '华夫饼🧇' },
      { img: '/photos/ty/ty吃/87.jpg', text: '🍽️' },
      { img: '/photos/ty/ty吃/89.jpg', text: '有一个面包好像不是很好吃哦' },
      { img: '/photos/ty/ty吃/2.jpg', text: '妈妈做得猪蹄😋' },
      { img: '/photos/ty/ty吃/1.jpg', text: '自烤恰巴塔' }
    ],
    text: '',
  },

  // ✨✨✨ Leon 的美食篇 ✨✨✨
  {
    type: 'gallery',
    title: '“一人食”碎碎念',
    date: '2025.10 - 2025.11',
    backgroundType: 'image',
    backgroundImage: '/photos/ty/leon吃/7.jpg', 
    gallery: [
      { img: '/photos/ty/leon吃/1.jpg', text: '牛肉蛋饭，溏心蛋😋' },
      { img: '/photos/ty/leon吃/2.jpg', text: '菜椒牛肉' },
      { img: '/photos/ty/leon吃/3.jpg', text: '生日陪我吃的麦当劳🥹' },
      { img: '/photos/ty/leon吃/4.jpg', text: '烤鱼+npc' },
      { img: '/photos/ty/leon吃/5.jpg', text: '番茄猪排意面' },
      { img: '/photos/ty/leon吃/6.jpg', text: '芝士烤土豆' },
      { img: '/photos/ty/leon吃/8.jpg', text: '剪发哥家吃烤肉' },
      { img: '/photos/ty/leon吃/9.jpg', text: '简简单单加个蛋' },
      { img: '/photos/ty/leon吃/10.jpg', text: '烤鸡翅😋' },
      { img: '/photos/ty/leon吃/11.jpg', text: '《午餐》' },
      { img: '/photos/ty/leon吃/12.jpg', text: '一整个的土豆🥔' },
      { img: '/photos/ty/leon吃/13.jpg', text: '质（脂）量最高的一餐' },
      { img: '/photos/ty/leon吃/14.jpg', text: '柴鸡胸肉+好吃的甜包' },
      { img: '/photos/ty/leon吃/15.jpg', text: '炒鸡肉 + 🍣' },
      { img: '/photos/ty/leon吃/17.jpg', text: '还是熟悉的猪排+NPC' }, 
      { img: '/photos/ty/leon吃/18.jpg', text: '天才双祺' },
      { img: '/photos/ty/leon吃/19.jpg', text: '。。。' },
      { img: '/photos/ty/leon吃/20.jpg', text: '。。。' },
      { img: '/photos/ty/leon吃/21.jpg', text: '菜椒炒肉丝，好好吃！' },
      { img: '/photos/ty/leon吃/22.jpg', text: '。😁。好吃。' },
      { img: '/photos/ty/leon吃/23.jpg', text: '今天的鸡排还不错哦' },
      { img: '/photos/ty/leon吃/24.jpg', text: '鸡肉（实际蔬菜）卷' },
      { img: '/photos/ty/leon吃/25.jpg', text: '排骨😋' }
    ],
    text: '有人说，分享欲是爱情的最高级。所以哪怕是一碗简单的面、一个路边的面包，我都想拍给你看。因为在这些琐碎的照片背后，藏着我没说出口的话：正在吃饭的这一刻，我又想你了。 谢谢你愿意接住我所有的碎碎念，让我在慕尼黑的每一餐都不觉得孤单。虽然我们无法同桌而食，但每一张照片发出的瞬间，我们都在陪对方度过一日三餐。是这些食物，连接了我们平行的生活，也连接了彼此想念的心。',
  },

    {
    type: 'transition',
    text: '一直到24号...'
  },

{
    type: 'content', 
    title: '跨越三年的“久别重逢”',
    image: '/photos/视频/1024.png', 
    date: '2025.10.24',
    // 👇 文案重点：告别过去，开启未来
    text: '从2022年10月到2025年10月。屏幕亮起的那一秒，是对这三年空白的正式告别，也是我们未来的第一行序言。\n\n看着镜头里那个熟悉又爱笑的你，我终于确信：过去的遗憾翻篇了，崭新的我们，开始了。',
    backgroundType: 'image', 
    backgroundImage: '/photos/视频/10242.JPG', 
  },

  // {
  //   type: 'transition',
  //   text: '然后，我们开始了......'
  // },

  // ✨✨✨ 新增：FaceTime 视频分批展示页 ✨✨✨
  {
    type: 'batch-gallery', 
    // 👇 标题修改：强调连续性
    title: '我们的FaceTime',
    date: '2025.10.24 - 至今',
    backgroundType: 'image',
    backgroundImage: '/photos/1024.jpg', 
    gallery: [
      { img: '/photos/视频/1.5.png' }, { img: '/photos/视频/7.PNG' },
      { img: '/photos/视频/8.PNG' }, { img: '/photos/视频/9.PNG' },
      
      { img: '/photos/视频/13.png' }, { img: '/photos/视频/12.png' },
      { img: '/photos/视频/2.png' }, { img: '/photos/视频/1.png' },
      
      { img: '/photos/视频/10.PNG' }, { img: '/photos/视频/14.png' },
      { img: '/photos/视频/15.PNG' }, { img: '/photos/视频/5.jpeg' }
    ],
    // 👇 正文修改：解释“停不下来”的含义，呼应前面的三年分别
    text: '从10月24日至今天，一天未断。原来所谓的“忙得没时间”在思念面前都是伪命题。\n\n因为想你这件事无法暂停，所以无论多忙，见你都是我必须完成的日常。这份不断的记录，是我对你“永远有空”的承诺。' 
  },

  {
    type: 'gallery',
    title: '视频让我们更近地接触对方，相互袒露',
    date: '2025.10 - 2025.11',
    backgroundType: 'image',
    backgroundImage: '/photos/1026/13.jpg', 
    gallery: [
      { img: '/photos/1026/1029.png' , text: '10.29 - “爱情比前途更难得”.“19岁无能为力的事太多，但对的人终将相遇”' },
      { img: '/photos/1026/1112.png', text: ' “11.12 - 我爱你。” “我也爱你。” ' },
      { img: '/photos/1026/11151.png', text: '11.15 - 隔着屏幕流泪的那一刻，以为这是脆弱'  },
      { img: '/photos/1026/11152.png', text: '11.15 - 互相的长信告诉我们，这不是脆弱，这是我们心贴得最近的时候。' },
    ],
    text: '在15号那晚的眼泪擦干后，你教会了我关于爱最重要的一课。\n\n我不自信地问“是不是不合适”，你却坚定地告诉我：“情绪价值是可以被替代的，但一个有性格、有主见、鲜活的廖双祺，是无法替代的。”\n\n这句话，是我在慕尼黑收到过最好的礼物。它让我明白，爱不是扮演完美，而是两个真实的灵魂，相互认领。谢谢你，看见并拥抱了那个独一无二的我。从这一晚开始，我不再只想做那个只会逗你笑的Leon，因为我知道，你愿意接住我的泪水和抱怨。也就是从这一刻起，我觉得我们不再是“异地恋”，而是“在一起”。',
  },

  {
    type: 'transition',
    text: '与此同时...'
  },

{
    type: 'gallery',
    title: '跨越千里的书信',
    date: '2025.10 - 2025.11',
    backgroundType: 'image',
    backgroundImage: '/photos/12.jpg', 
    gallery: [
      { img: '/photos/信/1.png' , text: '10月20号，寄出的第一封信《慕尼黑落叶的秋天》' },
      { img: '/photos/信/2.jpeg', text: '"这个弹吉他的黄发美女最萌"' },
      { img: '/photos/信/3.jpeg', text: '"命运是那5/6"'  },
      { img: '/photos/信/4.jpeg', text: '出发' },
      { img: '/photos/信/5.jpg', text: '到达'  },
      { img: '/photos/信/6.png', text: '10月末，以为不会寄出的信' },
      { img: '/photos/信/7.jpeg' },
      { img: '/photos/信/8.png', text: '"我发现..."' },
      { img: '/photos/信/9.jpeg', text: '对她的感情，是爱。'  },
      { img: '/photos/信/11.jpg' , text: 'ty寄的第一封！'},
      { img: '/photos/信/13.JPG', text: '11月15日收，开心' },
      { img: '/photos/信/14.jpeg', text: '拼得好好！好可爱' },
      { img: '/photos/信/15.jpeg', text: '爱你'  }
    ],
    text: '收到8000公里外寄来的信，真是一种神奇的感觉。8000公里的距离，光缆传输视频只需要0.1秒，但信件抵达却需要1个月。我们将日常交给了FaceTime，将内心珍藏的话交给了书信。\n\n这里有慕尼黑的落叶，有你亲手拼凑的可爱，还有我们在深夜里一笔一划写下的真心。这些信，可能就是我们在这个电子时代里，在这个异地恋的距离中，最笨拙也最真诚的浪漫证明。',
  },

    {
    type: 'transition',
    text: '就这样一步一步...又到了今天11月28号，你的生日🎂'
  },

{
    type: 'letter', 
    image: "/photos/信.jpg",
    title: '致我最爱的女孩 (From Leon in Munich)',
    // 👇 扩展后的长信内容
    text: `亲爱的Ty，生日快乐。

            从14岁那年的初遇，到19岁这年的重逢。中间走散的那两年，大概是命运为了让我们学会如何更好地去爱，才特意留出的空白。

            回看这短短两个月，命运的算法真的太奇妙了。
            如果那天我没有买那个面包机，如果那晚我没有发高烧，如果10月24日那个视频通话没有接通……我们依然是两条平行的线。但幸运的是，在无数个可能错过的分岔路口，我们都坚定地走向了对方。

            谢谢你，Ty。
            谢谢你在我试图用“别回我”来逃避时，用一句“晚点回你”接住了我；
            谢谢你在我面对慕尼黑的满地狼藉、觉得自己糟糕透顶时，告诉我“真实的Leon无法替代”；
            谢谢你愿意陪我吃每一顿“云晚餐”，谢谢你把生活的琐碎拼成可爱的拼豆，寄到我身边。

            在这个光缆传输只需要0.1秒、书信抵达却需要30天的世界里，我想要给你最老派也最长情的爱。
            虽然此刻慕尼黑是冬天，但只要想到你，我这里就是永远的晴天。

            19岁这年，能把你“切”回我的生命里，是我最大的幸运。
            愿今后的每一个生日，无论相隔多远，我都能陪在你身边。
            做你自己，快乐自由。

          爱你。`,
    buttonText: '点我，点我🐶'
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

// 修改：允许 Content 和 Gallery 都能处理分句逻辑
const currentSlideSentences = computed(() => {
  const slide = currentSlide.value
  if ((slide.type !== 'content' && slide.type !== 'gallery' && slide.type !== 'batch-gallery') || !slide.text) return []
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

  // 🔴 新增 2. Batch Gallery 模式逻辑
  if (slide.type === 'batch-gallery') {
    const batchSize = 4
    const batchCount = Math.ceil((slide.gallery?.length || 0) / batchSize)
    // 逻辑：总步数 = 1(空白起始) + 批次数量 + 文字数量
    // 文字显示的步数 = 当前步数 - (1 + 批次数量)
    // 举例：若有3批图。Step 1:空白; Step 2:第1批; Step 3:第2批; Step 4:第3批; Step 5:第1句文字
    const textStep = Math.max(0, contentStep.value - (1 + batchCount))
    return currentSlideSentences.value.slice(0, textStep)
  }

  // 3. 第6页(Index 5)特殊逻辑：步数减去图片数量
  if (currentIndex.value === 5 && slide.images) {
    const imageCount = slide.images.length
    const textStep = Math.max(0, contentStep.value - imageCount)
    return currentSlideSentences.value.slice(0, textStep)
  }
  
  // 4. 普通 Content 逻辑
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

// ⭐⭐⭐ 新增：全屏辅助函数 ⭐⭐⭐
const triggerFullScreen = () => {
  const docEl = document.documentElement as any
  // 尝试调用各种浏览器的全屏API
  const requestMethod = docEl.requestFullscreen || docEl.webkitRequestFullscreen || docEl.mozRequestFullScreen || docEl.msRequestFullscreen;

  if (requestMethod) {
    // 兼容写法
    if (docEl.requestFullscreen) {
       docEl.requestFullscreen().catch((e: any) => console.log('Fullscreen blocked or not supported', e))
    } else if (docEl.webkitRequestFullscreen) {
       docEl.webkitRequestFullscreen() 
    }
  }
}

const nextSlide = (isFromButton: boolean | Event = false) => {
  if (isLoading.value || showDeviceSelector.value || isLocked.value) return
  if (showFireworksPage.value) return

  const isConfirmed = isFromButton === true
  if (currentSlide.value.type === 'letter' && !isConfirmed) {
    return
  }

  if (currentSlide.value.type === 'cover' && !isTypingFinished.value) {
    return 
  }

  if (audioRef.value && audioRef.value.paused && !isMusicPlaying.value && currentIndex.value < slides.length - 1) {
    audioRef.value.play()
      .then(() => { isMusicPlaying.value = true })
      .catch((e) => console.log('等待交互播放', e))
  }

  const isContent = currentSlide.value.type === 'content'
  const isGallery = currentSlide.value.type === 'gallery'
  const isBatchGallery = currentSlide.value.type === 'batch-gallery'

  if (isContent || isGallery || isBatchGallery) {
    let totalSteps = 0

    if (isBatchGallery) {
       const batchSize = 4
       const batchCount = Math.ceil((currentSlide.value.gallery?.length || 0) / batchSize)
       totalSteps = 1 + batchCount + currentSlideSentences.value.length
    } else if (isGallery) {
      totalSteps = (currentSlide.value.gallery?.length || 0) + currentSlideSentences.value.length
    } else {
      if (currentIndex.value === 5 && currentSlide.value.images) {
        totalSteps = currentSlide.value.images.length + currentSlideSentences.value.length
      } else {
        totalSteps = currentSlideSentences.value.length
      }
    }

    if (contentStep.value < totalSteps) {
      contentStep.value++
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
      return 
    }
  }

  if (currentIndex.value === slides.length - 1) {
    if (audioRef.value) {
      audioRef.value.pause()
      audioRef.value.src = '/music/小幸运.mp3'
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
      nextTick(() => {
        const container = document.querySelector('.mode-mobile .content-main')
        if (container) container.scrollTop = 0
      })
    }, 500) 
  }
}

// --- 设备选择 ---
const selectDevice = (mode: string) => {
  // ⭐⭐⭐ 修改点：iPad或手机模式点击时，尝试触发全屏 ⭐⭐⭐
  if (mode === 'tablet' || mode === 'mobile') {
    triggerFullScreen()
  }
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
              <Transition name="gallery-pop" v-for="(item, index) in currentSlide.gallery" :key="index">
                <div 
                  class="gallery-item"
                  v-show="index < contentStep"
                >
                  <div class="polaroid-mini-card">
                    <img :src="item.img" loading="lazy" />
                  </div>
                  <p class="gallery-text">{{ item.text }}</p>
                </div>
              </Transition>
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
          v-else-if="currentSlide.type === 'batch-gallery'" 
          class="slide-section content gallery-mode" 
          :key="currentIndex"
          :style="{ backgroundImage: `url(${currentSlide.backgroundImage})` }"
        >
          <div class="background-overlay"></div>
          
          <div class="content-main gallery-container">
            <h3 class="gallery-title">{{ currentSlide.title }}</h3>
            
            <div class="gallery-grid batch-grid">
              <Transition name="gallery-pop" v-for="(item, index) in currentSlide.gallery" :key="index">
                <div 
                  class="gallery-item"
                  v-show="index < (contentStep - 1) * 4" 
                >
                  <div class="polaroid-mini-card">
                    <img :src="item.img" loading="lazy" />
                  </div>
                </div>
              </Transition>
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
          v-else-if="currentSlide.type === 'transition'" 
          class="slide-section transition-slide" 
          :key="currentIndex"
        >
          <div class="transition-content">
            <p>{{ currentSlide.text }}</p>
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
            <button class="gift-btn" @click.stop="nextSlide(true)">{{ currentSlide.buttonText }}</button>
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

/* --- 设备适配逻辑 (iPad 11寸优化版) --- */

.app-container.mode-tablet .content-main {
  transform: scale(0.85);
  width: 85%;
  max-width: 900px;
}

@media screen and (orientation: portrait) {
  .app-container.mode-tablet .content-main {
    flex-direction: column !important;
    align-items: center !important;
    justify-content: flex-start !important;
    transform: none !important;
    width: 85% !important;
    height: auto !important;
    max-height: 85vh;
    overflow-y: auto !important;
    padding: 30px 20px !important;
    gap: 25px;
  }
  .app-container.mode-tablet .polaroid {
    margin: 0 !important;
    width: 280px !important;
    transform: rotate(-2deg) !important;
    flex-shrink: 0;
  }
  .app-container.mode-tablet .photo-collage {
    margin: 0 !important;
    transform: scale(0.75) !important;
    flex-shrink: 0;
  }
  .app-container.mode-tablet .text-area {
    width: 100% !important;
    text-align: center !important;
    padding-left: 0 !important;
  }
  .app-container.mode-tablet .slide-title {
    text-align: center !important;
    font-size: 1.4rem !important;
  }
  .app-container.mode-tablet .text-area p.sentence-item {
    text-align: center !important;
    font-size: 1.15rem !important;
  }
  .app-container.mode-tablet .gallery-grid {
    gap: 20px;
  }
}

/* Mobile */
.app-container.mode-mobile .content-main {
  display: flex !important;
  flex-direction: column !important;
  align-items: center !important;
  justify-content: flex-start !important;
  width: 90vw !important; 
  max-width: 430px !important;
  height: auto;
  max-height: 85vh;
  padding: 50px 20px 120px 20px !important;
  margin: 20px auto !important; 
  left: auto !important;
  right: auto !important;
  transform: none !important;
  gap: 25px;
  overflow-y: auto !important; 
  overflow-x: hidden !important;
  -webkit-overflow-scrolling: touch;
  box-sizing: border-box !important;
}
.app-container.mode-mobile .polaroid {
  margin: 0 !important; 
  width: 240px !important; 
  padding: 12px 12px 40px 12px !important;
  align-self: center !important;
  transform: rotate(-2deg) !important;
  position: relative !important;
  left: auto !important;
  top: auto !important;
  flex-shrink: 0 !important;
}
.app-container.mode-mobile .photo-collage {
  margin: 0 !important;
  width: 280px !important;
  height: 250px !important;
  align-self: center !important;
  transform-origin: center center !important;
  transform: scale(1) !important;
  position: relative !important;
  left: auto !important;
  top: auto !important;
  flex-shrink: 0 !important;
}
.app-container.mode-mobile .collage-1, .app-container.mode-mobile .collage-2, .app-container.mode-mobile .collage-3, .app-container.mode-mobile .collage-4 {
  width: 130px !important; 
}
.app-container.mode-mobile .collage-1 { left: 0px !important; top: 0 !important; }
.app-container.mode-mobile .collage-2 { right: 0px !important; top: 15px !important; }
.app-container.mode-mobile .collage-3 { left: 10px !important; bottom: 15px !important; }
.app-container.mode-mobile .collage-4 { right: 10px !important; bottom: 0 !important; }
.app-container.mode-mobile .text-area {
  width: 100% !important;
  text-align: center !important;
  padding: 0 !important; 
  margin: 0 !important;
}
.app-container.mode-mobile .slide-title {
  font-size: 1.4rem !important;
  margin: 10px 0 15px 0 !important;
  text-align: center !important;
  display: block !important;
  width: 100% !important;
}
.app-container.mode-mobile .date-tag {
  font-size: 0.9rem !important;
  padding: 5px 16px !important;
  margin: 0 auto 20px auto !important;
  display: inline-block !important;
}
.app-container.mode-mobile .text-area p.sentence-item {
  font-size: 1.15rem !important;
  line-height: 1.8 !important;
  margin: 8px 0 !important;
  text-align: center !important;
}
.mode-mobile .gallery-item {
  width: 100% !important; 
  flex-direction: row; 
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
  background: rgba(255,255,255,0.4);
  padding: 10px;
  border-radius: 12px;
}
.mode-mobile .polaroid-mini-card {
  width: 130px; 
  flex-shrink: 0;
  transform: rotate(-2deg) !important;
}
.mode-mobile .gallery-text {
  flex-grow: 1;
  text-align: left;
  font-size: 1.1rem; 
  background: none;
  box-shadow: none;
  padding: 0;
  margin: 0;
  line-height: 1.4;
}

/* --- 🔴 修改后：Batch Gallery 专用 (强制一行4个) --- */

/* 1. 容器设置：允许换行，居中 */
.mode-mobile .gallery-grid.batch-grid {
  display: flex !important;
  flex-direction: row !important;
  flex-wrap: wrap !important;
  justify-content: center !important; /* 居中对齐 */
  align-content: flex-start !important;
  gap: 6px !important; /* 间距调小，因为照片变多了 */
  padding: 0 5px !important; /* 左右留一点点边距 */
}

/* 2. 单个照片项：强制宽度为 23% (4个就是 92%，加上间距刚好一行) */
.mode-mobile .batch-grid .gallery-item {
  width: 22% !important; /* 🔴 关键修改：从 45% 改为 22%，实现一行4个 */
  display: flex !important;
  flex-direction: column !important;
  padding: 0 !important; /* 去掉外层内边距 */
  background: none !important;
  margin-bottom: 5px !important;
  /* ✨✨✨ 这里删除了原本写死的 animation: pop-in ... 以免冲突 */
}

/* 3. 拍立得卡片内部微调：让图片撑满，白边减小 */
.mode-mobile .batch-grid .polaroid-mini-card {
  width: 100% !important; 
  padding: 3px 3px 12px 3px !important; /* 🔴 白边改薄，留出更多空间给照片 */
  transform: rotate(0deg) !important; /* 整齐排列，不歪斜 */
  box-shadow: 0 2px 5px rgba(0,0,0,0.1) !important;
}

/* 4. 图片本身高度自适应 */
.mode-mobile .batch-grid .polaroid-mini-card img {
  height: 65px !important; /* 🔴 限制高度，保证正方形或整齐的比例 */
  object-fit: cover !important;
}

/* 平板/电脑端也同步调整一下，保证电脑上也是一行4个或更多，不会太松散 */
.batch-grid .gallery-item {
  width: 180px; /* 电脑端保持原状或微调即可 */
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
  /* ✨✨✨ 修改点：移除了原本写死的 opacity:0 和 animation: pop-in ... */
  /* 让它默认可见，只有在 Transition 触发时才跑动画 */
}

/* ✨✨✨ 新增：Transition 动画类 ✨✨✨ */
/* 当 v-show 变为 true 时，Vue 会自动添加这个类 */
.gallery-pop-enter-active {
  animation: pop-in 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.gallery-pop-leave-active {
  transition: opacity 0.3s;
}
.gallery-pop-enter-from, .gallery-pop-leave-to {
  opacity: 0;
  transform: translateY(30px);
}

@keyframes pop-in {
  0% { opacity: 0; transform: translateY(40px) scale(0.8); }
  100% { opacity: 1; transform: translateY(0) scale(1); }
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

/* ✨✨✨ 新增：过渡页样式 ✨✨✨ */
.slide-section.transition-slide {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

.transition-content {
  z-index: 5;
  text-align: center;
  max-width: 800px;
}

.transition-content p {
  font-size: 2rem;
  color: var(--primary);
  line-height: 1.8;
  font-weight: bold;
  white-space: pre-line;
  font-family: 'ZCOOL KuaiLe', cursive, "Microsoft YaHei", sans-serif;
  text-shadow: 2px 2px 4px rgba(255, 255, 255, 0.8);
  animation: soft-float-up 2s ease-out forwards;
}

/* 手机端过渡页适配 */
.mode-mobile .transition-content p {
  font-size: 1.5rem;
}
</style>