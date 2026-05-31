<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

// 飛機捲動進度條邏輯
const scrollProgress = ref(0);
const handleScroll = () => {
  const scrollTop = window.scrollY || document.documentElement.scrollTop;
  const scrollHeight = document.documentElement.scrollHeight - document.documentElement.clientHeight;
  scrollProgress.value = scrollHeight > 0 ? (scrollTop / scrollHeight) * 100 : 0;
};
onMounted(() => window.addEventListener('scroll', handleScroll));
onUnmounted(() => window.removeEventListener('scroll', handleScroll));

// 1. ref 資料綁定：狀態管理（切換分頁）
const currentTab = ref('home'); // 'home' (首頁) / 'itinerary' (行程) / 'packing' (行李) / 'cityGuide' (城市指南) / 'survivalGuide' (生存指南) / 'exchange' (匯率)
const isMenuOpen = ref(false); // 控制漢堡選單展開狀態

// 2. 行程避坑心法資料
const itineraryTips = ref([
  { 
    id: 1, 
    title: '一天最多排 2-3 個景點', 
    summary: '新手常犯的錯就是貪心！把行程塞滿滿只會讓你都在趕路。建議以「上午一個、下午一個、晚上視體力安排」為原則。', 
    detail: '<p><strong>💡 新手容易忽略的隱形時間：</strong><br>換飯店整理行李、排隊等車、上洗手間、在景點拍照買紀念品，這些都會消耗大把時間。</p><p><strong>✅ 具體操作建議：</strong><br>每天挑選一個「一定要去」的核心景點（大魔王），其他 1-2 個點當作附屬行程。如果當天核心景點玩得太累，附屬行程直接放棄也不會遺憾。留點白給旅行，你才有空坐下來喝杯咖啡。</p>', 
    image: 'https://images.unsplash.com/photo-1499540633125-484965b60031?auto=format&fit=crop&w=600&q=80' 
  },
  { 
    id: 2, 
    title: '預留迷路與找路的時間', 
    summary: '在異國找路、搭錯車、找置物櫃都是非常正常的！建議每個點之間多抓 30-40 分鐘的緩衝時間，心情才不會焦慮。', 
    detail: '<p><strong>🧭 新手求生指南：</strong><br>不要把交通時間算得剛剛好。例如 Google Maps 顯示搭地鐵要 20 分鐘，請自動在行程表上寫 40 分鐘。</p><p><strong>✅ 實用小撇步：</strong><br>下飛機第一件事，先在手機存好飯店的英文/當地語言地址截圖。在大車站迷路時，不要一味埋頭看導航，直接把截圖拿給站務員或路邊店家看，通常 10 秒鐘就能幫你指明方向，省下大量盲目摸索的時間。</p>', 
    image: 'https://images.unsplash.com/photo-1506012787146-f92b2d7d6d96?auto=format&fit=crop&w=600&q=80' 
  },
  { 
    id: 3, 
    title: '找餐廳一定要有 B 備案', 
    summary: '網路上大推的必吃美食，現場可能大排長龍或是遇到臨時公休。記得在 Google Maps 上多存幾家附近的備案餐廳。', 
    detail: '<p><strong>⚠️ 避坑指南：</strong><br>越知名的網紅店，排隊時間越難預估（常常一排就是 1-2 小時起跳），這會直接拖垮你接下來的所有行程。</p><p><strong>✅ 聰明點餐法：</strong><br>出發前，在核心景點周邊的 Google Maps 上，點擊『餐廳』標籤，找出 2-3 家評分在 4.2 顆星以上、但不是超級名店的在地小館或美食街作為備案。當肚子極餓、名店又要排隊時，立刻啟用備案，才不會因為低血糖在國外跟旅伴吵架。</p>', 
    image: 'https://images.unsplash.com/photo-1514933651103-005eec06c04b?auto=format&fit=crop&w=600&q=80' 
  },
  { 
    id: 4, 
    title: '頭尾兩天行程保持空白', 
    summary: '第一天剛下飛機通常很疲憊，最後一天則要提前抵達機場。這兩天最好只安排在飯店附近吃飯或簡單購物，別跑太遠。', 
    detail: '<p><strong>⏱️ 時間算給你聽：</strong><br>假設下午 2 點飛機降落，出關（1小時）+ 機場車程到市區（1小時）+ 飯店 Check-in 放行李，弄完通常已經傍晚 5-6 點了。</p><p><strong>✅ 正確的節奏：</strong><br><strong>第一天：</strong> 飯店放完行李後，在週邊商圈吃個熱騰騰的晚餐，去便利商店採買零食，早點休息適應環境。<br><strong>最後一天：</strong> 必須提早 2.5-3 小時到機場。當天早上唯一的行程就是吃個悠閒的早餐、檢查行李，然後直接前往機場免稅店做最後採購，拒絕任何驚險趕飛機的行為。</p>', 
    image: 'https://images.unsplash.com/photo-1436491865332-7a61a109cc05?auto=format&fit=crop&w=600&q=80' 
  },
  { 
    id: 5, 
    title: '出發前確認營業時間與公休日', 
    summary: '很多國家的博物館、遊樂園或知名餐廳會有固定的公休日（例如週一休館）。排行程時務必再次確認，免得現場撲空。', 
    detail: '<p><strong>💣 新手常踩的雷：</strong><br>以為景點天天都開。例如：日本很多公營博物館週一休館、韓國景福宮週二休館、歐洲許多商店週日甚至完全不營業！</p><p><strong>✅ 行前檢查清單：</strong><br>在台灣排好行程的當週，逐一用 Google Maps 檢查你安排該景點的那天（例如週三），當地的營業時間是否正常。若遇到公修，立刻跟其他天對調行程。</p>', 
    image: 'https://live.staticflickr.com/65535/48178189286_a1a84c1047_b_d.jpg' 
  },
  { 
    id: 6, 
    title: '景點「分區集中」安排，拒絕折返跑', 
    summary: '強烈建議打開 Google Maps 將景點標上星星，把距離相近的排在同一天，才不會浪費大量時間和體力在搭車折返跑。', 
    detail: '<p><strong>🗺️ 視覺化排行程法：</strong><br>新手排行程最忌諱『看文字清單』排。你應該把想去的地方全部在 Google Maps 上存成同一個清單（例如：2026日本行）。</p><p><strong>✅ 連連看原則：</strong><br>當地圖上的星星全部浮現後，你會發現它們自然聚集成幾大群落。把同一個圈圈裡的景點排在同一天（例如：淺草+晴空塔一天；新宿+澀谷一天）。這能幫你省下至少一半以上的交通費與寶貴的腿力。</p>', 
    image: 'https://img.magnific.com/free-vector/location-pin-multiple-colours-set-four_78370-8200.jpg?semt=ais_hybrid&w=740&q=80' 
  },
  { 
    id: 7, 
    title: '永遠要有「雨天備案」', 
    summary: '天氣是旅行中最不可控的變數！如果當天安排了戶外樂園、大自然景點，一定要在附近找好幾個室內備案。', 
    detail: '<p><strong>🌧️ 心態與準備：</strong><br>下雨通常會重創戶外拍照和行走的興致。與其到了現場掃興，不如在排行程時就做好『晴雨交叉表』。</p><p><strong>✅ 萬用室內備案清單：</strong><br>如果氣象預報當天降雨機率高，果斷將行程切換成：大型 Shopping Mall、水族館、室內主題樂園、美術館、或是預約一間極有質感的下午茶咖啡廳。人在室內，下雨照樣可以玩得很優雅！</p>', 
    image: 'https://images.unsplash.com/photo-1534274988757-a28bf1a57c17?auto=format&fit=crop&w=600&q=80' 
  },
  { 
    id: 8, 
    title: '別低估「車站內」的步行時間', 
    summary: 'Google Maps 顯示搭車 10 分鐘，不代表你 10 分鐘後就能抵達。在大城市魔王級大站，光轉乘就要在車站裡走上 10 到 15 分鐘！', 
    detail: '<p><strong>🚶 魔王大車站真相：</strong><br>東京的新宿、梅田，首爾的弘大、首爾車站，結構複雜得像迷宮。從地鐵閘門走到正確的月台，或是兩條路線之間的轉乘，在裡面走上 1 萬步都是常有的事。</p><p><strong>✅ 給新手的安全建議：</strong><br>如果你當天有訂好預約制餐廳，或是要趕特定班次的火車/新幹線，請將 Google Maps 顯示的交通時間直接乘上 1.5 倍。多出來的時間讓你可以在大車站裡看懂指標、安心找路，不用崩潰狂奔。</p>', 
    image: 'https://shared.fastly.steamstatic.com/store_item_assets/steam/apps/287980/ss_4801bccf6ca0a7ca5dd6c0bfd03cbca0321741b3.1920x1080.jpg?t=1724386220' 
  },
  { 
    id: 9, 
    title: '熱門票券與餐廳必須「提前預訂」', 
    summary: '現在全球旅遊大爆發，熱門地標（如澀谷 SKY、環球影城快速通關）現場通常買不到票。行程大綱一出來，務必提早 1 到 2 個月預訂。', 
    detail: '<p><strong>🎫 殘酷的現實：</strong><br>像東京 SHIBUYA SKY、哈利波特影城，或是知名地標的觀景台，現在都實施總量管制。當天現場排隊是絕對進不去的。</p><p><strong>✅ 行前必做：</strong><br>只要行程大綱定案，第一件事就是上 Klook、KKday 或官網查詢放票時間。熱門時段（如黃昏、假日）通常一個月前開放就秒殺。提早買好票，到現場直接掃 QR Code 快速通關，才是聰明旅人的做法。</p>', 
    image: 'https://meet.eslite.com/CMS/Files/@M091/2023-07/south_korea_attractions/south_korea_attractions05.jpg' 
  }
]);

// 3. 行李清單資料
const defaultPackingList = [
  { id: 1, name: '護照 (效期需大於六個月！)', checked: false, isEssential: true },
  { id: 2, name: '當地網卡 / eSIM / Wi-Fi 機', checked: false, isEssential: true },
  { id: 3, name: '海外高回饋信用卡與適量當地現金', checked: false, isEssential: true },
  { id: 4, name: '萬用轉接頭與行動電源', checked: false, isEssential: true },
  { id: 5, name: '個人常備藥品 (腸胃藥、止痛藥)', checked: false, isEssential: false },
  { id: 6, name: '舒適好走的鞋子 (絕對不要穿新鞋！)', checked: false, isEssential: false }
];

// 透過 JSON 深拷貝初始化，避免改動到預設資料
const packingList = ref(JSON.parse(JSON.stringify(defaultPackingList)));

onMounted(() => {
  const savedList = localStorage.getItem('travel-packing-list');
  if (savedList) {
    packingList.value = JSON.parse(savedList);
  }
});

watch(packingList, (newList) => {
  localStorage.setItem('travel-packing-list', JSON.stringify(newList));
}, { deep: true }); // deep: true 才能監聽到陣列內部物件 (如 checked) 的改變

// 新增自訂行李功能
const newItemName = ref('');
const addNewItem = () => {
  if (newItemName.value.trim() !== '') {
    packingList.value.push({
      id: Date.now(),
      name: newItemName.value,
      checked: false,
      isEssential: false
    });
    newItemName.value = ''; // 新增後清空輸入框
  }
};

