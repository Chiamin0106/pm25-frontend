<template>
  <div class="container">
    <header>
      <h1>PM2.5 環境監測管理系統</h1>
    </header>

    <section class="card input-panel">
      <h3> 數據管理面板</h3>
      <div class="input-grid">  
        <div class="input-group">
          <label>測站名稱</label>
          <input v-model="form.sitename" type="text" placeholder="例如: 北科大" />
        </div>
        <div class="input-group">
          <label>監測月份</label>
          <input v-model="form.monitormonth" type="number" placeholder="202512" />
        </div>
        <div class="input-group">
          <label>PM2.5 濃度</label>
          <input v-model="form.concentration" type="number" step="0.1" />
        </div>
      </div>

      <div class="action-buttons">
        <button @click="handlePost" class="btn btn-post">Q6: 新增資料</button>
        <button @click="handlePut" class="btn btn-put">Q7: 修改濃度</button>
        <button @click="handleDelete" class="btn btn-del">Q8: 刪除站點</button>
        <button @click="clearLog" class="btn btn-clear">清空日誌</button>
      </div>
    </section>

    <section class="card query-panel">
      <h3>🔍 API 自動化查詢</h3>
      <div class="query-grid">
        <button @click="handleGet('q1')">Q1: 基隆站資料</button>
        <button @click="handleGet('q2')">Q2: 1月前10高</button>
        <button @click="handleGet('q3')">Q3: 每月平均</button>
        <button @click="handleGet('q4')">Q4: 全年極值</button>
        <button @click="handleGet('q5')">Q5: 每月最高站</button>
        <button @click="handleGet('q9')" class="highlight">Q9: 進階分級報表</button>
      </div>
    </section>

    <section class="card display-panel">
      <div class="display-header">
        <h3>🖥️ 伺服器回傳訊號 (JSON)</h3>
      </div>
      <div class="console-box">
        <pre v-if="apiResponse">{{ JSON.stringify(apiResponse, null, 2) }}</pre>
        <p v-else class="placeholder">系統連線就緒，等待指令...</p>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'; //畫面跟著變
import axios from 'axios'; //與sever溝通

// 後端網址
const BASE_URL = 'http://localhost:3000/api';

// 預設值
const apiResponse = ref(null);
const form = reactive({
  sitename: '北科大',
  monitormonth: 202605,
  concentration: 15.0
});

// --- [ API 操作邏輯 ] ---

// GET 查詢類 (Q1-Q5, Q9)
const handleGet = async (qNumber) => {
  try {
    const res = await axios.get(`${BASE_URL}/${qNumber}`);
    apiResponse.value = res.data;
  } catch (err) {
    apiResponse.value = { error: "查詢失敗", detail: err.message };
  }
};

// Q6: 新增資料 (POST)
const handlePost = async () => {
  try {
    const res = await axios.post(`${BASE_URL}/q6`, {
      sitename: form.sitename,
      monitormonth: Number(form.monitormonth),
      concentration: Number(form.concentration)
    });
    apiResponse.value = res.data;
  } catch (err) { apiResponse.value = err.response?.data || err.message; }
};

// Q7: 修改資料 (PUT) - 🚀 已經補上 monitormonth 精準打擊
const handlePut = async () => {
  try {
    const res = await axios.put(`${BASE_URL}/q7`, {
      sitename: form.sitename,
      monitormonth: Number(form.monitormonth),
      concentration: Number(form.concentration)
    });
    apiResponse.value = res.data;
  } catch (err) { apiResponse.value = err.response?.data || err.message; }
};

// Q8: 刪除資料 (DELETE) - 🚀 已經補上 monitormonth 雙重防護
const handleDelete = async () => {
  try {
    const res = await axios.delete(`${BASE_URL}/q8`, {
      data: { 
        sitename: form.sitename,
        monitormonth: Number(form.monitormonth)
      }
    });
    apiResponse.value = res.data;
  } catch (err) { apiResponse.value = err.response?.data || err.message; }
};

const clearLog = () => { apiResponse.value = null; };
</script>

<style scoped>
.container { max-width: 900px; margin: 0 auto; padding: 20px; background: #f8f9fa; min-height: 100vh; font-family: 'Inter', system-ui, sans-serif; }
header { text-align: center; margin-bottom: 30px; border-bottom: 2px solid #eee; padding-bottom: 10px; }
h1 { color: #2c3e50; font-weight: 700; }

.card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); margin-bottom: 20px; border: 1px solid #eee; }
h3 { margin-top: 0; font-size: 1em; color: #34495e; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 20px; }

.input-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
.input-group label { display: block; font-size: 0.85em; color: #666; margin-bottom: 5px; font-weight: 600; }
.input-group input { width: 100%; padding: 12px; border: 1px solid #ddd; border-radius: 6px; box-sizing: border-box; font-size: 14px; }
.input-group input:focus { outline: none; border-color: #42b983; box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.1); }

.action-buttons, .query-grid { display: flex; flex-wrap: wrap; gap: 10px; }

.btn { padding: 12px 20px; border: none; border-radius: 6px; cursor: pointer; font-weight: 600; font-size: 13px; transition: all 0.2s; }
.btn:hover { transform: translateY(-1px); filter: brightness(1.1); }
.btn-post { background: #3498db; color: white; }
.btn-put { background: #f39c12; color: white; }
.btn-del { background: #e74c3c; color: white; }
.btn-clear { background: #95a5a6; color: white; }

.query-grid button { padding: 10px 15px; background: #fff; border: 1px solid #ddd; border-radius: 6px; cursor: pointer; font-size: 13px; color: #555; }
.query-grid button:hover { border-color: #42b983; color: #42b983; }
.query-grid .highlight { background: #42b983; color: white; border: none; }

.console-box { background: #1e1e1e; color: #9cdcfe; padding: 20px; border-radius: 8px; font-family: 'Consolas', monospace; min-height: 200px; overflow-x: auto; box-shadow: inset 0 2px 10px rgba(0,0,0,0.2); }
pre { margin: 0; font-size: 13px; line-height: 1.6; }
.placeholder { color: #555; font-style: italic; }
</style>