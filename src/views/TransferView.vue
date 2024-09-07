<script>
import colors from '../../public/MRTinfo/mrtColors.json';
import '../css/mrtStyles.css'; // 引入你的樣式檔案
import stationsData from '../../public/MRTinfo/stations.json';

// const getButtonColor = (lineid) => {
//   return colors[lineid] || '#000000';
// };

export default {
  data() {
    return {
      stationName: '',
      trackInfo: [],
      errorMessage: ''
    };
  },
  computed: {
    endStation() {
      return this.$route.query.endStation;
    },
    endStationLine() {
      return this.$route.query.endStationLine;
    }
  },

  methods: {
    getInfo() {
      console.log("Station: ", this.endStation)
      const stationNameEncoded = encodeURIComponent(this.endStation);
      fetch(`http://localhost:3000/track-info?stationName=${stationNameEncoded}`)
        .then((response) => response.json())
        .then((data) => {
          if (data.error) {
            this.errorMessage = data.error;
            this.trackInfo = [];
          } else {
            this.trackInfo = data;
            this.errorMessage = '';
            
            const targetLineId = this.endStationLine; // 目標線路 ID，例如 "BR"
            const targetLine = stationsData.find(line => line.lineid === targetLineId);
            
            const targetStations = targetLine ? new Set(targetLine.stations.map(station => station[2])) : new Set();
            console.log("targetStations: ", targetStations)
            // 過濾掉不在目標線路中的站點
            this.trackInfo = data.filter(item => !targetStations.has(item.DestinationName));
          }
        })
        .catch((error) => {
          this.errorMessage = 'Error fetching data: ' + error;
        });
    },
    findLines() {
      if (this.endStation === '') {
        this.lines = [];
        return;
      }

      // 清空現有結果
      this.lines = [];

      // 遍歷所有路線並查找車站名稱
      stationsData.forEach((line) => {
        const stationFound = line.stations.find(
          (station) => station[2] === this.endStation
        );
        if (stationFound && line.lineid !== this.endStationLine) {
          this.lines.push({
            lineid: line.lineid,
            lineName: line.lineName
          });
        }
      });

      // 如果找不到車站，顯示無結果
      if (this.lines.length === 0) {
        console.log('No lines found for station:', this.stationName);
      }
      else {
        console.log('Lines found for station:', this.lines);
      }
    },
    getButtonColor(lineId) {
      return colors[lineId] || '#000000'; // 如果找不到顏色，預設為黑色
    },
  },
  mounted() {
    this.findLines();
    this.getInfo();
    
    // 設置定時器，每 30 秒調用 getInfo 方法
    this.intervalId = setInterval(() => {
      this.getInfo();
    }, 10000); // 10 秒更新一次
  },
  beforeDestroy() {
    // 清除定時器
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  },
  beforeRouteLeave(to, from, next) {
    // 清除定時器，確保在頁面離開時停止
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
    next();
  }
};
</script>

<template>
  <div class="py-4">
    <p class="text-grey-500 mt-4 mb-2 px-4">{{ endStation }} 轉乘資訊</p>
    <!-- <p>{{ endStation }}</p> -->
    <div class="favorites-container">
        <div v-for="(item, index) in trackInfo" :key="index" class="favorite-route">
          <div class="favorite-content">
            <div class="color-block" :style="{ backgroundColor: getButtonColor(lines[0]?.lineid) }"></div>
            <p>往 {{ item.DestinationName }}</p>
            <div class="time-info">
              <template v-if="item.CountDown.includes(':')">
                {{ item.CountDown }} 後到站
              </template>
              <template v-else-if="item.CountDown == '營運時間已過'">
                營運時間已過
              </template>
              <template v-else>
                <span class="warning-text">即將進站</span>
              </template>
            </div> 
          </div>
        </div>
        <!-- <div class="favorites-container">
          <div v-for="(route, index) in favoriteRoutes" :key="index" class="favorite-route">
            <div class="favorite-content">
              <div class="color-block" :style="{ backgroundColor: getButtonColor(route.line?.lineid)}">
              </div>
              <p>{{ route.start?.name }} - {{ route.end?.name }}</p>
              <button @click="removeFavoriteRoute(index)" class="delete-favorite-button">
                <img src="../assets/images/delete.svg" alt="Delete" class="icon" />
              </button>
              <button @click="setFavoriteTimer(route)" class="set-timer-button">
                <img src="../assets/images/alarm.svg" alt="Alarm" class="icon" />
              </button>
            </div>
          </div>
        </div> -->
      </div>
      
      <!-- <div v-for="(route, index) in favoriteRoutes" :key="index" class="favorite-route">
        <div class="favorite-content">
          <div class="color-block" :style="{ backgroundColor: getButtonColor(route.line?.lineid)}">
          </div>
          <p>{{ route.start?.name }} - {{ route.end?.name }}</p>
          <button @click="removeFavoriteRoute(index)" class="delete-favorite-button">
            <img src="../assets/images/delete.svg" alt="Delete" class="icon" />
          </button>
          <button @click="setFavoriteTimer(route)" class="set-timer-button">
            <img src="../assets/images/alarm.svg" alt="Alarm" class="icon" />
          </button>
        </div>
      </div> -->

  </div>
</template> 

<style scoped>
  .error {
    color: red;
  }
  .warning-text {
    color: red;
  }
</style>