// 計算行李打包完成度
const progress = computed(() => {
  if (packingList.value.length === 0) return 0;
  const completed = packingList.value.filter(item => item.checked).length;
  return Math.round((completed / packingList.value.length) * 100);
});

// 清除所有已勾選的行李項目
const clearCheckedItems = () => {
  packingList.value.forEach(item => {
    item.checked = false;
  });
};

// 刪除特定的行李項目
const deleteItem = (id) => {
  packingList.value = packingList.value.filter(item => item.id !== id);
};

// 還原預設行李清單
const resetPackingList = () => {
  if (confirm('確定要還原成預設的行李清單嗎？這會清除你所有自訂的項目喔！')) {
    packingList.value = JSON.parse(JSON.stringify(defaultPackingList));
  }
};

// 4. 多層級熱門城市指南資料 (Mock Data)
const countriesData = ref([
  {
    id: 'japan',
    code: 'JP',
    name: '日本',
    image: 'https://images.unsplash.com/photo-1493976040374-85c8e12f0c0e?auto=format&fit=crop&w=600&q=80',
    cities: [
      {
        id: 'tokyo',
        name: '東京',
        enName: 'Tokyo',
        image: 'https://images.unsplash.com/photo-1540959733332-eab4deabeeaf?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '澀谷天空 (SHIBUYA SKY)', 
            description: '東京目前最火紅的露天觀景台，擁有 360 度無死角絕美市景。',
            tickets: '極度搶手！強烈建議提前 4 週於 Klook 或官網預訂「日落時段」門票。',
            hours: '10:00 - 22:30 (最後入場 21:20)',
            transport: '搭乘各線至「澀谷站」，經由 Scramble Square 電梯直達 14 樓售票處。',
            website: 'https://www.shibuya-scramble-square.com/sky/'
          },
          { 
            name: '淺草寺', 
            description: '東京最古老的寺廟，著名的「雷門」大燈籠是必拍地標。',
            tickets: '免費參觀。',
            hours: '本堂開放時間為 06:00 - 17:00 (10月至3月為 06:30 開門)，境內24小時開放。',
            transport: '搭乘銀座線或都營淺草線至「淺草站」步行 5 分鐘。',
            website: 'https://www.senso-ji.jp/'
          },
          {
            name: '東京鐵塔',
            description: '最具代表性的東京經典地標，橘紅色的塔身在夜間點燈後非常浪漫。',
            tickets: '可於現場購票或透過 KKday 提前購買 Main Deck 門票 (約 1,200 日圓)。',
            hours: '09:00 - 22:30 (最後入場 22:00)',
            transport: '搭乘大江戶線至「赤羽橋站」步行 5 分鐘。',
            website: 'https://www.tokyotower.co.jp/'
          }
        ],
        hours: '百貨公司大多於 10:00 - 20:00 營業，部分餐廳與居酒屋會營業至 22:00 甚至更晚。',
        notes: '東京地鐵在通勤時間非常擁擠，新手排行程時請盡量避開早上 8:00-9:00 及晚上 17:30-19:00 的尖峰時段喔！'
      },
      {
        id: 'osaka',
        name: '大阪',
        enName: 'Osaka',
        image: 'https://images.unsplash.com/photo-1590559899731-a382839e5549?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '環球影城 (USJ)', 
            description: '亞洲首座環球影城，包含超級任天堂世界、哈利波特魔法世界等超高人氣園區。',
            tickets: '建議提前 1-2 個月於 Klook 或 KKday 購買「門票」與「快速通關」。若想進任天堂園區，需提早抽整理券或購買保證入園方案。',
            hours: '每日開園時間不同 (約 08:30 - 21:30)，請務必提前至官網查詢行事曆。',
            transport: '搭乘 JR 夢咲線至「環球城站 (Universal City)」，出站步行 5 分鐘。',
            website: 'https://www.usj.co.jp/web/zh/tw'
          },
          {
            name: '道頓堀 & 心齋橋',
            description: '大阪的美食與購物中心，必拍固力果跑跑人招牌。',
            tickets: '免費，帶著空肚子跟錢包來就好！',
            hours: '多數商店營業至 21:00，餐廳與居酒屋則至深夜。',
            transport: '搭乘御堂筋線至「難波站」或「心齋橋站」。',
            website: null
          },
          {
            name: '大阪城',
            description: '大阪的歷史地標，天守閣非常壯觀，春天更是賞櫻勝地。',
            tickets: '天守閣門票成人 600 日圓，若搭配大阪周遊卡可免費入場。',
            hours: '09:00 - 17:00 (最後入場 16:30，櫻花季會延長)',
            transport: '搭乘谷町線或中央線至「谷町四丁目站」步行。',
            website: 'https://www.osakacastle.net/'
          }
        ],
        hours: '環球影城開園時間每日不同，建議提前查好並提早 1 小時到場排隊。',
        notes: '大阪人熱情且步調稍快。搭乘手扶梯時，大阪習慣「站右邊，左邊通行」（與東京剛好相反）。'
      },
      {
        id: 'kyoto',
        name: '京都',
        enName: 'Kyoto',
        image: 'https://images.unsplash.com/photo-1624253321171-1be53e12f5f4?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '清水寺', 
            description: '京都最古老的寺廟，著名的「清水舞台」懸空建造，四季風景皆美。',
            tickets: '現場購票，成人約 400 日圓。',
            hours: '通常為 06:00 - 18:00 (特定夜間參拜期間會延長)。',
            transport: '於京都車站搭乘市營巴士 206 號至「五条坂」下車步行 10 分鐘。',
            website: 'https://www.kiyomizudera.or.jp/'
          },
          { 
            name: '伏見稻荷大社', 
            description: '以綿延不絕的「千本鳥居」聞名，是外國遊客心中最具代表性的日本景點之一。',
            tickets: '免費參觀。',
            hours: '24 小時開放 (建議清晨前往避開人潮)。',
            transport: '搭乘 JR 奈良線至「稻荷站」下車即達。',
            website: 'http://inari.jp/'
          },
          { 
            name: '嵐山', 
            description: '竹林小徑與渡月橋是必訪景點，可以搭乘充滿復古風情的嵐山小火車。',
            tickets: '嵐山小火車單程約 880 日圓，強烈建議提前線上預約。',
            hours: '景點多為 24 小時開放，但小火車與周邊寺廟通常 17:00 關門。',
            transport: '搭乘 JR 嵯峨野線至「嵯峨嵐山站」。',
            website: 'https://www.sagano-kanko.co.jp/'
          }
        ],
        hours: '多數寺廟與古蹟會在 17:00 左右關門，建議把這類行程排在早上。',
        notes: '京都的公車系統發達但也容易塞車，若距離不遠建議搭乘地鐵或是租借腳踏車移動。'
      }
    ]
  },
  {
    id: 'korea',
    code: 'KR',
    name: '韓國',
    image: 'https://images.unsplash.com/photo-1517154421773-0529f29ea451?auto=format&fit=crop&w=600&q=80',
    cities: [
      {
        id: 'seoul',
        name: '首爾',
        enName: 'Seoul',
        image: 'https://media.istockphoto.com/id/2169581951/zh/%E7%85%A7%E7%89%87/seoul-south-korea-skyline-at-night.jpg?s=612x612&w=0&k=20&c=DLriPqSmZaC3odVpavSRlaUsG3E4mgKo2ruEW3dG9-c=',
        attractions: [
          { 
            name: '景福宮', 
            description: '韓國最具代表性的朝鮮時代宮殿，附近有很多韓服體驗店。',
            tickets: '成人 3,000 韓元。💡 隱藏技巧：只要穿著傳統韓服，即可「免費」入園！',
            hours: '09:00 - 18:00 (冬季提前至 17:00)，每週二公休！',
            transport: '搭乘地鐵 3 號線至「景福宮站」 5 號出口。',
            website: 'http://www.royalpalace.go.kr/'
          },
          {
            name: '南山首爾塔',
            description: '首爾的地標，能將首爾市區夜景盡收眼底，也是韓劇的熱門拍攝地。',
            tickets: '可於 KKday 提前購買纜車及觀景台套票。',
            hours: '10:30 - 22:30 (假日延長至 23:00)。',
            transport: '搭乘南山纜車，或由明洞站搭乘南山循環巴士 01 號。',
            website: 'https://www.nseoultower.co.kr/'
          },
          {
            name: '明洞商圈',
            description: '韓國美妝、服飾與街頭小吃的大本營，這裡換錢所的匯率通常也是最好的。',
            tickets: '免費。',
            hours: '街頭攤販約從 16:00 開始營業至 23:00。',
            transport: '搭乘地鐵 4 號線至「明洞站」。',
            website: null
          }
        ],
        hours: '東大門、明洞等熱門商圈多半營業至 22:00 或深夜，是很棒的晚上行程。',
        notes: '韓國部分傳統餐廳有「最少需點兩人份」的規定，如果是獨旅，建議先查好「一人友善」的餐廳名單。'
      },
      {
        id: 'busan',
        name: '釜山',
        enName: 'Busan',
        image: 'https://blog-static.kkday.com/zh-hk/blog/wp-content/uploads/%E6%B5%B7%E9%9B%B2%E5%8F%B0%E5%B0%8F%E7%81%AB%E8%BB%8A-supplier.jpg',
        attractions: [
          { 
            name: '海雲台', 
            description: '釜山最著名的海灘，夏季有海水浴場，周邊可搭乘超人氣的海濱小火車 (Blue Line Park)。',
            tickets: '海灘免費。海濱小火車極為熱門，務必提早 2-3 週上官網預訂。',
            hours: '全天開放。',
            transport: '搭乘地鐵 2 號線至「海雲台站」步行前往。',
            website: 'https://www.bluelinepark.com/'
          },
          {
            name: '甘川洞文化村',
            description: '被稱為「韓國的馬丘比丘」，滿山色彩繽紛的可愛小房子，必拍小王子雕像。',
            tickets: '免費參觀，可購買地圖集章。',
            hours: '09:00 - 18:00 (此為居民生活區，請放低音量)。',
            transport: '搭乘地鐵 1 號線至「土城站」，轉乘小巴 (沙下1-1, 西區2, 西區2-2) 上山。',
            website: null
          },
          {
            name: '廣安里海水浴場',
            description: '釜山年輕人最愛的海灘，晚上可以看到閃閃發光的廣安大橋，週末常有燈光秀或無人機表演。',
            tickets: '免費。',
            hours: '全天開放。',
            transport: '搭乘地鐵 2 號線至「廣安站」步行 10 分鐘。',
            website: null
          }
        ],
        hours: '海鮮市場通常一早就開，很適合安排為早午餐行程；部分海景咖啡廳營業至 20:00。',
        notes: '釜山地形多山丘，前往甘川洞等景點請務必穿著好走的鞋子，建議搭乘公車或計程車上山。'
      },
      {
        id: 'jeju',
        name: '濟州島',
        enName: 'Jeju',
        image: 'https://images.trvl-media.com/place/6119706/4772a0ed-6cb4-49ba-b0b0-7e084e8f2606.jpg',
        attractions: [
          { 
            name: '城山日出峰', 
            description: '濟州島最著名的世界自然遺產，是一座由海底火山爆發形成的火山口。',
            tickets: '成人 5,000 韓元。',
            hours: '07:30 - 19:00 (每月第一個週一公休)。',
            transport: '距離市區較遠，建議搭乘急行巴士 111 號或包車前往。',
            website: 'https://www.visitjeju.net/'
          },
          {
            name: '牛島',
            description: '位於濟州島東部的絕美小島，海水清澈見底，可以租借電動腳踏車環島，必吃花生冰淇淋。',
            tickets: '來回船票約 10,500 韓元。',
            hours: '船班受天氣影響，通常為 08:00 - 17:00 之間。',
            transport: '至「城山港」搭船，航程約 15 分鐘。',
            website: null
          },
          {
            name: '東門傳統市場',
            description: '濟州島歷史最悠久的傳統市場，晚上會變身為熱鬧的夜市，可以買到各式柑橘伴手禮跟黑豬肉小吃。',
            tickets: '免費。',
            hours: '日間市場 08:00 - 21:00，夜市 18:00 - 24:00。',
            transport: '位於濟州市區，可搭乘多路公車至「東門圓環站」。',
            website: null
          }
        ],
        hours: '自然景點開放時間多受日落時間影響，冬天前往建議 16:00 前結束戶外行程。',
        notes: '濟州島大眾運輸班次較少且距離遠，若不包車或自駕，建議提前查好公車時刻表。'
      }
    ]
  },
  {
    id: 'china',
    code: 'CN',
    name: '中國',
    image: 'https://img.magnific.com/free-photo/day-city-view_1417-1835.jpg?semt=ais_hybrid&w=740&q=80',
    cities: [
      {
        id: 'shanghai',
        name: '上海',
        enName: 'Shanghai',
        image: 'https://images.unsplash.com/photo-1538428494232-9c0d8a3ab403?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '外灘 (The Bund)', 
            description: '欣賞黃浦江兩岸的萬國建築博覽群與陸家嘴現代摩天大樓，夜景極其震撼。',
            tickets: '免費參觀。',
            hours: '全天開放，觀景燈光通常在 22:00 關閉。',
            transport: '搭乘地鐵2號線或10號線至「南京東路站」，步行約10分鐘。',
            website: null
          },
          { 
            name: '東方明珠廣播電視塔', 
            description: '上海經典地標，可於高空透明觀光廊俯瞰上海市區全景。',
            tickets: '成人約 199 人民幣起，建議提前線上購票。',
            hours: '09:00 - 21:00。',
            transport: '搭乘地鐵2號線至「陸家嘴站」。',
            website: 'https://www.orientalpearltower.com/'
          },
          { 
            name: '上海迪士尼度假區', 
            description: '亞洲最大的迪士尼樂園，擁有獨家且必玩的「創極速光輪」雲霄飛車。',
            tickets: '依淡旺季浮動（約 475 - 799 人民幣），強烈建議提前於官網或 Klook 購買。',
            hours: '通常為 08:30 - 21:30，請依官網當日公告為準。',
            transport: '搭乘地鐵11號線終點站「迪士尼站」。',
            website: 'https://www.shanghaidisneyresort.com/'
          }
        ],
        hours: '市區商場與熱門景點大多營業至 21:30-22:00。',
        notes: '🥟 必吃小吃：生煎包、小籠包、排骨年糕。\n⚠️ 注意事項：中國大陸行動支付普及率近 100%，強烈建議出發前在台灣先下載並綁定好「支付寶 (Alipay)」或「微信支付 (WeChat Pay)」的信用卡。'
      },
      {
        id: 'chengdu',
        name: '成都',
        enName: 'Chengdu',
        image: 'https://p2.itc.cn/q_70/images03/20231009/d6cb99fa71504f0e93ec2e3c075a076d.jpeg',
        attractions: [
          { 
            name: '成都大熊貓繁育研究基地', 
            description: '近距離觀賞可愛的大熊貓與小熊貓，也是頂流女明星「花花」的家。',
            tickets: '全票 55 人民幣，強烈建議提前於微信小程式預訂。',
            hours: '07:30 - 18:00（建議一早就入園，熊貓早上最活躍，下午大多在睡覺）。',
            transport: '搭乘地鐵3號線至「熊貓大道站」，轉乘景區直通車。',
            website: null
          },
          { 
            name: '寬窄巷子', 
            description: '保留清朝古街道的市井文化，有許多茶館、掏耳朵體驗與特色文創小店。',
            tickets: '免費參觀。',
            hours: '全天開放，多數店家營業至 22:00。',
            transport: '搭乘地鐵4號線至「寬窄巷子站」。',
            website: null
          },
          { 
            name: '錦里古街', 
            description: '體驗濃厚三國文化的古風商業街，夜晚紅燈籠亮起時氣氛極佳，非常適合拍照。',
            tickets: '免費參觀。',
            hours: '全天開放，夜晚燈景最美。',
            transport: '搭乘公車或計程車至武侯祠附近。',
            website: null
          }
        ],
        hours: '成都是著名的不夜城，古街與春熙路商圈的串串香店很多都營業至深夜。',
        notes: '🍢 必吃小吃：擔擔麵、串串香、三大炮、道地麻辣火鍋。\n⚠️ 注意事項：四川飲食偏麻辣且重油，腸胃較敏感的新手出發前記得準備個人常備腸胃藥。'
      },
      {
        id: 'chongqing',
        name: '重慶',
        enName: 'Chongqing',
        image: 'https://images.unsplash.com/photo-1555899434-94d1368aa7af?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '洪崖洞', 
            description: '依山而建的傳統吊腳樓，夜晚點燈後金碧輝煌，被譽為現實版的「神隱少女」湯屋。',
            tickets: '免費，但旺季強烈建議提前於微信公眾號預約入園以防限流。',
            hours: '亮燈時間依季節變化（通常為 20:00 - 23:00）。',
            transport: '地鐵1號線或6號線至「小什字站」，出站後步行前往。',
            website: null
          },
          { 
            name: '李子壩站 (輕軌穿樓)', 
            description: '重慶超奇特的交通奇景，輕軌列車直接穿過居民樓的壯觀畫面。',
            tickets: '只要購買輕軌車票即可在車上體驗，或是到站外觀景台免費拍照。',
            hours: '跟隨輕軌營運時間。',
            transport: '搭乘輕軌2號線至「李子壩站」。',
            website: null
          },
          { 
            name: '解放碑', 
            description: '重慶最繁華的市中心步行街，也是抗戰勝利的精神象徵，周圍百貨商場林立。',
            tickets: '免費。',
            hours: '全天開放。',
            transport: '地鐵2號線至「臨江門站」，或地鐵1號線至「較場口站」。',
            website: null
          }
        ],
        hours: '重慶的火鍋店與消夜文化極度興盛，深夜 1-2 點在街上找東西吃非常容易。',
        notes: '🌶️ 必吃小吃：重慶小麵、酸辣粉、九宮格火鍋、酥肉。\n⚠️ 注意事項：重慶被稱為「8D 魔幻城市」，有時在 1 樓搭電梯到 10 樓出來發現還是 1 樓的馬路。導航在這裡極易失效，找路時「多問路人」才是上策！'
      },
      {
        id: 'xiamen',
        name: '廈門',
        enName: 'Xiamen',
        image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/5/51/Xiamen_night_cityscape_2018_-_Flickr_-_Jaykhuang.jpg/330px-Xiamen_night_cityscape_2018_-_Flickr_-_Jaykhuang.jpg',
        attractions: [
          { 
            name: '鼓浪嶼', 
            description: '充滿歐式洋樓與文青小店的「海上花園」，全島禁行汽機車，漫步極為愜意。',
            tickets: '來回船票約 35-50 人民幣，需攜帶台胞證並「提前實名線上購票」。',
            hours: '全天開放，但旅客需依購買的船班時間往返。',
            transport: '至廈門郵輪中心廈鼓碼頭搭船。',
            website: null
          },
          { 
            name: '南普陀寺', 
            description: '閩南著名的佛教聖地，背山面海，風景秀麗，旁邊緊鄰廈門大學。',
            tickets: '免費，但通常需提前於微信小程式預約。',
            hours: '08:00 - 17:00。',
            transport: '搭乘公車至「南普陀站」或「廈大西村站」。',
            website: null
          },
          { 
            name: '曾厝垵', 
            description: '由傳統漁村改造而成的文創美食街，巷弄交錯，有超級多特色小吃。',
            tickets: '免費參觀。',
            hours: '多數攤販與店家營業至晚上 23:00。',
            transport: '搭乘公車或沿環島路騎行自行車前往。',
            website: null
          }
        ],
        hours: '鼓浪嶼多數景點與店家約在 18:00 關門，建議白天前往；曾厝垵與中山路步行街晚上最熱鬧。',
        notes: '🍜 必吃小吃：沙茶麵、海蠣煎、土筍凍、花生湯。\n⚠️ 注意事項：廈門氣候溫和，但夏天較為炎熱且偶有颱風，安排海島或乘船行程請務必密切關注天氣預報。'
      }
    ]
  },
  {
    id: 'southeast-asia',
    code: 'SEA',
    name: '東南亞',
    image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS7PNR1mZRFPMHXlOlPAH5ceoosZx7bcPbzsQ&s',
    cities: [
      {
        id: 'bangkok',
        name: '曼谷',
        enName: 'Bangkok',
        image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRV-ObMOCxpCRea38aVoZccza4isMEXapsjwA&s',
        attractions: [
          { 
            name: 'ICONSIAM 暹羅天地', 
            description: '號稱世界級的奢華百貨，一樓直接把超浮誇的泰國水上市場搬進室內！',
            tickets: '免費參觀。',
            hours: '10:00 - 22:00',
            transport: '搭乘 BTS 至「Saphan Taksin 站」轉乘免費接駁船。',
            website: 'https://www.iconsiam.com/'
          },
          { 
            name: '喬德夜市 (Jodd Fairs)', 
            description: '曼谷目前最火紅的文青夜市，環境乾淨且充滿創意小店，必吃巨無霸火山排骨。',
            tickets: '免費入場。',
            hours: '16:00 - 00:00',
            transport: '搭乘 MRT 至「Phra Ram 9 站」步行 5 分鐘。',
            website: null
          },
          { 
            name: '鄭王廟 (Wat Arun)', 
            description: '絕美的白色佛塔，是體驗穿著傳統泰服拍照的最佳地點。',
            tickets: '門票 100 泰銖，周邊租泰服約 200-300 泰銖。',
            hours: '08:00 - 18:00',
            transport: '搭乘 MRT 至「Itsaraphap 站」步行，或搭乘渡輪至 Wat Arun 碼頭。',
            website: null
          }
        ],
        hours: '夜市通常從 17:00 營業至午夜，百貨商場則約在 10:00 - 22:00。',
        notes: '🥘 必吃小吃：火山排骨、芒果糯米飯、泰式奶茶、冬蔭功。\n⚠️ 注意事項：曼谷市區塞車非常嚴重，上下班尖峰時間請盡量搭乘 BTS 或 MRT，避免搭計程車卡在車陣中。'
      },
      {
        id: 'phuquoc',
        name: '富國島',
        enName: 'Phu Quoc',
        image: 'https://master-orange.s3.ap-northeast-1.amazonaws.com/media/6806cc61d7f84.jpg',
        attractions: [
          { 
            name: '太陽世界香島自然公園 (世界最長跨海纜車)', 
            description: '搭乘金氏世界紀錄最長的跨海纜車，鳥瞰富國島絕美海景與漁村。',
            tickets: '建議於 Klook 或 KKday 提前購買纜車與樂園套票。',
            hours: '09:00 - 17:00（注意纜車中午有停駛休息時間）。',
            transport: '位於富國島南部，建議搭乘 Grab 或飯店接駁車前往。',
            website: 'https://honthom.sunworld.vn/'
          },
          { 
            name: '珍珠野生動物園 (Vinpearl Safari)', 
            description: '全越南最大的半野生動物園，可以搭乘遊園車近距離看獅子老虎，還能餵長頸鹿。',
            tickets: '成人約 650,000 越南盾，可提前線上購票。',
            hours: '09:00 - 16:00',
            transport: '位於富國島北部，可搭乘 Vinbus 免費接駁車。',
            website: 'https://vinwonders.com/en/vinpearl-safari-phu-quoc/'
          },
          { 
            name: '富國島大世界 (Grand World)', 
            description: '被稱為「越南威尼斯」，擁有超好拍的運河與歐式建築，晚上還有免費的大型水舞秀。',
            tickets: '園區免費開放，搭乘貢多拉船需另外購票。',
            hours: '全天開放，水舞秀通常在 21:30。',
            transport: '位於富國島北部，緊鄰珍珠遊樂園。',
            website: null
          }
        ],
        hours: '樂園及纜車通常 09:00 開始營業，傍晚 17:00 左右關閉，行程請排在白天。',
        notes: '🦑 必吃小吃：越南河粉 (Pho)、越式法國麵包、海鮮燒烤、椰子冰淇淋。\n⚠️ 注意事項：雖然台灣人直飛富國島免簽證，但若要轉機到越南其他城市（如胡志明市）則必須事先辦理越南簽證！'
      },
      {
        id: 'chiangmai',
        name: '清邁',
        enName: 'Chiang Mai',
        image: 'https://res.klook.com/image/upload/fl_lossy.progressive,q_60/Mobile/City/cswglxpstphljdourpfu.jpg',
        attractions: [
          { 
            name: '塔佩門與古城區 (Tha Phae Gate)', 
            description: '清邁古城區的東門，保留了完整的紅磚城牆，是餵鴿子與拍文青照的必訪地標。',
            tickets: '免費。',
            hours: '24 小時開放。',
            transport: '位於古城區東側，從市區多數地方步行或搭雙條車皆可輕鬆抵達。',
            website: null
          },
          { 
            name: '清邁週日步行街 (Sunday Night Market)', 
            description: '全清邁最大、最熱鬧的市集，充滿當地手工藝品、服飾與街頭美食。',
            tickets: '免費。',
            hours: '每週日 16:00 - 22:30',
            transport: '從塔佩門一路延伸至古城內（Ratchadamnoen Road）。',
            website: null
          },
          { 
            name: '素貼山雙龍寺 (Wat Phra That Doi Suthep)', 
            description: '清邁最著名的寺廟，擁有一座金碧輝煌的佛塔，還能從山頂俯瞰清邁市區全景。',
            tickets: '門票 30 泰銖，搭乘電梯另加 20 泰銖。',
            hours: '06:00 - 20:00',
            transport: '建議於古城區或清邁大學門口搭乘紅色雙條車上山。',
            website: null
          }
        ],
        hours: '週末的文創市集與夜市大多從下午 16:00 開始，寺廟則通常在 17:00 關門。',
        notes: '🥥 必吃小吃：泰北咖哩麵 (Khao Soi)、泰北香腸、香蕉煎餅。\n⚠️ 注意事項：清邁古城區多為單行道，若租借機車務必配戴安全帽並攜帶國際駕照，當地警察經常臨檢。'
      },
      {
        id: 'singapore',
        name: '新加坡',
        enName: 'Singapore',
        image: 'https://images.unsplash.com/photo-1525625293386-3f8f99389edd?auto=format&fit=crop&w=600&q=80',
        attractions: [
          { 
            name: '星耀樟宜 (Jewel Changi)', 
            description: '擁有世界最高室內瀑布「雨漩渦」的超浮誇機場商場，帶來被熱帶雨林包圍的視覺震撼。',
            tickets: '商場免費，部分頂樓遊樂設施需購票。',
            hours: '商場 24 小時營業，瀑布開放時間約為 11:00 - 22:00。',
            transport: '直接連接樟宜機場 T1、T2、T3 航廈。',
            website: 'https://www.jewelchangiairport.com/'
          },
          { 
            name: '濱海灣花園 (Gardens by the Bay)', 
            description: '宛如阿凡達世界的擎天樹叢 (Supertrees)，以及兩座壯觀的冷室植物園。',
            tickets: '戶外花園免費；冷室植物園建議提前線上購票。',
            hours: '戶外區域 05:00 - 02:00；冷室 09:00 - 21:00。',
            transport: '搭乘地鐵至「海灣舫站 (Bayfront)」。',
            website: 'https://www.gardensbythebay.com.sg/'
          },
          { 
            name: '聖淘沙環球影城 (USS)', 
            description: '東南亞唯一的環球影城，園區小而美，必玩《變形金剛》與《神鬼傳奇》設施。',
            tickets: '強烈建議提早於 Klook 購票。',
            hours: '10:00 - 19:00（時間常有變動，請以官網為準）。',
            transport: '搭乘地鐵至「港灣站 (HarbourFront)」轉乘聖淘沙捷運。',
            website: 'https://www.rwsentosa.com/en/attractions/universal-studios-singapore'
          }
        ],
        hours: '濱海灣花園擎天樹免費區開放至凌晨 02:00，燈光秀每晚 19:45 及 20:45 各一場。',
        notes: '🦀 必吃小吃：辣椒螃蟹、海南雞飯、肉骨茶、咖椰吐司。\n⚠️ 注意事項：新加坡法規極為嚴格，絕對禁止攜帶口香糖入境，且在地鐵站及車廂內連「喝水」都會面臨高額罰款！'
      },
      {
        id: 'kualalumpur',
        name: '吉隆坡',
        enName: 'Kuala Lumpur',
        image: 'https://www.ijmland.com/MM2H/hk-zh/images/gallery/full/klcity.jpg',
        attractions: [
          { 
            name: '雙峰塔 (Petronas Twin Towers)', 
            description: '吉隆坡最具代表性的地標，曾經是世界第一高樓，晚上的點燈極其耀眼。',
            tickets: '底部商場與 KLCC 公園免費；登塔需購票（約 85 馬幣）。',
            hours: '觀景台 09:00 - 21:00（每週一休館）。',
            transport: '搭乘輕軌 LRT 至「KLCC 站」。',
            website: 'https://www.petronastwintowers.com.my/'
          },
          { 
            name: '黑風洞 (Batu Caves)', 
            description: '壯觀的石灰岩溶洞神廟，前方有著巨大的金色神像與彩虹階梯。',
            tickets: '免費參觀。',
            hours: '08:00 - 19:00',
            transport: '搭乘 KTM 通勤鐵路至「Batu Caves 站」。',
            website: null
          },
          { 
            name: '亞羅街夜市 (Jalan Alor)', 
            description: '吉隆坡最熱鬧的美食街，整條街排滿了大排檔，是吃宵夜的絕佳去處。',
            tickets: '免費。',
            hours: '16:00 - 03:00（越晚越熱鬧）。',
            transport: '搭乘單軌列車 Monorail 至「武吉免登站 (Bukit Bintang)」，步行 5 分鐘。',
            website: null
          }
        ],
        hours: '黑風洞建議清晨前往避開炎熱與人潮，夜市通常越晚越熱鬧，營業至凌晨。',
        notes: '🍢 必吃小吃：福建炒粿條、沙爹、椰漿飯 (Nasi Lemak)、煎蕊 (Cendol)。\n⚠️ 注意事項：參觀清真寺或印度教神廟（如黑風洞）時，請務必穿著保守，避免無袖上衣及短褲短裙。'
      }
    ]
  }
]);

