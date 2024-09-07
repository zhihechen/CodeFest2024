<script setup lang="ts">
import { computed, ref, onMounted } from 'vue';
import { RouterLink, useRoute } from 'vue-router';
import { useFormStore } from '@/stores/form';
import { useUserStore } from '@/stores/user';
import { storeToRefs } from 'pinia';
import { useConnectionMessage } from '@/composables/useConnectionMessage';
import { useHandleConnectionData } from '@/composables/useHandleConnectionData';
import ServiceTabsView from '@/components/organisms/ServiceTabsView.vue';
import BaseInput from '@/components/atoms/BaseInput.vue';
import MRTroutes from '../../public/MRTinfo/stations.json';
import runningTime from '../../public/MRTinfo/running_time.json';
import type { User } from '@/stores/user';

const store = useFormStore();

store.reset();

const userStore = useUserStore();

const { user } = storeToRefs(userStore);

const handleUserInfo = (event: { data: string }) => {
  const result: { name: string; data: User } = JSON.parse(event.data);

  user.value = result.data;
};

useConnectionMessage('userinfo', null);

useHandleConnectionData(handleUserInfo);

const route = useRoute();

const activeTab = ref(0);

if (route.query.isSearch) {
  activeTab.value = 1;
}

import colors from '../../public/MRTinfo/mrtColors.json';

const calculateTravelTime = (start: Station | null, end: Station | null): number | null => {
  if (!start || !end || !selectedLine.value) {
    return null;
  }
  var stationsBetween;
  if (selectedDirection.value === 'forward') {
    stationsBetween = getStationsBetween(start, end);
  } else {
    stationsBetween = getStationsBetween(end, start);
  }
  console.log('stationsBetween: ', stationsBetween);
  let totalTime = 0;

  for (let i = 0; i < stationsBetween.length - 1; i++) {
    const currentStation = stationsBetween[i];
    const nextStation = stationsBetween[i + 1];

    // 查找當前站與下一站之間的行車時間
    const travelTimeEntry = runningTime.find((time) =>
      time.TravelTimes.some(
        (travelTime) =>
          (travelTime.FromStationID === currentStation.code &&
            travelTime.ToStationID === nextStation.code) ||
          (travelTime.FromStationID === nextStation.code &&
            travelTime.ToStationID === currentStation.code)
      )
    );

    if (travelTimeEntry) {
      const matchedTime = travelTimeEntry.TravelTimes.find(
        (travelTime) =>
          (travelTime.FromStationID === currentStation.code &&
            travelTime.ToStationID === nextStation.code) ||
          (travelTime.FromStationID === nextStation.code &&
            travelTime.ToStationID === currentStation.code)
      );

      if (matchedTime) {
        totalTime += matchedTime.RunTime;
        if (selectedDirection.value === 'forward' && i < stationsBetween.length - 2) {
          totalTime += matchedTime.StopTime;
        } else if (selectedDirection.value === 'backward' && i > 0) {
          totalTime += matchedTime.StopTime;
        }
      }
    }
  }

  return totalTime;
};

interface Station {
  id: number;
  code: string;
  name: string;
  line: string;
}

interface MRTLine {
  lineid: string;
  lineName: string;
  stations: Station[];
}

const favoriteRoutes = ref<{ start: Station | null; end: Station | null; line: MRTLine | null }[]>(
  []
);

const loadFavoriteRoutes = () => {
  favoriteRoutes.value = store.favoriteRoutes || [];
};
loadFavoriteRoutes();

const addFavoriteRoute = () => {
  if (startStation.value && endStation.value && selectedLine.value) {
    // Ensure line is selected
    favoriteRoutes.value.push({
      start: startStation.value,
      end: endStation.value,
      line: selectedLine.value // Add the line value
    });
    store.favoriteRoutes = [...favoriteRoutes.value]; // Save to store
    startStation.value = null;
    endStation.value = null;
    selectedLine.value = null; // Reset the line selection after adding
  }
};

const removeFavoriteRoute = (index: number) => {
  favoriteRoutes.value.splice(index, 1);
};

const handleTabChange = (index: number) => {
  activeTab.value = index;
};

// 初始化 selectedLine，給它一個合理的初始值（例如空的 MRTLine）
// const selectedLine: Ref<MRTLine | null> = ref(null);
const selectedLine = ref<MRTLine | null>(null);

const selectedStationType = ref<'start' | 'end'>('start');
const startStation = ref<Station | null>(null);
const endStation = ref<Station | null>(null);

