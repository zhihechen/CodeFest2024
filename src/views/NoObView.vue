<template>
    <div class="container">
      <h1 class="title">🚉 捷運站無障礙資訊查詢</h1>
      <div class="search-container">
        <input
          v-model="stationName"
          placeholder="輸入車站名稱"
          class="search-input"
        />
        <button @click="fetchStationInfo" class="search-button">查詢</button>
      </div>
  
      <div v-if="stationInfo" class="info-card">
        <h2 class="station-title">{{ stationInfo.車站名稱 }}無障礙資訊</h2>
        <p><strong>🚪 出口電梯/無障礙坡道位置：</strong>{{ stationInfo['出口電梯/無障礙坡道位置'] }}</p>
        <p><strong>🛂 無障礙閘門位置：</strong>{{ stationInfo.無障礙閘門位置 }}</p>
        <p><strong>🚻 無障礙廁所位置：</strong>{{ stationInfo.無障礙廁所位置 }}</p>
        <p><strong>🚋 列車門開方向：</strong>{{ stationInfo.列車門開方向 }}</p>
        <p><strong>🦽 車廂輪椅優先停靠區：</strong>{{ stationInfo.車廂輪椅優先停靠區 }}</p>
      </div>
  
      <div v-if="errorMessage" class="error-message">
        <p>{{ errorMessage }}</p>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        stationName: '',
        stationData: null,
        stationInfo: null,
        errorMessage: ''
      };
    },
    methods: {
      async fetchStationInfo() {
        this.stationInfo = null;
        this.errorMessage = '';
  
        if (!this.stationName.trim()) {
          this.errorMessage = '請輸入有效的車站名稱';
          return;
        }
  
        try {
          const response = await fetch('/station_data.json');
          if (!response.ok) {
            throw new Error('Failed to fetch station data');
          }
          this.stationData = await response.json();
  
          const station = this.stationData.find(
            (item) => item.車站名稱 === this.stationName.trim()
          );
  
          if (station) {
            this.stationInfo = station;
          } else {
            this.errorMessage = '找不到該車站資訊，請重新輸入。';
          }
        } catch (error) {
          console.error('Error fetching station data:', error);
          this.errorMessage = '無法讀取資料，請稍後再試。';
        }
      }
    }
  };
  </script>
  
  <style scoped>
  /* 設置主要色彩與整體風格 */
  .container {
    max-width: 600px;
    margin: 50px auto;
    padding: 20px;
    background-color: #f5f5f5;
    border-radius: 10px;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
    font-family: 'Arial', sans-serif;
  }
  
  .title {
    font-size: 1.8rem;
    text-align: center;
    margin-bottom: 20px;
    color: #2db6c7;
  }
  
  .search-container {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
  }
  
  .search-input {
    flex: 1;
    padding: 10px;
    border: 2px solid #2db6c7;
    border-radius: 5px;
    font-size: 1rem;
    margin-right: 10px;
  }
  
  .search-button {
    padding: 10px 20px;
    background-color: #2db6c7;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.3s;
  }
  
  .search-button:hover {
    background-color: #2db6c7;
  }
  
  .info-card {
    padding: 20px;
    background-color: white;
    border-radius: 10px;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s;
  }
  
  .info-card:hover {
    transform: translateY(-5px);
  }
  
  .station-title {
    font-size: 1.5rem;
    margin-bottom: 10px;
    color: #2db6c7;
  }
  
  p {
    font-size: 1.1rem;
    line-height: 1.5;
    margin: 10px 0;
  }
  
  .error-message p {
    color: red;
    text-align: center;
    font-weight: bold;
  }
  </style>
  