// 選擇國家與城市的狀態管理
const selectedCountry = ref(null);
const selectedCityModal = ref(null);
const selectedAttraction = ref(null);
const selectedTipModal = ref(null);

const openCityModal = (city) => {
  selectedCityModal.value = city;
  selectedAttraction.value = null; // 每次打開新城市，重置選取的景點
};

// 5. 新手生存指南資料 (Accordion Data)
const survivalGuides = ref([
  {
    id: 'tools',
    title: '📱 1. 必備數位工具與網路',
    content: `
      <p><strong>🌐 上網方案大比拼：</strong></p>
      <ul>
        <li><strong>原機漫遊：</strong>免換卡最方便，適合懶人與商務客，但價格通常較高。</li>
        <li><strong>實體 SIM 卡：</strong>價格便宜穩定，但需要手動換卡，且須保管好台灣原本的 SIM 卡。</li>
        <li><strong>eSIM：</strong>掃 QR Code 即可上網，免換卡超級推薦！（購買前須確認手機型號是否支援）</li>
        <li><strong>Wi-Fi 分享器：</strong>適合多人同行平攤費用，但缺點是需要多帶一台機器並隨時充電，且大家不能離機器太遠。</li>
      </ul>
      <p><strong>📲 出國神級 App 清單：</strong></p>
      <ul>
        <li><strong>語言翻譯：</strong>Google 翻譯、Papago（去日韓必備，圖片與語音翻譯超級準確）。</li>
        <li><strong>匯率換算：</strong>極簡匯率（介面乾淨直覺，結帳時按一下就不怕被坑）。</li>
        <li><strong>交通導航：</strong>除了 Google Maps，還推薦當地專屬 App（例如日本的「乘換案內」、歐洲的「Omio」）。</li>
        <li><strong>記帳軟體：</strong>旅行跡、Tricount（特別適合與旅伴共同分母記帳，結算超輕鬆）。</li>
      </ul>
    `
  },
  {
    id: 'money',
    title: '💰 2. 金錢與預算管理',
    content: `
      <p><strong>💵 現金 vs. 信用卡比例：</strong><br>
      新手最常問「要換多少現金？」這完全取決於當地的無現金普及度！例如去日本可以抓 30% 現金 / 70% 刷卡；去韓國甚至 10% 現金就夠了。建議大面額鈔票分開放，錢包只放當天的零用錢即可。</p>
      <p><strong>💳 海外刷卡神卡推薦：</strong><br>
      海外刷卡通常會收 1.5% 手續費，所以一定要準備「海外消費回饋大於 1.5%」的信用卡來抵銷手續費，甚至倒賺！建議 Visa 與 Mastercard 各準備一張以分散風險，出發前也可以先將卡片綁定到 Apple Pay 等手機支付上。</p>
      <p><strong>🥷 防盜防搶小撇步：</strong><br>
      去歐洲或東南亞等扒手熱區，後背包絕對要往前背！貴重物品（大鈔與護照）建議貼身攜帶，且護照正本不要跟所有現金放在同一個錢包裡。在餐廳吃飯，手機千萬不要放在桌上，包包的背帶最好繞過大腿或椅子。</p>
    `
  },
  {
    id: 'airport',
    title: '✈️ 3. 機場交通指南',
    content: `
      <p>很多新手覺得最可怕的不是跑景點，而是「剛下飛機拖著大行李，茫然不知道怎麼去飯店」的那一刻！</p>
      <p><strong>🚆 機場交通概念建立：</strong></p>
      <ul>
        <li><strong>機場快線（鐵路）：</strong>速度最快、時間好掌控（不塞車），但票價較高。</li>
        <li><strong>客運巴士：</strong>票價較便宜、有些路線甚至能直達大型飯店門口，免去搬行李上下樓梯的痛苦，但有塞車風險。</li>
        <li><strong>計程車 / 包車：</strong>費用最高，但如果搭乘紅眼班機、攜帶超多件大行李，或是同行有長輩小孩，這絕對是花錢買開心的最佳解。</li>
      </ul>
      <p><strong>🏨 住宿挑選心法：</strong><br>
      強烈建議新手，「第一天的飯店」盡量選在<strong>機場交通能直達的大車站</strong>附近（例如去東京選上野 / 新宿，去大阪選難波）。這樣一出站就能立刻去飯店放行李，大幅減少拖著 20 吋大行李箱在迷宮般的地鐵裡轉車的崩潰感！</p>
    `
  },
  {
    id: 'emergency',
    title: '🏥 4. 緊急狀況與保險',
    content: `
      <p><strong>🛡️ 旅遊不便險/平安險怎麼買：</strong><br>
      不怕一萬，只怕萬一！用白話文來說，若遇到「班機大延誤」、「行李轉盤上等不到行李」，保險公司會理賠你重買一套衣服與生活用品的費用；如果突發疾病必須在國外看醫生，保險更能幫你省下幾萬塊的醫療帳單。這筆小錢千萬別省！</p>
      <p><strong>🚨 護照遺失處理流程：</strong><br>
      出發前，請務必將「護照影本」、「身分證正反面影本」及「大頭照 2 張」跟護照正本分開存放。萬一在國外遺失護照：<br>
      1. 先去當地警察局報案取得「遺失證明」。<br>
      2. 帶著證明與大頭照，到台灣駐當地使館處申請「入國證明書」即可回台。</p>
      <p><strong>💊 生病急救包：</strong><br>
      國外的成藥不一定適合自己的體質，一定要自己帶常備藥品：腸胃藥、止痛藥、感冒藥、暈車藥與外傷 OK 繃。若有慢性病也記得多帶幾天份的藥量備用。</p>
    `
  }
]);