// 監聽起點站和終點站的變化並計算行車時間
const travelTime = computed(() => {
  return calculateTravelTime(startStation.value, endStation.value);
});

const travelTimeFormatted = computed(() => {
  const timeInSeconds = calculateTravelTime(startStation.value, endStation.value);

  if (timeInSeconds === null) {
    return null;
  }

  const minutes = Math.floor(timeInSeconds / 60);
  const seconds = timeInSeconds % 60;

  return `${minutes}分${seconds}秒`;
});

const onStationSelected = (station: Station) => {
  if (selectedStationType.value === 'start') {
    startStation.value = station;
  } else if (selectedStationType.value === 'end') {
    endStation.value = station;
  }

  if (startStation.value && endStation.value) {
    console.log('行車時間:', travelTimeFormatted.value);
  }
};

const convertedMRTroutes = MRTroutes.map((line) => ({
  ...line,
  stations: line.stations.map((station) => ({
    id: station[0] as number,
    code: station[1] as string,
    name: station[2] as string,
    line: station[3] as string
  }))
}));

const getStationsBetween = (start: Station | null, end: Station | null): Station[] => {
  if (!start || !end || !selectedLine.value) {
    return [];
  }

  const stations = selectedLine.value.stations;
  const startIndex = stations.findIndex((station) => station.code === start.code);
  const endIndex = stations.findIndex((station) => station.code === end.code);

  if (startIndex === -1 || endIndex === -1 || startIndex > endIndex) {
    return [];
  }

  return stations.slice(startIndex, endIndex + 1);
};

const onRouteSelect = (line: MRTLine) => {
  selectedLine.value = line;
  console.log('Selected line:', line); // 檢查這裡是否有正確的路線資料
  console.log('Selected line stations:', line.stations); // 檢查站點資料
};

const getButtonColor = (lineid: string) => {
  return colors[lineid as keyof typeof colors] || '#000000';
};

const setFavoriteTimer = (route: { start: Station | null; end: Station | null }) => {
  if (route.start && route.end) {
    startStation.value = route.start;
    endStation.value = route.end;
    startCountdown(); // Start the countdown when a favorite route is selected
  }
};

const showAlert = ref(false);
const countdown = ref(10); // 15分鐘的倒數計時（900秒）

const notifyUser = () => {
  // 看權限 電腦會通知 手機好像要越
  if (Notification.permission === 'granted') {
    showNotification();
  } else if (Notification.permission !== 'denied') {
    Notification.requestPermission().then((permission) => {
      if (permission === 'granted') {
        showNotification();
      }
    });
  }
};

const showNotification = () => {
  // 通知長怎樣
  new Notification('Notification', {
    body: "It's time to get off !!!!!",
    icon: 'https://via.placeholder.com/100' // 選圖
  });
};

const startCountdown = () => {
  showAlert.value = false;
  const interval = setInterval(() => {
    countdown.value--;
    if (countdown.value <= 0) {
      // Play sound
      const audio = new Audio('/440.mp3');
      audio.play();
      clearInterval(interval);
      alert('時間到！');
      notifyUser();
      // 讓手機震動 500 毫秒
      if (navigator.vibrate) {
        navigator.vibrate(500);
        alert('震動');
      } else {
        alert('沒震動');
      }
    }
  }, 1000);
};

// 在 tab2 的 JS 開始處，新增一個方向選擇變數
const selectedDirection = ref<'forward' | 'backward'>('forward');

// 根據方向選擇顯示不同的站點
const getStationsForDirection = (line: MRTLine, direction: 'forward' | 'backward') => {
  return direction === 'forward' ? line.stations : [...line.stations].reverse();
};

const getFirstAndLastStation = computed(() => {
  if (!selectedLine.value) {
    return { first: null, last: null };
  }

  const stations = getStationsForDirection(selectedLine.value, selectedDirection.value);

  return {
    first: stations[0] || null,
    last: stations[stations.length - 1] || null
  };
});
</script>

