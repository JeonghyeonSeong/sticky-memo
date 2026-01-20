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
          끝말잇기를 시작하세요...
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
  // 브라우저가 허용하는 안정적인 사이즈로 설정 (350px 정도가 가장 안전합니다)
  window.resizeTo(350, 480); 
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
  if (gameLogs.value.length > 0) {
    const lastWord = gameLogs.value[gameLogs.value.length - 1].word;
    const lastChar = lastWord.charAt(lastWord.length - 1);
    if (word[0] !== lastChar) {
      alert(`'${lastChar}'로 시작해야 합니다!`);
      return;
    }
  }
  push(logsRef, { word, timestamp: Date.now() });
  newWord.value = '';
};
</script>

<style>
/* 모든 요소에 box-sizing 적용하여 테두리가 너비에 포함되게 함 */
* { box-sizing: border-box; }

html, body, #app {
  margin: 0 !important;
  padding: 0 !important;
  width: 100% !important;
  height: 100% !important;
  overflow: hidden !important;
  background-color: #fef3b4 !important;
}

.app-container {
  width: 100%;
  height: 100%;
  position: relative;
  display: flex;
  justify-content: center; /* 창이 커져도 중앙 정렬 */
}

.sticky-note {
  width: 100%;
  max-width: 100%; /* 부모 너비를 넘지 않음 */
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
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.note-body::-webkit-scrollbar { display: none; }

.memo-line { color: #333; margin-bottom: 4px; word-break: break-all; }
.bullet { color: #888; margin-right: 5px; }

.hidden-input {
  border: none;
  background: transparent;
  padding: 15px;
  outline: none;
  font-size: 14px;
  width: 100%;
  border-top: 1px dashed rgba(0,0,0,0.1);
}

/* 짤림 방지 전면 오버레이 */
.full-overlay {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background-color: #fef3b4;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.overlay-content {
  width: 100%;
  padding: 20px;
  text-align: center;
}

.warning-text { color: #ff5f56; font-weight: bold; font-size: 12px; margin-bottom: 5px; }
.main-text { font-size: 15px; color: #333; margin-bottom: 25px; font-weight: bold; }

.btn-group {
  display: flex;
  flex-direction: column; /* 세로 배치가 좁은 창에서 가장 안전함 */
  gap: 10px;
  padding: 0 20px;
}

.btn-group button {
  width: 100%;
  padding: 12px 0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  font-weight: bold;
}

.btn-confirm { background-color: #ff5f56; color: white; }
.btn-hide { background-color: rgba(0,0,0,0.05); color: #666; }
</style>