// 控制 Accordion 展開狀態的變數
const activeGuideId = ref(null);
const stampedIds = ref([]); // 記錄剛關閉的面板，用於觸發蓋章特效

const toggleGuide = (id) => {
  if (activeGuideId.value === id) {
    activeGuideId.value = null; // 收合
    if (!stampedIds.value.includes(id)) {
      stampedIds.value.push(id);
      setTimeout(() => {
        stampedIds.value = stampedIds.value.filter(stampedId => stampedId !== id);
      }, 1000); // 1秒後移除蓋章
    }
  } else {
    activeGuideId.value = id; // 展開
  }
};

// 匯率換算機狀態與資料
const twdAmount = ref(1000);
const exchangeRates = ref({
  JPY: { name: '🇯🇵 日圓 (JPY)', rate: 4.75, symbol: '¥' },
  KRW: { name: '🇰🇷 韓元 (KRW)', rate: 42.5, symbol: '₩' },
  THB: { name: '🇹🇭 泰銖 (THB)', rate: 1.12, symbol: '฿' },
  CNY: { name: '🇨🇳 人民幣 (CNY)', rate: 0.22, symbol: '¥' }
});

// 頁尾動態引言
const footerQuote = computed(() => {
  switch (currentTab.value) {
    case 'home':
      return '「準備好迎接你的第一次自助旅行了嗎？點擊開始規劃，探索屬於你的冒險！」✨';
    case 'itinerary':
      return '「好的行程是成功的一半，留點空白給迷路，有時會有意想不到的驚喜！」✨';
    case 'packing':
      return '「帶上最輕便的行囊，把剩下的空間留給滿滿的回憶與戰利品吧！」🎒';
    case 'cityGuide':
      return '「世界這麼大，總有一個城市的心跳，能與你現在的頻率產生共鳴。」🌍';
    case 'survivalGuide':
      return '「遇到突發狀況先深呼吸，這些都將成為你未來跟朋友炫耀的冒險故事！」💪';
    case 'exchange':
      return '「精打細算不吃虧，省下來的錢剛好拿去多吃一頓好料！」💸';
    default:
      return '';
  }
});
</script>

