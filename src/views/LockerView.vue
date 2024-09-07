<template>
    <div>
      <h1>置物櫃查詢</h1>
      <!-- <form @submit.prevent="getLockerInfo">
        <input type="text" v-model="stationName" placeholder="請輸入站名" required />
        <button type="submit">取得資訊</button>
      </form> -->
  
      <div id="result">
        <table v-if="lockerData.length > 0" id="resultTable">
          <thead>
            <tr>
              <th>站名</th>
              <th>置物櫃大小</th>
              <th>位置</th>
              <th>費率</th>
              <th>剩餘可用數目</th>
              <th>總數目</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in lockerData" :key="item.StationName + item.Size">
              <td>{{ item.StationName }}</td>
              <td>{{ item.Size }}</td>
              <td>{{ item.LockerDescription }}</td>
              <td>{{ item.payment }}</td>
              <td>{{ item.QuantityAvailable }}</td>
              <td>{{ item.Total }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        stationName: '',
        lockerData: []
      };
    },
    methods: {
      async getLockerInfo() {
        try {
          this.stationName = this.$route.query.endStation;;
          const response = await fetch(`http://localhost:3001/track-info?stationName=${encodeURIComponent(this.stationName)}`);
          const data = await response.json();
          
          // Extract JSON data (assuming it's before the XML response)
          const jsonData = JSON.parse(data.split('<?xml')[0]);
  
          this.lockerData = jsonData;
        } catch (error) {
          console.error(`Error: ${error.message}`);
        }
      }
    },
    mounted() {
      // 在組件掛載後呼叫 getLockerInfo 方法
      this.getLockerInfo();
      console.log(this.lockerData)
    }
  };
  </script>
  
  <style scoped>
  /* 全局樣式重置 */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f0f4f8;
    color: #333;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    padding: 20px;
  }
  h1 {
    text-align: center;
    font-size: 2.5em;
    color: #333;
    margin-bottom: 30px;
  }
  form {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    margin-bottom: 30px;
  }
  input[type="text"] {
    padding: 12px 20px;
    font-size: 16px;
    width: 300px;
    border: 2px solid #ddd;
    border-radius: 25px;
    outline: none;
    transition: border 0.3s ease, box-shadow 0.3s ease;
  }
  input[type="text"]:focus {
    border-color: #2db6c7;
    box-shadow: 0 0 5px rgba(108, 99, 255, 0.5);
  }
  button {
    padding: 12px 20px;
    font-size: 16px;
    background-color: #2db6c7;
    color: #fff;
    border: none;
    border-radius: 25px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }
  button:hover {
    background-color: #2db6c7;
    transform: translateY(-2px);
  }
  #result {
    display: flex;
    justify-content: center; /* 水平置中 */
    align-items: center;     /* 垂直置中，若有需要可以保留 */
    width: 100%;
    max-width: 900px;
    margin: 20px auto;
    overflow-x: auto;
  }
  
  table {
    width: 100%;
    border-collapse: collapse;
    background-color: #fff;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transition: all 0.3s ease;
  }
  th, td {
    padding: 15px;
    text-align: left;
  }
  th {
    background-color: #2db6c7;
    color: #fff;
    font-weight: bold;
    text-transform: uppercase;
  }
  td {
    border-bottom: 1px solid #ddd;
    color: #555;
  }
  tr:hover {
    background-color: #f9f9f9;
  }
  </style>
  