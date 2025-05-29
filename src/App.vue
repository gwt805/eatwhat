<template>
  <div class="random">
    <div class="logo"><img :src="eat" draggable="false" /></div>
    <div id="wrapper">
      <h1 style="color: #ff9733">{{ selectedFood }}</h1>
      <button class="button" @click="toggle" circle no-pulse>{{ buttonText }}</button>
    </div>
    <div ref="floatContainer" class="float-container"></div>
  </div>
</template>

<script setup lang="ts">
import eat from "./assets/eat.png"
import { ref, onMounted, onBeforeUnmount } from 'vue'

const selectedFood = ref('')
const buttonText = ref('开始')
let run = ref(false)
let timer: number | null = null
const floatContainer = ref<HTMLElement | null>(null)
const foods = ['鱼蛋', '火锅', '口味虾', '凉拌藕片', '黄山炖鸽', '拍黄瓜', '叫化鸡', '提拉米苏', '肠旺面', '日式烤肉', '炒年糕', '麦当劳', '杭州小笼包', '清炒时蔬', '烤土豆', '宁夏滩羊肉', '三明治', '文昌鸡', '烤生蚝', '扬州炒饭', '凉拌苦瓜', '凉拌皮蛋豆腐', '黄山双石', '青稞饼', '鸭脖', '杨枝甘露', '手抓饭', '煎饼', '拉面', '素馅饺子', '燕麦粥', '酸辣汤', '手抓饼', '烤冷面', '小笼包', '咖啡', '素炒三丝', '凉拌木耳', '梧州纸包鸡', '青海焜锅', '西餐', '张掖面皮', '雪梨银耳汤', '紫菜蛋花汤', '大盘鸡', '砂锅菜', '贵州辣子鸡', '螺蛳粉', '红豆沙', '冬瓜排骨汤', '黑森林蛋糕', '酸汤鱼', '清补凉', '凉皮', '胡辣汤', '东南亚菜', '烧麦', '西湖牛肉羹', '生煎包', '南宁卷筒粉', '九转大肠', '凉拌海蜇皮', '番茄牛腩汤', '玉米排骨汤', '烤包子', '番茄炒蛋', '芒果布丁', '瓦罐汤', '阳澄湖大闸蟹', '烤全羊', '灰豆子', '桂林米粉', '素包子', '糖葫芦', '澳门烧肉', '牛肉串', '丝娃娃', '鸡蛋灌饼', '榴莲千层', '稀饭', '玉米饼', '兰州酿皮', '风干牛肉', '豆汁焦圈', '珍珠奶茶', '葱油饼', '德州扒鸡', '香辣蟹', '香菇青菜', '台湾火锅', '鲜花饼', '手把肉', '冒菜', '汽锅鸡', '港式奶茶', '虾饺', '宁波汤圆', '蛋包饭', '荔浦芋扣肉', '凉拌土豆丝', '绿豆沙', '酸菜炖白肉', '萝卜排骨汤', '沙茶面', '锅盔', '烤肉', '麻辣子鸡', '烧鹅', '芝麻糊', '凉拌紫菜', '牛排', '剁椒鱼头', '韩式烤肉', '鸡爪', '三鲜豆皮', '海南粉', '葱油拌面', '宫保鸡丁', '涮羊肉', '柠檬茶', '意大利面', '印度咖喱饭', '赣南小炒鱼', '雪碧', '奶豆腐', '湘西腊肉', '肠粉', '东坡肉', '清蒸大闸蟹', '水煮牛肉', '椰子鸡', '臭桂鱼', '糖油粑粑', '清蒸多宝鱼', '披萨', '烤鱿鱼', '海南鸡饭', '素馅煎饼', '煎饼果子', '油炸金蝉', '泰式冬阴功汤', '广式早茶', '双皮奶', '青海老酸奶', '花溪牛肉粉', '羊蝎子', '奶茶', '傣味烤鱼', '大锅菜', '九江炒粉', '萝卜炖牛腩', '卤肉饭', '龙抄手', '热干面', '蛋挞', '鸡排', '叉烧', '煲仔饭', '串串香', '锅饼卷大葱', '热狗', '芬达', '烤羊腿', '炒河粉', '南昌拌粉', '云南火锅', '快餐', '岐山臊子面', '烤红薯', '甜醅子', '麻婆豆腐', '符离集烧鸡', '红烧肉', '佛跳墙', '蜂蜜柚子茶', '牛羊肉泡馍', '毛豆腐', '蚵仔煎', '刀削面', '虫草花炖鸡汤', '武昌鱼', '担担面', '肉燕', '丽江粑粑', '花旗参炖雪梨', '凉拌海带丝', '蟹壳黄', '清蒸羊羔肉', '红烧茄子', '酸菜鱼', '砂锅粥', '烤鸡', '萍乡小炒肉', '慕斯蛋糕', '土豆炖牛肉', '糊汤粉', '澳门豆捞', '西湖醋鱼', '烤串', '羊肉串', '虫草花炖瘦肉', '港式早茶', '墨西哥玉米饼', '长沙臭豆腐', '素馅馄饨', '厦门蚵仔煎', '速冻水饺', '铜锅涮肉', '莲藕汤', '土笋冻', '福州鱼丸', '手抓羊肉', '羊杂碎', '馕', '猪扒包', '糖醋排骨', '澳门杏仁饼', '可乐', '蒜蓉蒸虾', '老友粉', '豆浆', '兰州拉面', '叉烧包', '美年达', '遵义羊肉粉', '卤煮火烧', '汉堡', '菠萝油', '希腊沙拉', '花旗参炖鸡', '蒜蓉粉丝蒸扇贝', '酸奶酿皮', '木糠布甸', '烤韭菜', '景德镇鱼头', '包子', '西班牙海鲜饭', '油泼面', '馒头', '寿司', '八宝粥', '油条', '玉米', '鸡蛋', '油香', '松鼠桂鱼', '牛奶', '和乐蟹', '炸酱面', '番茄蛋花汤', '炒粉', '过桥米线', '北京烤鸭', '果汁', '椒盐虾', '羊肉泡馍', '龙井虾仁', '葡式蛋挞', '番茄炖牛腩', '白切鸡', '肉夹馍', '烤茄子', '鲁菜四喜丸子', '烤玉米', '盐水鸭'];

