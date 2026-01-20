<template>
  <div class="app-container">
    <div class="sticky-note">
      <div class="note-header">
        <div class="header-left" @click="openResetModal" title="새 메모 시작">+</div>
        <div class="header-title">스티커 메모</div>
        <div class="header-right" @click="openResetModal">×</div>
      </div>
      
      <div class="note-body" ref="scrollContainer">
        <div v-if="gameLogs.length === 0" class="system-msg">
          메모를 자유롭게 남겨보세요...
        </div>
        <div v-for="(item, index) in gameLogs" :key="index" class="memo-line">
          <span class="bullet">•</span> {{ item.word }}
        </div>
      </div>

      <input 
        type="text" 
        v-model="newWord" 
        @keyup.enter="submitWord" 
        placeholder="여기에 메모하세요..." 
        class="hidden-input"
        autofocus
      />
    </div>

    <div v-if="isModalOpen" class="full-overlay">
      <div class="overlay-content">
        <p class="warning-text">⚠️ 기록 삭제</p>
        <p class="main-text">모든 내용을 지울까요?</p>
        <div class="btn-group">
          <button @click="confirmDelete" class="btn-confirm">삭제하기</button>
          <button @click="isModalOpen = false" class="btn-hide">돌아가기</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
import { initializeApp } from 'firebase/app';
import { getDatabase, ref as dbRef, push, onValue, remove } from 'firebase/database';

// Firebase 설정 (정현님 프로젝트 정보 유지)
const firebaseConfig = {
  apiKey: "AIzaSyBbUmMXG01BetxUCRcx03gpbYmk-vb-SXo",
  authDomain: "sticker-2da9a.firebaseapp.com",
  databaseURL: "https://sticker-2da9a-default-rtdb.firebaseio.com", 
  projectId: "sticker-2da9a",
  storageBucket: "sticker-2da9a.firebasestorage.app",
  messagingSenderId: "711701476041",
  appId: "1:711701476041:web:71e6c5e1e2d56801455318",
  measurementId: "G-5Y54BX0YKM"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);
const logsRef = dbRef(db, 'game/logs');

const gameLogs = ref([]);
const newWord = ref('');
const scrollContainer = ref(null);
const isModalOpen = ref(false);

const openResetModal = () => { isModalOpen.value = true; };
const confirmDelete = () => {
  remove(logsRef).then(() => {
    gameLogs.value = [];
    isModalOpen.value = false;
  });
};

const handleKeydown = (e) => {
  if (e.key === 'Escape') window.location.href = 'https://www.google.com';
};

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
  onValue(logsRef, (snapshot) => {
    const data = snapshot.val();
    gameLogs.value = data ? Object.values(data) : [];
    nextTick(() => {
      if (scrollContainer.value) scrollContainer.value.scrollTop = scrollContainer.value.scrollHeight;
    });
  });
});

onUnmounted(() => { window.removeEventListener('keydown', handleKeydown); });

const submitWord = () => {
  const word = newWord.value.trim();
  if (!word) return;
  push(logsRef, { word, timestamp: Date.now() });
  newWord.value = '';
};
</script>

<style>
* { box-sizing: border-box; }
html, body, #app { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; background-color: #fef3b4; }

.app-container {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  justify-content: flex-start; /* [수정] 왼쪽 정렬 */
}

.sticky-note {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #fef3b4;
  font-family: 'Malgun Gothic', sans-serif;
}

.note-header {
  height: 32px;
  display: flex;
  justify-content: space-between;
  padding: 0 12px;
  align-items: center;
  background-color: rgba(0,0,0,0.05);
  color: #555;
  font-size: 12px;
}

.note-body {
  flex: 1;
  padding: 15px;
  overflow-y: auto;
  font-size: 14px;
  line-height: 1.7;
  text-align: left; /* [수정] 텍스트 왼쪽 정렬 */
  scrollbar-width: none;
}
.note-body::-webkit-scrollbar { display: none; }

.memo-line { 
  color: #333; 
  margin-bottom: 4px; 
  word-break: break-all;
  display: flex; /* 불렛과 텍스트 나란히 배치 */
  align-items: flex-start;
}

.bullet { color: #888; margin-right: 8px; flex-shrink: 0; }

.hidden-input {
  border: none;
  background: transparent;
  padding: 15px;
  outline: none;
  font-size: 14px;
  width: 100%;
  text-align: left; /* [수정] 입력창도 왼쪽 정렬 */
  border-top: 1px dashed rgba(0,0,0,0.1);
}

.full-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background-color: #fef3b4; display: flex; justify-content: center; align-items: center; z-index: 9999; }
.overlay-content { width: 100%; padding: 20px; text-align: center; }
.warning-text { color: #ff5f56; font-weight: bold; font-size: 12px; margin-bottom: 5px; }
.main-text { font-size: 15px; color: #333; margin-bottom: 25px; font-weight: bold; }
.btn-group { display: flex; flex-direction: column; gap: 10px; padding: 0 20px; }
.btn-group button { width: 100%; padding: 12px 0; border: none; border-radius: 6px; cursor: pointer; font-size: 13px; font-weight: bold; }
.btn-confirm { background-color: #ff5f56; color: white; }
.btn-hide { background-color: rgba(0,0,0,0.05); color: #666; }
.system-msg { color: #aaa; text-align: left; margin-top: 5px; font-size: 12px; }
</style>