<template>
  <main>
    <ServiceTabsView v-model="activeTab">
      <template #tab0>
        <div class="py-4">
          <section class="flex items-center px-4">
            <BaseInput v-model="searchValue" placeholder="搜尋捷運站" class="flex-grow" />
            <button class="search-button" @click="onSearchClick">
              <img src="@/assets/images/search-icon.svg" alt="搜尋" />
            </button>
          </section>
          <p class="text-grey-500 mt-4 mb-2 px-4">請選擇路線</p>
          <div class="route-buttons-container">
            <button
              v-for="line in convertedMRTroutes"
              :key="line.lineid"
              class="route-button"
              @click="onRouteSelect(line)"
              :style="{ borderColor: getButtonColor(line.lineid), color: 'black' }"
            >
              {{ line.lineName }}
            </button>
          </div>
        </div>

        <!-- 動態顯示選取路線的所有站點 -->
        <div v-if="selectedLine" class="stations-buttons-container">
          <!-- 選擇方向 -->
          <div class="flex items-center justify-center my-2">
            <label
              class="radio-label"
              data-value="forward"
              :class="{ 'direction-active': selectedDirection === 'forward' }"
            >
              <input type="radio" v-model="selectedDirection" value="forward" />
              {{
                selectedLine.stations[0].name +
                ' -> ' +
                selectedLine.stations[selectedLine.stations.length - 1].name
              }}
            </label>
            <label
              class="radio-label"
              data-value="backward"
              :class="{ 'direction-active': selectedDirection === 'backward' }"
            >
              <input type="radio" v-model="selectedDirection" value="backward" />
              {{
                selectedLine.stations[selectedLine.stations.length - 1].name +
                ' -> ' +
                selectedLine.stations[0].name
              }}
            </label>
          </div>
          <div class="flex items-center justify-center my-2">
            <label class="radio-label" :class="{ 'start-active': selectedStationType === 'start' }">
              <input type="radio" v-model="selectedStationType" value="start" />
              選取起點站
            </label>
            <label class="radio-label" :class="{ 'end-active': selectedStationType === 'end' }">
              <input type="radio" v-model="selectedStationType" value="end" />
              選取終點站
            </label>
          </div>

          <!-- 顯示選取的起點和終點 -->
          <p class="text-grey-500 mt-4 mb-2 px-4">{{ selectedLine.lineName }}的站點</p>
          <div class="stations-buttons">
            <button
              v-for="station in getStationsForDirection(selectedLine, selectedDirection)"
              :key="station.code"
              class="station-button"
              @click="onStationSelected(station)"
              :class="{
                'start-selected': startStation?.code === station.code,
                'end-selected': endStation?.code === station.code,
                'middle-station': getStationsBetween(startStation, endStation).some(
                  (s) => s.code === station.code
                )
              }"
              :style="{ borderColor: getButtonColor(station.line), color: 'black' }"
            >
              {{ station.code + ' ' + station.name }}
            </button>
          </div>

          <div class="button-container">
            <button
              v-show="startStation && endStation"
              @click="addFavoriteRoute"
              class="add-favorite-button"
            >
              加入收藏
            </button>
            <button
              v-show="startStation && endStation"
              @click="startCountdown"
              class="set-alarm-button"
            >
              設定鬧鐘
            </button>
          </div>

          <!-- 顯示倒數計時 -->
          <div v-if="countdown < 900">
            <p>鬧鐘將在 {{ Math.floor(countdown / 60) }} 分 {{ countdown % 60 }} 秒後響起。</p>
          </div>
        </div>
      </template>

      <template #tab1>
        <div class="py-4">
          <p class="text-grey-500 mt-4 mb-2 px-4">我的收藏路線</p>
          <div class="favorites-container">
            <div v-for="(route, index) in favoriteRoutes" :key="index" class="favorite-route">
              <p>{{ route.start?.name }} - {{ route.end?.name }}</p>
              <button @click="removeFavoriteRoute(index)" class="delete-favorite-button">
                刪除
              </button>
              <button @click="setFavoriteTimer(route)" class="set-timer-button">設定鬧鐘</button>
            </div>
          </div>

          <!-- Add countdown display here -->
          <div v-if="countdown.value > 0" class="countdown-display">
            鬧鐘將在 {{ Math.floor(countdown.value / 60) }} 分 {{ countdown.value % 60 }} 秒後響起。
          </div>
        </div>
      </template>
    </ServiceTabsView>
  </main>
</template>

<style lang="postcss">
.search-button {
  @apply bg-primary-500 p-1 rounded-lg;
  @apply h-11 w-11;
  @apply flex justify-center items-center;
  @apply -translate-x-1;
}
.route-buttons-container {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  padding: 0.9rem;
}
.route-button {
  @apply text-black text-xs p-2 rounded-lg;
  flex: 1 1 calc(25% - 0.5rem);
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  border-radius: 9999px;
  background-color: transparent;
  border: 2px solid;
  color: black;
}

.option-title {
  @apply relative;
  @apply before:content-[''];
  @apply before:w-1.5 before:h-0.5;
  @apply before:bg-primary-500;
  @apply before:inline-block;
  @apply before:absolute before:-left-3.5 before:top-1/2 before:-translate-y-1/2;
}