const toggle = () => {
  const list = foods;

  if (!run.value) {
    buttonText.value = '停止'
    timer = window.setInterval(() => {
      const index = Math.floor(Math.random() * list.length)
      const food = list[index]
      selectedFood.value = food

      const temp = document.createElement('span')
      temp.className = 'temp'
      temp.innerHTML = food

      const rTop = Math.random() * (document.documentElement.clientHeight - 50)
      const rLeft = Math.random() * (document.documentElement.clientWidth - 50)
      const rSize = Math.random() * (37 - 14) + 14
      const rColor = `rgba(0,0,0,${Math.random().toFixed(2)})`

      Object.assign(temp.style, {
        position: 'absolute',
        top: `${rTop}px`,
        left: `${rLeft}px`,
        fontSize: `${rSize}px`,
        color: rColor,
        display: 'none',
        zIndex: '-10'
      })

      if (floatContainer.value) {
        floatContainer.value.appendChild(temp)
      }
      setTimeout(() => {
        temp.style.display = 'block'
        temp.style.opacity = '1'
        temp.style.transition = 'opacity 0.5s'
        setTimeout(() => {
          temp.style.opacity = '0'
          setTimeout(() => {
            if (floatContainer.value) {
              floatContainer.value.removeChild(temp)
            }
            // floatContainer.value.removeChild(temp)
          }, 500)
        }, 500)
      }, 0)
    }, 50)
    run.value = true
  } else {
    buttonText.value = '不行，换一个'
    clearInterval(timer!)
    run.value = false
  }
}

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Enter') {
    toggle()
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleKeydown)
  if (timer) clearInterval(timer)
})

</script>

<style scoped lang="less">
.random {
  height: 100vh;
  width: 100vw;
  background-position: center;
  background-image: url("./assets/bg.jpg");
  background-repeat: repeat;
  overflow: hidden;

  .logo {
    margin: 0 auto;
    margin-top: 10px;
    width: 200px;
    height: 40px;
    background-color: #e9e9e9;

    img {
      width: 100%;
      height: 100%
    }
  }

  #wrapper {
    width: 500px;
    height: 90px;
    text-align: center;
    margin: auto;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);

    .button {
      background-color: rgba(0, 0, 0, .55);
      display: inline-block;
      padding: 0 1em;
      outline: 0;
      border: 3px solid #ddd;
      border-radius: 25px;
      color: #fff;
      font-size: 19px;
      font-family: "Microsoft YaHei";
      line-height: 2;
      cursor: pointer;
      transition: .5s;
      -webkit-appearance: none
    }
  }

  .float-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none; // 确保不阻碍点击
    z-index: 1; // 比默认内容略高，但低于弹窗等
  }
}
</style>