<template>
  <div class="travel-guide-app">
    <!-- 飛機捲動進度條 -->
    <div class="scroll-progress-container">
      <div class="scroll-fill" :style="{ width: scrollProgress + '%' }">
        <span class="airplane-icon">✈️</span>
      </div>
    </div>

    <!-- 透明遮罩：當選單展開時，點擊選單外部區域即可自動收合 -->
    <div v-if="isMenuOpen" class="menu-overlay" @click="isMenuOpen = false"></div>

    <!-- 頂部漢堡選單 (Hamburger Menu) -->
    <div class="menu-container">
      <button class="hamburger-btn" @click="isMenuOpen = !isMenuOpen">☰ 探索指南</button>
      <Transition name="dropdown">
        <div v-if="isMenuOpen" class="dropdown-menu">
          <button :class="{ active: currentTab === 'itinerary' }" @click="currentTab = 'itinerary'; isMenuOpen = false">🗺️ 行程避坑心法</button>
          <button :class="{ active: currentTab === 'packing' }" @click="currentTab = 'packing'; isMenuOpen = false">🎒 行李打包全攻略</button>
          <button :class="{ active: currentTab === 'cityGuide' }" @click="currentTab = 'cityGuide'; isMenuOpen = false">🏙️ 熱門城市指南</button>
          <button :class="{ active: currentTab === 'survivalGuide' }" @click="currentTab = 'survivalGuide'; isMenuOpen = false">🛡️ 新手生存指南</button>
          <button :class="{ active: currentTab === 'exchange' }" @click="currentTab = 'exchange'; isMenuOpen = false">💱 實用匯率換算機</button>
        </div>
      </Transition>
    </div>

    <!-- 主視覺區塊 (Hero Image) -->
    <header class="hero">
      <div class="hero-overlay"></div>
      <div class="hero-content">
        <svg class="hero-icon" viewBox="0 0 24 24"><path d="M21 16v-2l-8-5V3.5c0-.83-.67-1.5-1.5-1.5S10 2.67 10 3.5V9l-8 5v2l8-2.5V19l-2 1.5V22l3.5-1 3.5 1v-1.5L13 19v-5.5l8 2.5z"/></svg>
        <h1>出發吧！新手村</h1>
        <p class="subtitle">從行程規劃到行李打包，第一次出國自助不出錯</p>
        
        <!-- CTA 行動呼籲按鈕 -->
        <button v-if="currentTab === 'home'" class="cta-btn" @click="currentTab = 'itinerary'">開始規劃旅程 ➔</button>
      </div>
    </header>

    <main>
      <Transition name="slide-fade" mode="out-in">
      <!-- 行程篇：使用 v-if 條件顯示 -->
      <section v-if="currentTab === 'itinerary'" key="itinerary" class="itinerary-section">
        <div class="section-header">
          <svg viewBox="0 0 24 24"><path d="M20.5 3l-.16.03L15 5.1 9 3 3.36 4.9c-.21.07-.36.25-.36.48V20.5c0 .28.22.5.5.5l.16-.03L9 18.9l6 2.1 5.64-1.9c.21-.07.36-.25.36-.48V3.5c0-.28-.22-.5-.5-.5zM15 19l-6-2.11V5l6 2.11V19z"/></svg>
          <h2>排行程避坑心法</h2>
        </div>
        <p class="intro-text">不要再把行程塞滿滿啦！掌握以下原則，讓你的第一次自助旅行輕鬆又好玩：</p>
        
        <div class="cards-container">
          <!-- 行程卡片：使用 v-for 列表渲染 -->
          <div v-for="tip in itineraryTips" :key="tip.id" class="tip-card clickable-card" @click="selectedTipModal = tip">
            <div class="tip-image-wrapper">
              <img :src="tip.image" :alt="tip.title" class="tip-image" />
            </div>
            <div class="tip-content">
              <h3>{{ tip.title }}</h3>
              <p>{{ tip.summary }}</p>
              <div class="read-more">閱讀詳細避坑指南 ➔</div>
            </div>
          </div>
        </div>
        
        <!-- 行程心法詳細資訊 Modal -->
        <Transition name="fade">
          <div v-if="selectedTipModal" class="modal-overlay" @click.self="selectedTipModal = null">
            <div class="modal-content">
              <button class="modal-close-btn" @click="selectedTipModal = null">✕</button>
              <div class="modal-header tip-modal-header" :style="{ backgroundImage: `url(${selectedTipModal.image})` }">
                <div class="overlay"></div>
                <h2 class="modal-title">{{ selectedTipModal.title }}</h2>
              </div>
              <div class="modal-body tip-modal-body">
                <div v-html="selectedTipModal.detail"></div>
              </div>
            </div>
          </div>
        </Transition>
      </section>

      <!-- 行李篇：使用 v-if 條件顯示 -->
      <section v-else-if="currentTab === 'packing'" key="packing" class="packing-section">
        <div class="section-header">
          <svg viewBox="0 0 24 24"><path d="M12 2c-4.42 0-8 3.58-8 8v10c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V10c0-4.42-3.58-8-8-8zm4 11H8v-2h8v2z"/></svg>
          <h2>行李打包互動清單</h2>
        </div>
        <p class="intro-text">帶重點就好！跟著清單檢查，沒帶到的絕對不漏勾。</p>
        
        <!-- 打包進度條 -->
        <div class="progress-bar-container" :class="{ 'is-complete': progress === 100 }">
          <div class="progress-header">
            <p>打包進度：{{ progress }}%</p>
            <div class="action-group">
              <button v-if="progress > 0" class="clear-btn" @click="clearCheckedItems">↺ 清除已勾選</button>
              <button class="clear-btn reset-btn" @click="resetPackingList">⚠️ 還原預設</button>
            </div>
          </div>
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progress + '%' }">
              <span v-if="progress === 100" class="takeoff-airplane">✈️</span>
            </div>
          </div>
        </div>

        <!-- 新增自訂物品 -->
        <div class="add-item">
          <input 
            v-model="newItemName" 
            @keyup.enter="addNewItem"
            placeholder="還有什麼想帶的？例如：拍立得..."
          />
          <button @click="addNewItem">新增</button>
        </div>

        <!-- 互動清單：使用 v-for 渲染並用 v-model 綁定勾選狀態 -->
        <ul class="checklist">
          <li v-for="item in packingList" :key="item.id" :class="{ checked: item.checked }">
            <label>
              <input type="checkbox" v-model="item.checked" />
              <span class="item-name">
                {{ item.name }}
                <span v-if="item.isEssential" class="tag essential">必備</span>
              </span>
            </label>
            <button class="delete-item-btn" @click="deleteItem(item.id)" title="刪除此項目">
              ✕
            </button>
          </li>
        </ul>
      </section>

      <!-- 城市指南篇：使用 v-if 條件顯示 -->
      <section v-else-if="currentTab === 'cityGuide'" key="cityGuide" class="city-guide-section">
        <Transition name="fade" mode="out-in">
        <!-- 第一層：選擇國家 -->
        <div v-if="!selectedCountry" key="country-selection" class="city-selection">
          <div class="section-header center">
            <h2>你想去哪個國家玩？</h2>
          </div>
          <p class="intro-text">點擊下方國家，探索專屬的熱門城市防雷指南：</p>
          
          <div class="cards-container grid-2">
            <!-- 使用 v-for 渲染國家按鈕 -->
            <button 
              v-for="country in countriesData" 
              :key="country.id" 
              class="city-card-btn"
              @click="selectedCountry = country"
            >
              <div class="country-image-wrapper">
                <img :src="country.image" :alt="country.name" class="city-image" />
                <div class="country-name-overlay">
                  <h3>{{ country.name }}</h3>
                </div>
              </div>
            </button>
          </div>
        </div>

        <!-- 第二層：選擇城市 -->
        <div v-else key="city-selection" class="city-selection">
          <div class="top-actions">
            <button class="back-btn-top" @click="selectedCountry = null">
              ⬅️ 返回
            </button>
          </div>
          <div class="section-header center">
            <h2><span class="country-tag">{{ selectedCountry.code }}</span>{{ selectedCountry.name }} 的熱門城市</h2>
          </div>
          <p class="intro-text">選擇準備前往的城市，查看必去景點與注意事項：</p>

          <div class="cards-container grid-3">
            <!-- 使用 v-for 渲染城市按鈕 -->
            <button 
              v-for="city in selectedCountry.cities" 
              :key="city.id" 
              class="city-card-btn ticket-card"
              @click="openCityModal(city)"
            >
              <div class="city-image-wrapper">
                <img :src="city.image" :alt="city.name" class="city-image" />
              </div>
              <div class="city-info">
                <div class="city-text">
                  <h3 class="city-title">{{ city.name }}</h3>
                  <span class="city-subtitle">{{ city.enName }}</span>
                </div>
                <div class="ticket-right">
                  <span class="arrow-icon">➔</span>
                </div>
              </div>
            </button>
          </div>
        </div>
        </Transition>

        <!-- 第三層：城市詳細資訊 Modal -->
        <Transition name="fade">
          <div v-if="selectedCityModal" class="modal-overlay" @click.self="selectedCityModal = null; selectedAttraction = null;">
            <div class="modal-content">
              <button class="modal-close-btn" @click="selectedCityModal = null; selectedAttraction = null;">✕</button>

              <div class="modal-header" :style="{ backgroundImage: `url(${selectedCityModal.image})` }">
                <div class="overlay"></div>
                <div class="boarding-pass-labels">
                  <span>FLIGHT<br><b>{{ selectedCountry.code }}-{{ selectedCityModal.id.slice(0,3).toUpperCase() }}</b></span>
                  <span>CLASS<br><b>ECONOMY</b></span>
                </div>
                <h2 class="modal-title">{{ selectedCityModal.name }} 旅遊防雷包</h2>
              </div>

              <div class="modal-body">
                <h3 class="modal-sec-title">📍 熱門景點</h3>
                <p class="small-hint">💡 點擊下方景點查看詳細攻略</p>
                <div class="attraction-tags">
                  <button 
                    v-for="(attr, index) in selectedCityModal.attractions" 
                    :key="index" 
                    class="tag attraction clickable"
                    :class="{ active: selectedAttraction === attr }"
                    @click="selectedAttraction = (selectedAttraction === attr ? null : attr)"
                  >
                    {{ attr.name }}
                  </button>
                </div>

                <!-- 動態顯示所選景點詳細資訊 -->
                <Transition name="slide-fade">
                  <div v-if="selectedAttraction" class="attraction-detail-card">
                    <h4>📌 {{ selectedAttraction.name }}</h4>
                    <p class="attr-desc" v-if="selectedAttraction.description">{{ selectedAttraction.description }}</p>
                    <ul class="attr-info-list">
                      <li v-if="selectedAttraction.tickets"><strong>🎟️ 門票：</strong>{{ selectedAttraction.tickets }}</li>
                      <li v-if="selectedAttraction.hours"><strong>⏰ 營業時間：</strong>{{ selectedAttraction.hours }}</li>
                      <li v-if="selectedAttraction.transport"><strong>🚆 交通：</strong>{{ selectedAttraction.transport }}</li>
                      <li v-if="selectedAttraction.website">
                        <strong>🔗 官網：</strong>
                        <a :href="selectedAttraction.website" target="_blank" rel="noopener noreferrer">點此前往查看</a>
                      </li>
                    </ul>
                  </div>
                </Transition>

                <h3 class="modal-sec-title">⏰ 營業時間</h3>
                <p class="modal-text">{{ selectedCityModal.hours }}</p>

                <div class="note-box mt-30">
                  <h3>⚠️ 特別注意事項</h3>
                  <p>{{ selectedCityModal.notes }}</p>
                </div>
              </div>
            </div>
          </div>
        </Transition>
      </section>

      <!-- 生存指南篇：使用 v-if 條件顯示 (Accordion 實作) -->
      <section v-else-if="currentTab === 'survivalGuide'" key="survivalGuide" class="survival-guide-section">
        <div class="section-header">
          <svg viewBox="0 0 24 24"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-6h2v6zm0-8h-2V7h2v2z"/></svg>
          <h2>新手生存指南</h2>
        </div>
        <p class="intro-text">遇到突發狀況不慌張！點擊下方標題展開生存守則：</p>
        
        <div class="accordion">
          <div 
            v-for="guide in survivalGuides" 
            :key="guide.id" 
            class="accordion-item"
            :class="{ 'is-open': activeGuideId === guide.id }"
          >
            <button class="accordion-header" @click="toggleGuide(guide.id)">
              <h3>{{ guide.title }}</h3>
              <span class="toggle-icon">
                <svg viewBox="0 0 24 24" class="chevron"><path d="M7.41 8.59L12 13.17l4.59-4.58L18 10l-6 6-6-6 1.41-1.41z"/></svg>
              </span>
            </button>
            <div v-show="activeGuideId === guide.id" class="accordion-body">
              <div v-html="guide.content"></div>
            </div>
            
            <!-- 護照蓋章動畫 -->
            <div v-if="stampedIds.includes(guide.id)" class="stamp-effect">CHECKED</div>
          </div>
        </div>
      </section>

      <!-- 匯率換算機篇：使用 v-else-if 條件顯示 -->
      <section v-else-if="currentTab === 'exchange'" key="exchange" class="exchange-section">
        <div class="section-header">
          <svg viewBox="0 0 24 24"><path d="M11.8 10.9c-2.27-.59-3-1.2-3-2.15 0-1.09 1.01-1.85 2.7-1.85 1.78 0 2.44.85 2.5 2.1h2.21c-.07-1.72-1.12-3.3-3.21-3.81V3h-3v2.16c-1.94.42-3.5 1.68-3.5 3.61 0 2.31 1.91 3.46 4.7 4.13 2.5.6 3 1.48 3 2.41 0 .69-.49 1.79-2.7 1.79-2.06 0-2.87-.92-2.98-2.1h-2.2c.12 2.19 1.76 3.42 3.68 3.83V21h3v-2.15c1.95-.37 3.5-1.5 3.5-3.55 0-2.84-2.43-3.81-4.7-4.4z"/></svg>
          <h2>實用匯率換算機</h2>
        </div>
        <p class="intro-text">輸入新台幣，快速換算日韓泰中熱門旅遊幣值！（💡 匯率為浮動參考值，請以銀行當日牌告為準）</p>

        <div class="exchange-calculator">
          <div class="twd-input-container">
            <label>🇹🇼 新台幣 (TWD)</label>
            <div class="input-wrapper">
              <span class="currency-symbol">NT$</span>
              <input type="number" v-model.number="twdAmount" min="0" placeholder="請輸入金額" />
            </div>
          </div>
          <div class="exchange-results">
            <div v-for="(currency, code) in exchangeRates" :key="code" class="result-card">
              <span class="currency-name">{{ currency.name }}</span>
              <span class="currency-value">{{ currency.symbol }} {{ (twdAmount * currency.rate).toLocaleString('en-US', { maximumFractionDigits: 2 }) }}</span>
            </div>
          </div>
        </div>
      </section>
      </Transition>
    </main>

    <!-- 頁尾結語：呼應希望讀者離開時的感受 -->
    <div class="quote-footer">
      <Transition name="fade" mode="out-in">
        <p :key="currentTab">{{ footerQuote }}</p>
      </Transition>
    </div>

    <footer class="site-footer">
      <p>© 2026 出發吧！新手村. All rights reserved.</p>
    </footer>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;700;900&family=Noto+Sans+TC:wght@400;500;700;900&display=swap');