.situation-button {
  @apply text-primary-500;
  @apply first:rounded-l last:rounded-r;
  @apply border border-primary-500;
  @apply py-0.5;

  &--active {
    @apply bg-primary-500 text-white;
  }
}

.radio-label-container {
  margin-top: 2px; /* 調整這個值讓元素靠近上面 */
}

.radio-label {
  display: inline-block;
  padding: 6px 12px; /* 減少內部填充 */
  margin: 0 5px; /* 減少外部邊距 */
  border: 2px solid transparent;
  border-radius: 15px; /* 減少圓角半徑 */
  font-size: 12px; /* 調整字體大小 */
  cursor: pointer;
  transition: background-color 0.3s ease; /* 可以調整或刪除過渡效果 */
}

.radio-label input {
  display: none;
}

.radio-label.start-active {
  border-color: green;
  background-color: rgba(0, 255, 0, 0.2); /* 淺綠色 */
  transition: background-color 0.3s ease; /* 可以調整或刪除過渡效果 */
}

.radio-label.end-active {
  border-color: red;
  background-color: rgba(255, 0, 0, 0.2); /* 淺紅色 */
  transition: background-color 0.3s ease; /* 可以調整或刪除過渡效果 */
}

.route-buttons-container {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem; /* 按鈕之間的間距 */
  padding: 0.9rem; /* 與畫面邊界的空隙 */
}

.route-button {
  @apply text-black text-xs p-2 rounded-lg;
  flex: 1 1 calc(25% - 0.5rem);
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  border-radius: 9999px; /* 圓角 */
  background-color: transparent; /* 透明背景 */
  border: 2px solid; /* 邊框 */
  color: black; /* 文字顏色 */
}

.stations-buttons-container {
  margin-top: 1rem;
}

.stations-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem; /* 按鈕之間的間距 */
  padding: 0.9rem; /* 與畫面邊界的空隙 */
}

.station-button {
  @apply text-black text-xs p-2 rounded-lg;
  flex: 1 1 auto;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  border-radius: 9999px; /* 圓角 */
  background-color: transparent; /* 透明背景 */
  border: 2px solid; /* 邊框 */
  color: black; /* 文字顏色 */
}

/* 起點站按鈕樣式 */
.station-button.start-selected {
  background-color: rgba(0, 255, 0, 0.3); /* 半透明綠色 */
  border-color: green !important;
  color: black;
  transition: background-color 0.3s ease;
}

/* 終點站按鈕樣式 */
.station-button.end-selected {
  background-color: rgba(255, 0, 0, 0.3); /* 半透明紅色 */
  border-color: red !important;
  color: black;
  transition: background-color 0.3s ease;
}

.middle-station {
  background-color: rgba(128, 128, 128, 0.3); /* 半透明灰色 */
  border-color: grey;
  color: black;
  transition: background-color 0.3s ease;
}

.buttons-group {
  display: flex;
  gap: 1rem;
  margin-top: 1rem;
}

.favorites-container {
  padding: 1rem;
}

/* 按鈕選取時樣式 */
.radio-label.direction-active {
  border-color: #2db6c7;
  background-color: #2db6c7;
  color: white;
}

.favorite-route {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px;
  background-color: #f9f9f9;
  margin-bottom: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.favorite-route button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
.favorite-route p {
  @apply flex-grow;
}
.delete-favorite-button {
  @apply bg-red-500 p-1 rounded-lg text-white;
  cursor: pointer;
  border: none;
}
.delete-favorite-button:hover {
  background-color: #ff7875;
}
.button-group {
  display: flex;
  gap: 1rem;
  align-items: center;
}
.set-timer-button {
  background-color: #1890ff;
  color: white;
}
.favorite-route button:last-child {
  margin-left: 0;
}

.favorite-route button:last-child {
  margin-left: 0;
}
.set-timer-button:hover {
  background-color: #40a9ff;
}
.add-favorite-button {
  @apply bg-primary-500 p-2 rounded-lg text-white;
  margin-top: 1rem;
  cursor: pointer;
  border: none;
  display: block;
}
.set-alarm-button {
  @apply bg-secondary-500 p-2 rounded-lg text-white;
  cursor: pointer;
  border: none;
  flex: 1;
}
.delete-favorite-button {
  background-color: #ff4d4f;
  color: white;
}

/* Countdown Display */
.countdown-display {
  margin-top: 1rem;
  padding: 0.5rem;
  background-color: #f0f0f0;
  border-radius: 8px;
  text-align: center;
  font-weight: bold;
}
</style>