/* 加入全局極淺灰背景色 */
:global(body) {
  background-color: #f0f4f8; /* 帶有天空藍調的極淺灰 */
  margin: 0;
}

/* 基礎樣式設定，針對年輕族群採用簡潔明亮的風格 */
.travel-guide-app {
  max-width: 850px;
  margin: 0 auto;
  font-family: 'Nunito', 'Noto Sans TC', sans-serif;
  color: #334155;
  padding: 40px 20px;
  position: relative; /* 給下拉選單作為絕對定位的基準點 */
}

/* 沉浸式主視覺 Hero Image */
.hero {
  position: relative;
  height: 350px;
  background-image: url('https://images.unsplash.com/photo-1436491865332-7a61a109cc05?auto=format&fit=crop&w=1200&q=80');
  background-size: cover;
  background-position: center;
  border-radius: 24px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  margin-bottom: 40px;
  box-shadow: 0 10px 30px rgba(14, 165, 233, 0.15);
}
.hero-overlay {
  position: absolute; inset: 0; background: linear-gradient(to top, rgba(15, 23, 42, 0.8), rgba(15, 23, 42, 0.2));
}
.hero-content {
  position: relative; z-index: 1; color: white; padding: 0 20px;
  display: flex; flex-direction: column; align-items: center;
}
.hero-icon { width: 56px; height: 56px; margin-bottom: 16px; fill: #facc15; filter: drop-shadow(0 4px 6px rgba(0,0,0,0.3)); }
.hero h1 { font-size: 2.6em; font-weight: 900; margin: 0 0 12px 0; color: #fff; text-shadow: 0 2px 8px rgba(0,0,0,0.4); letter-spacing: 2px; }
.hero .subtitle {
  color: #e2e8f0; font-size: 1.2em; font-weight: 500; margin: 0;
  text-shadow: 0 2px 4px rgba(0,0,0,0.5);
}

/* 漢堡選單樣式 */
.menu-container {
  position: absolute;
  top: 60px; /* 貼合內距 */
  right: 40px;
  z-index: 100;
}
.hamburger-btn {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(4px);
  color: #0f172a;
  border: none;
  padding: 10px 20px;
  border-radius: 30px;
  font-weight: 800;
  font-size: 1em;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  transition: all 0.3s;
}
.hamburger-btn:hover { background: #fff; transform: translateY(-2px); box-shadow: 0 6px 16px rgba(0,0,0,0.2); }

/* 選單透明背景遮罩，用於點擊外部關閉選單 */
.menu-overlay {
  position: fixed;
  inset: 0;
  z-index: 90; /* 確保在 menu-container (z-index: 100) 下方 */
}

.dropdown-menu {
  position: absolute;
  top: 100%; right: 0;
  margin-top: 10px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.15);
  display: flex; flex-direction: column;
  min-width: 220px; padding: 8px;
}
.dropdown-menu button {
  background: transparent; border: none; padding: 14px 18px;
  text-align: left; font-size: 1.05em; font-weight: 700;
  color: #475569; border-radius: 10px; cursor: pointer; transition: all 0.2s;
}
.dropdown-menu button:hover { background: #f1f5f9; color: #0ea5e9; }
.dropdown-menu button.active { background: #e0f2fe; color: #0284c7; }

/* CTA 導流按鈕 */
.cta-btn {
  margin-top: 25px;
  padding: 14px 32px;
  font-size: 1.15em;
  font-weight: 900;
  color: #78350f;
  background: #facc15;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  box-shadow: 0 6px 20px rgba(250, 204, 21, 0.4);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.cta-btn:hover { background: #fef08a; transform: translateY(-3px) scale(1.05); box-shadow: 0 10px 25px rgba(250, 204, 21, 0.6); }

/* 內容區塊與卡片 */
.section-header { display: flex; align-items: center; gap: 20px; margin-bottom: 20px; }
.section-header svg { width: 100px; height: 100px; fill: #0ea5e9; flex-shrink: 0; filter: drop-shadow(0 4px 6px rgba(14,165,233,0.2)); }
.section-header h2 { margin: 0; display: flex; align-items: center; gap: 10px; }
.country-tag { background: #e2e8f0; color: #475569; font-size: 0.6em; padding: 4px 10px; border-radius: 8px; font-weight: 800; letter-spacing: 1px; }

.intro-text { font-size: 1.1em; margin-bottom: 20px; line-height: 1.6; }
.cards-container { display: grid; gap: 15px; }
.grid-2 { grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); }
.grid-3 { grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); }

/* 卡片 UI 全面升級：純白、圓角、無綠色粗框、柔和陰影 */
.tip-card {
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 6px 16px rgba(0,0,0,0.06);
  overflow: hidden; /* 確保圖片不超出圓角 */
  display: flex;
  flex-direction: column;
  transition: all 0.3s ease;
  border: 2px solid transparent;
}
.clickable-card { cursor: pointer; }
.clickable-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgba(0,0,0,0.12);
  border-color: #e0f2fe;
}
.tip-image-wrapper {
  width: 100%;
  height: 200px;
  overflow: hidden;
}
.tip-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}
.tip-card:hover .tip-image {
  transform: scale(1.03); /* 滑鼠懸停時圖片微微放大 */
}
.tip-content {
  padding: 20px;
}
.tip-card h3 { margin-top: 0; color: #0ea5e9; font-weight: 700; font-size: 1.3em; margin-bottom: 10px; }
.tip-card p { margin: 0; color: #475569; line-height: 1.7; }
.read-more {
  margin-top: 15px;
  font-size: 0.95em;
  font-weight: 700;
  color: #0ea5e9;
  display: flex; align-items: center; gap: 5px;
}

/* 行李打包進度條與清單 */
.progress-bar-container { background: #fff; padding: 20px; border-radius: 16px; margin-bottom: 20px; box-shadow: 0 6px 16px rgba(0,0,0,0.06); }
.progress-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.action-group { display: flex; gap: 10px; }
.progress-bar-container p { font-weight: 700; color: #0ea5e9; margin: 0; }
.clear-btn { background: #f8fafc; color: #64748b; border: 1px solid #e2e8f0; padding: 4px 12px; border-radius: 20px; cursor: pointer; font-size: 0.85em; font-weight: 700; transition: all 0.2s; display: flex; align-items: center; gap: 4px; }
.clear-btn:hover { background: #e2e8f0; color: #334155; transform: translateY(-1px); box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
.reset-btn { color: #ef4444; background: #fff5f5; border-color: #fecaca; }
.reset-btn:hover { background: #fee2e2; color: #b91c1c; }
.progress-bar { width: 100%; height: 12px; background-color: #e2e8f0; border-radius: 6px; }
.progress-fill { height: 100%; background: linear-gradient(90deg, #38bdf8, #0284c7); border-radius: 6px; position: relative; transition: width 0.4s cubic-bezier(0.4, 0, 0.2, 1); }

/* 100% 達成時的閃爍微光與飛機特效 */
.progress-bar-container.is-complete {
  animation: completeGlow 1s ease-out;
}
@keyframes completeGlow {
  0% { box-shadow: 0 6px 16px rgba(0,0,0,0.06), 0 0 0 0 rgba(14, 165, 233, 0.6); }
  50% { box-shadow: 0 6px 16px rgba(0,0,0,0.06), 0 0 15px 6px rgba(14, 165, 233, 0); }
  100% { box-shadow: 0 6px 16px rgba(0,0,0,0.06), 0 0 0 0 rgba(14, 165, 233, 0); }
}

.takeoff-airplane {
  position: absolute;
  right: -15px; 
  top: -18px;
  font-size: 28px;
  line-height: 1;
  pointer-events: none;
  z-index: 50;
  filter: drop-shadow(0 4px 6px rgba(0,0,0,0.3));
  animation: takeOffAnim 1.5s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}
@keyframes takeOffAnim {
  0% { transform: translate(0, 0) scale(0.5); opacity: 0; }
  15% { transform: translate(0, 0) scale(1.2) rotate(-15deg); opacity: 1; }
  40% { transform: translate(15px, -15px) scale(1.2) rotate(-25deg); opacity: 1; }
  100% { transform: translate(300px, -200px) scale(1.5) rotate(-15deg); opacity: 0; }
}

.add-item { display: flex; gap: 10px; margin-bottom: 20px; }
.add-item input { flex-grow: 1; padding: 12px 15px; border: 2px solid #e2e8f0; border-radius: 12px; font-family: inherit; font-size: 1em; transition: border-color 0.3s; }
.add-item input:focus { outline: none; border-color: #0ea5e9; }
.add-item button { padding: 10px 25px; background-color: #0ea5e9; color: white; border: none; border-radius: 12px; cursor: pointer; font-weight: 700; font-size: 1.05em; transition: background 0.3s; }
.add-item button:hover { background-color: #0284c7; }
.checklist { list-style: none; padding: 0; }
.checklist li {
  display: flex; justify-content: space-between; align-items: center;
  background: #fff; margin-bottom: 10px; padding: 15px; border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05); transition: all 0.3s;
}
.checklist li:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0,0,0,0.08);
}
.checklist li.checked { opacity: 0.7; background: #f8fafc; }
.checklist li.checked .item-name { text-decoration: line-through; color: #94a3b8; }
.checklist label { display: flex; align-items: center; cursor: pointer; font-size: 1.1em; font-weight: 500; }
.checklist input[type="checkbox"] { margin-right: 15px; width: 22px; height: 22px; accent-color: #0ea5e9; cursor: pointer; }
.delete-item-btn { background: none; border: none; color: #cbd5e1; font-size: 1.2em; cursor: pointer; padding: 4px 10px; border-radius: 50%; transition: all 0.2s; margin-left: 10px; }
.delete-item-btn:hover { color: #ef4444; background: #fee2e2; }
.tag.essential {
  background-color: #f97316; color: white; font-size: 0.75em; padding: 4px 10px;
  border-radius: 20px; margin-left: 10px; font-weight: 700; letter-spacing: 0.5px;
}

/* 城市指南樣式 */
.city-selection { text-align: center; }
.city-card-btn {
  background: #fff; border: none; padding: 0; border-radius: 16px;
  cursor: pointer; font-size: 1.1em; transition: all 0.3s; width: 100%;
  box-shadow: 0 6px 16px rgba(0,0,0,0.06);
  overflow: hidden;
  text-align: left;
}
.city-card-btn:hover {
  transform: translateY(-4px); 
  box-shadow: 0 12px 24px rgba(0,0,0,0.12);
}
.city-image-wrapper {
  width: 100%;
  height: 160px;
  overflow: hidden;
}
.city-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.4s ease;
}
.city-card-btn:hover .city-image {
  transform: scale(1.03); /* 滑鼠懸停時圖片微微放大 */
}
.city-info { padding: 15px 20px; display: flex; justify-content: space-between; align-items: center; }
.city-text { display: flex; flex-direction: column; text-align: left; }
.city-title { margin: 0; color: #333; font-weight: 800; font-size: 1.3em; }
.city-subtitle { color: #94a3b8; font-size: 0.85em; font-weight: 700; margin-top: 4px; }
.ticket-right { display: flex; align-items: center; gap: 12px; }
.arrow-icon { color: #cbd5e1; font-size: 1.2em; transition: transform 0.3s ease, color 0.3s ease; }
.city-card-btn:hover .arrow-icon { transform: translateX(4px); color: #0ea5e9; }

/* 登機證卡片樣式 (Boarding Pass Card) */
.ticket-card {
  position: relative;
  box-shadow: none !important; /* 取消原生陰影，避免被 mask 切掉 */
  filter: drop-shadow(0 6px 12px rgba(0,0,0,0.06)); /* 改用 drop-shadow 讓陰影符合打洞形狀 */
  /* 加大凹槽半徑到 16px */
  -webkit-mask: 
    radial-gradient(circle at 0 160px, transparent 16px, black 17px) top left / 51% 100% no-repeat,
    radial-gradient(circle at 100% 160px, transparent 16px, black 17px) top right / 51% 100% no-repeat;
  mask:
    radial-gradient(circle at 0 160px, transparent 16px, black 17px) top left / 51% 100% no-repeat,
    radial-gradient(circle at 100% 160px, transparent 16px, black 17px) top right / 51% 100% no-repeat;
}
.city-card-btn.ticket-card:hover {
  filter: drop-shadow(0 12px 24px rgba(14, 165, 233, 0.15));
}
.ticket-card::before {
  content: '';
  position: absolute;
  left: 20px; right: 20px; top: 160px; /* 往內縮對齊凹槽 */
  border-top: 2px dashed #cbd5e1;
  z-index: 1; /* 降低層級，避免蓋住文字 */
  pointer-events: none;
}

/* 國家特殊卡片設計 */
.country-image-wrapper { position: relative; height: 220px; width: 100%; }
.country-name-overlay { position: absolute; inset: 0; background: linear-gradient(to top, rgba(0,0,0,0.7), transparent); display: flex; align-items: flex-end; justify-content: center; padding-bottom: 30px; transition: background 0.3s; }
.country-name-overlay h3 { color: white; font-size: 2.2em; text-shadow: 0 2px 8px rgba(0,0,0,0.6); margin: 0; }
.city-card-btn:hover .country-name-overlay { background: linear-gradient(to top, rgba(0,0,0,0.8), rgba(0,0,0,0.2)); }

/* 第三層：Modal 樣式 */
.modal-overlay {
  position: fixed; inset: 0;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(5px);
  display: flex; justify-content: center; align-items: center;
  z-index: 1000; padding: 20px;
}
.modal-content {
  background: #fff; border-radius: 20px;
  width: 100%; max-width: 550px; max-height: 90vh;
  overflow-y: auto; position: relative;
  box-shadow: none;
  filter: drop-shadow(0 20px 40px rgba(0,0,0,0.3));
  /* Modal 也是大型登機證造型 */
  -webkit-mask: 
    radial-gradient(circle at 0 200px, transparent 20px, black 21px) top left / 51% 100% no-repeat,
    radial-gradient(circle at 100% 200px, transparent 20px, black 21px) top right / 51% 100% no-repeat;
  mask: 
    radial-gradient(circle at 0 200px, transparent 20px, black 21px) top left / 51% 100% no-repeat,
    radial-gradient(circle at 100% 200px, transparent 20px, black 21px) top right / 51% 100% no-repeat;
}
.modal-content::before {
  content: ''; position: absolute;
  left: 25px; right: 25px; top: 200px;
  border-top: 3px dashed #cbd5e1; 
  z-index: 1; pointer-events: none;
}
.modal-close-btn {
  position: absolute; top: 15px; right: 15px;
  width: 36px; height: 36px; border-radius: 50%;
  border: none; background: rgba(255,255,255,0.9);
  color: #334155; font-size: 1.2em; font-weight: bold;
  cursor: pointer; z-index: 10; display: flex; justify-content: center; align-items: center;
  transition: all 0.2s; box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
.modal-close-btn:hover { background: #fff; transform: scale(1.1); color: #f43f5e; }
.modal-header {
  position: relative; height: 200px; background-size: cover; background-position: center;
  display: flex; align-items: flex-end; padding: 25px 30px;
  box-sizing: border-box; /* 加入這個，確保高度固定為 200px，不被 padding 撐開 */
}
.modal-header .overlay { position: absolute; inset: 0; background: linear-gradient(to top, rgba(15,23,42,0.9), transparent); }
.boarding-pass-labels {
  position: absolute; top: 25px; left: 30px; right: 30px;
  display: flex; justify-content: space-between; z-index: 2;
  border-bottom: 1px solid rgba(255,255,255,0.3); padding-bottom: 12px;
}
.boarding-pass-labels span { color: rgba(255,255,255,0.85); font-size: 0.8em; letter-spacing: 1px; line-height: 1.4; }
.boarding-pass-labels b { color: #fff; font-size: 1.6em; display: block; margin-top: 4px; }
.modal-title { position: relative; z-index: 2; margin: 0; color: white; font-size: 2.2em; font-weight: 900; text-shadow: 0 2px 8px rgba(0,0,0,0.4); }
.modal-body { padding: 30px; }

/* 行程心法專屬 Modal 微調 */
.tip-modal-header { height: 180px; }
.tip-modal-body { font-size: 1.1em; line-height: 1.8; color: #334155; }
.tip-modal-body :deep(strong) { color: #0ea5e9; font-size: 1.05em; }
.tip-modal-body :deep(p) { margin: 0 0 20px 0; }
.tip-modal-body :deep(p:last-child) { margin-bottom: 0; }

.modal-sec-title { color: #0ea5e9; font-size: 1.3em; font-weight: 800; margin: 0 0 15px 0; display: flex; align-items: center; gap: 8px; }
.small-hint { color: #64748b; font-size: 0.9em; margin: -10px 0 15px 0; }
.modal-text { font-size: 1.1em; color: #475569; line-height: 1.7; margin: 0; }
.attraction-tags { display: flex; flex-wrap: wrap; gap: 10px; margin-bottom: 20px; }
.tag.attraction { background: #e0f2fe; color: #0369a1; border: none; padding: 8px 16px; border-radius: 20px; font-weight: 700; font-size: 0.95em; transition: all 0.2s; font-family: inherit; }
.tag.attraction.clickable { cursor: pointer; border: 2px solid transparent; }
.tag.attraction.clickable:hover { background: #bae6fd; transform: translateY(-2px); box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.tag.attraction.clickable.active { background: #0284c7; color: white; box-shadow: 0 4px 10px rgba(2, 132, 199, 0.3); }

/* 景點詳細資訊卡片 */
.attraction-detail-card {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 25px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.02);
}
.attraction-detail-card h4 { margin: 0 0 10px 0; color: #0f172a; font-size: 1.15em; font-weight: 800; }
.attr-desc { margin: 0 0 15px 0; color: #475569; line-height: 1.6; }
.attr-info-list { list-style: none; padding: 0; margin: 0; }
.attr-info-list li { margin-bottom: 10px; line-height: 1.6; color: #334155; display: flex; align-items: flex-start; }
.attr-info-list li:last-child { margin-bottom: 0; }
.attr-info-list strong { flex-shrink: 0; margin-right: 5px; color: #0f172a; }
.attr-info-list a { color: #0ea5e9; text-decoration: none; font-weight: 700; }
.attr-info-list a:hover { text-decoration: underline; }

/* 獨立區塊 */
.note-box { background: #fff7ed; padding: 20px 25px; border-left: 6px solid #f59e0b; border-radius: 12px; margin-bottom: 25px; }
.note-box h3 { margin-top: 0; color: #d97706; margin-bottom: 10px; font-weight: 800; }
.note-box p { margin: 0; line-height: 1.7; color: #92400e; }

/* 通用 */
.mt-30 { margin-top: 30px; }
.mt-40 { margin-top: 40px; }
.top-actions { display: flex; justify-content: flex-start; margin-bottom: 20px; }
.back-btn-top {
  background-color: #e2e8f0; color: #334155; border: none; padding: 10px 20px;
  border-radius: 30px; cursor: pointer; font-size: 1.05em; font-weight: 700; transition: all 0.3s;
  display: flex; align-items: center; gap: 6px; box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}
.back-btn-top:hover { background-color: #cbd5e1; transform: translateX(-4px); box-shadow: 0 4px 10px rgba(0,0,0,0.1); }

/* Accordion 生存指南樣式 */
.accordion { display: flex; flex-direction: column; gap: 15px; }
.accordion-item {
  background: #fff;
  border: none;
  position: relative; /* 確保蓋章定位相對於面板 */
  border-radius: 16px;
  box-shadow: 0 6px 16px rgba(0,0,0,0.06);
  transition: all 0.3s ease;
}
.accordion-item:hover {
  box-shadow: 0 10px 20px rgba(0,0,0,0.08);
}
.accordion-item.is-open {
  box-shadow: 0 12px 24px rgba(14, 165, 233, 0.15);
}
.accordion-header {
  width: 100%; display: flex; justify-content: space-between; align-items: center;
  padding: 15px 20px; background: transparent; border: none; cursor: pointer; text-align: left;
}
.accordion-header h3 { margin: 0; color: #0ea5e9; font-size: 1.15em; font-weight: 800; }
.toggle-icon .chevron { width: 24px; height: 24px; fill: #94a3b8; transition: transform 0.3s ease; }
.accordion-item.is-open .toggle-icon .chevron {
  transform: rotate(180deg);
  fill: #0ea5e9;
}
.accordion-body {
  padding: 0 20px 20px;
  color: #475569;
  line-height: 1.7;
  font-size: 1.05em;
}
.accordion-body :deep(p) { margin: 0 0 12px 0; }
.accordion-body :deep(p:last-child) { margin-bottom: 0; }
.accordion-body :deep(ul) { margin: 0 0 15px 0; padding-left: 25px; }
.accordion-body :deep(li) { margin-bottom: 6px; }
.accordion-body :deep(strong) { color: #0ea5e9; font-weight: 800; }

/* 匯率換算機 */
.exchange-calculator { background: #f8fafc; border-radius: 16px; padding: 25px; margin-top: 15px; border: 1px solid #e2e8f0; }
.twd-input-container { display: flex; flex-direction: column; margin-bottom: 20px; }
.twd-input-container label { font-weight: 800; color: #334155; margin-bottom: 8px; }
.input-wrapper { display: flex; align-items: center; background: #fff; border: 2px solid #cbd5e1; border-radius: 12px; overflow: hidden; transition: all 0.3s; }
.input-wrapper:focus-within { border-color: #0ea5e9; box-shadow: 0 0 0 3px rgba(14, 165, 233, 0.15); }
.currency-symbol { padding: 12px 15px; background: #f1f5f9; color: #64748b; font-weight: 800; border-right: 1px solid #cbd5e1; }
.input-wrapper input { flex-grow: 1; border: none; padding: 12px 15px; font-size: 1.15em; font-weight: 800; color: #0f172a; outline: none; }
.exchange-results { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; }
.result-card { background: #fff; padding: 15px 20px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.02); display: flex; flex-direction: column; gap: 5px; border: 1px solid #e2e8f0; }
.currency-name { color: #64748b; font-size: 0.9em; font-weight: 700; }
.currency-value { color: #0ea5e9; font-size: 1.3em; font-weight: 900; }

/* 護照蓋章特效 (Stamp Animation) */
.stamp-effect {
  position: absolute;
  right: 50px; bottom: 40px;
  color: #ef4444; border: 4px solid #ef4444; border-radius: 8px;
  padding: 4px 16px; font-size: 1.8em; font-weight: 900;
  letter-spacing: 3px; font-family: 'Courier New', Courier, monospace;
  pointer-events: none; z-index: 99; opacity: 0;
  animation: stampAnim 1s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
}
@keyframes stampAnim {
  0% { opacity: 0; transform: rotate(-20deg) scale(2.5); }
  15% { opacity: 0.9; transform: rotate(-20deg) scale(0.95); }
  25% { opacity: 1; transform: rotate(-20deg) scale(1); }
  80% { opacity: 1; transform: rotate(-20deg) scale(1); }
  100% { opacity: 0; transform: rotate(-20deg) scale(1); }
}

/* 飛機捲動進度條 */
.scroll-progress-container { position: fixed; top: 0; left: 0; width: 100%; height: 5px; background: repeating-linear-gradient(to right, transparent, transparent 5px, #e2e8f0 5px, #e2e8f0 10px); z-index: 9999; }
.scroll-fill { height: 100%; background-color: #0ea5e9; position: relative; }
.airplane-icon { position: absolute; right: -15px; top: -14px; font-size: 22px; transform: rotate(45deg); line-height: 1; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.2)); }

/* 頁尾結語 */
.quote-footer {
  margin-top: 50px; text-align: center; color: #475569;
  padding: 30px; background: #fff; border-radius: 24px;
  font-size: 1.1em; font-style: normal; font-weight: 700; box-shadow: 0 6px 16px rgba(0,0,0,0.04);
}

.site-footer {
  margin-top: 30px; text-align: center; color: #64748b;
  padding: 20px; background: #e2e8f0; border-radius: 16px;
  font-size: 0.9em; font-weight: 500;
}

/* Vue 過場動畫 (Transitions) */
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease-out;
}
.slide-fade-enter-from {
  opacity: 0;
  transform: translateX(20px);
}
.slide-fade-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Dropdown 下拉動畫 */
.dropdown-enter-active, .dropdown-leave-active { transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1); transform-origin: top right; }
.dropdown-enter-from, .dropdown-leave-to { opacity: 0; transform: scale(0.95) translateY(-10px); }

/* Vue TransitionGroup 清單動畫 */
.list-enter-active,
.list-leave-active { transition: all 0.4s ease; }
.list-enter-from,
.list-leave-to { opacity: 0; transform: translateX(30px); }
.list-leave-active { position: absolute; width: calc(100% - 20px); }
</style>