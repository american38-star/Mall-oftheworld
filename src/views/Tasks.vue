<template>
  <div class="game-page">
    <!-- الرصيد -->
    <div class="top-bar">
      <div class="balance-gold">
        <i class="fas fa-coins"></i>
        <span>رصيدك: <strong>{{ balance.toFixed(2) }} USDT</strong></span>
      </div>
    </div>

    <!-- رسالة الفوز/الخسارة -->
    <transition name="slide-fade">
      <div v-if="showResultMessage" class="result-message" :class="resultType">
        <i :class="resultIcon"></i>
        <span>{{ resultText }}</span>
      </div>
    </transition>

    <!-- زر الرجوع عندما تكون اللعبة مفتوحة -->
    <div v-if="gameOpened" class="back-button-container">
      <button @click="closeGame" class="back-button">
        <i class="fas fa-arrow-right"></i>
        رجوع إلى الألعاب
      </button>
    </div>

    <!-- التبويبات - تظهر فقط عندما لا تكون اللعبة مفتوحة -->
    <div v-if="!gameOpened" class="tabs">
      <button 
        v-for="tab in gamesList" 
        :key="tab.id"
        :class="{active: selectedGame === tab.id}" 
        @click="openGame(tab.id)"
      >
        <span class="tab-icon">{{ tab.icon }}</span>
        <span class="tab-text">{{ tab.name }}</span>
      </button>
    </div>

    <!-- عرض اللعبة المختارة - تظهر بملء الشاشة -->
    <div v-if="gameOpened" class="game-fullscreen">
      <!-- اللعبة الجديدة فقط -->
      <ChickenRoadGame 
        v-if="selectedGame === 'chicken-game'"
        :balance="balance"
        @update-balance="updateBalance"
        @show-result="showResult"
      />
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase"
import { doc, getDoc, updateDoc } from "firebase/firestore"

// استيراد اللعبة الجديدة فقط
import ChickenRoadGame from './components/games/ChickenRoadGame.vue'

export default {
  name: "Games",
  
  components: {
    ChickenRoadGame
  },
  
  data() {
    return {
      gameOpened: false,
      selectedGame: "chicken-game",
      showResultMessage: false,
      resultType: '',
      resultIcon: '',
      resultText: '',
      resultTimeout: null,
      
      gamesList: [
        { id: 'chicken-game', name: 'Chicken Road Game', icon: '🐔' }
      ],
      
      balance: 0,
      gameError: ""
    }
  },
  
  async created() {
    const user = auth.currentUser
    if (!user) {
      console.log("لا يوجد مستخدم مسجل")
      return
    }
    
    try {
      const snap = await getDoc(doc(db, "users", user.uid))
      if (snap.exists()) {
        this.balance = Number(snap.data().balance || 0)
        console.log("تم جلب الرصيد:", this.balance)
      } else {
        console.log("المستخدم ليس لديه رصيد")
        this.balance = 0
      }
    } catch (error) {
      console.error("خطأ في جلب الرصيد:", error)
    }
  },
  
  methods: {
    openGame(gameId) {
      this.selectedGame = gameId
      this.gameOpened = true
      this.gameError = ""
      console.log("تم فتح اللعبة:", gameId)
    },
    
    closeGame() {
      this.gameOpened = false
      console.log("تم إغلاق اللعبة")
    },
    
    async updateBalance(newBalance) {
      console.log("تحديث الرصيد من:", this.balance, "إلى:", newBalance)
      this.balance = newBalance
      
      const user = auth.currentUser
      if (user) {
        try {
          await updateDoc(doc(db, "users", user.uid), {
            balance: this.balance
          })
          console.log("تم تحديث الرصيد في Firebase:", this.balance)
        } catch (error) {
          console.error("خطأ في تحديث الرصيد:", error)
        }
      }
    },
    
    showResult(message, isWin) {
      if (this.resultTimeout) clearTimeout(this.resultTimeout)
      
      this.resultText = message
      this.resultType = isWin ? 'win-message' : 'lose-message'
      this.resultIcon = isWin ? 'fas fa-trophy' : 'fas fa-skull'
      this.showResultMessage = true
      
      this.resultTimeout = setTimeout(() => {
        this.showResultMessage = false
      }, 2000)
    }
  }
}
</script>

<style scoped>
/* ===== التصميم العام ===== */
.game-page {
  background: linear-gradient(135deg, #0a0f1e 0%, #1a1f2f 100%);
  min-height: 100vh;
  color: #ffffff;
  padding: 15px;
  text-align: center;
  direction: rtl;
  font-family: 'Montserrat', 'Cairo', sans-serif;
  position: relative;
  overflow-x: hidden;
}

/* ===== الشريط العلوي ===== */
.top-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  z-index: 10;
  position: relative;
}

.balance-gold {
  background: linear-gradient(145deg, #1e2333, #131826);
  padding: 12px 30px;
  border-radius: 100px;
  border: 1px solid #ffd700;
  box-shadow: 0 5px 20px rgba(255, 215, 0, 0.2), inset 0 0 10px rgba(255, 215, 0, 0.1);
  display: flex;
  align-items: center;
  gap: 12px;
  backdrop-filter: blur(10px);
}

.balance-gold i {
  color: #ffd700;
  font-size: 22px;
  filter: drop-shadow(0 0 8px #ffd700);
}

.balance-gold strong {
  color: #ffd700;
  font-size: 20px;
  margin-right: 5px;
  text-shadow: 0 0 15px rgba(255, 215, 0, 0.5);
}

/* ===== رسالة النتيجة ===== */
.result-message {
  position: fixed;
  top: 80px;
  left: 50%;
  transform: translateX(-50%);
  padding: 15px 30px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 18px;
  font-weight: 700;
  z-index: 9999;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  border: 1px solid;
  backdrop-filter: blur(10px);
  min-width: 280px;
  justify-content: center;
}

.win-message {
  background: linear-gradient(145deg, #1a2f1a, #0f1f0f);
  border-color: #4caf50;
  box-shadow: 0 0 30px rgba(76, 175, 80, 0.3);
  color: #4caf50;
}

.win-message i {
  color: #4caf50;
  filter: drop-shadow(0 0 8px #4caf50);
}

.lose-message {
  background: linear-gradient(145deg, #2f1a1a, #1f0f0f);
  border-color: #f44336;
  box-shadow: 0 0 30px rgba(244, 67, 54, 0.3);
  color: #f44336;
}

.lose-message i {
  color: #f44336;
  filter: drop-shadow(0 0 8px #f44336);
}

.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from,
.slide-fade-leave-to {
  transform: translateX(-50%) translateY(-20px);
  opacity: 0;
}

/* ===== زر الرجوع ===== */
.back-button-container {
  margin-bottom: 20px;
  text-align: right;
}

.back-button {
  background: linear-gradient(145deg, #1e2333, #131826);
  color: #ffd700;
  border: 1px solid #ffd700;
  padding: 10px 25px;
  border-radius: 50px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.back-button:hover {
  background: linear-gradient(135deg, #ffd700, #ffed4a);
  color: #0a0f1e;
  transform: translateX(-5px);
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.4);
}

/* ===== التبويبات ===== */
.tabs {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
  margin-bottom: 25px;
  max-height: 200px;
  overflow-y: auto;
  padding: 15px;
  background: rgba(30, 35, 51, 0.7);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  border: 1px solid rgba(255, 215, 0, 0.2);
  scrollbar-width: thin;
  scrollbar-color: #ffd700 #1e2333;
}

.tabs::-webkit-scrollbar {
  width: 8px;
}

.tabs::-webkit-scrollbar-track {
  background: #1e2333;
  border-radius: 10px;
}

.tabs::-webkit-scrollbar-thumb {
  background: #ffd700;
  border-radius: 10px;
  box-shadow: inset 0 0 5px #ffed4a;
}

.tabs button {
  padding: 12px 20px;
  border-radius: 50px;
  background: linear-gradient(145deg, #1e2333, #131826);
  color: #ffffff;
  border: 1px solid rgba(255, 215, 0, 0.3);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.tabs button:hover {
  border-color: #ffd700;
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(255, 215, 0, 0.3);
}

.tabs .active {
  background: linear-gradient(135deg, #ffd700, #ffed4a, #ffd700);
  color: #0a0f1e;
  border: none;
  box-shadow: 0 5px 25px rgba(255, 215, 0, 0.5);
}

.tab-icon {
  font-size: 18px;
  filter: drop-shadow(0 0 5px currentColor);
}

.tab-text {
  font-size: 12px;
}

/* ===== وضع ملء الشاشة ===== */
.game-fullscreen {
  min-height: calc(100vh - 150px);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 10px 0;
}

/* ===== تحسينات الجوال ===== */
@media (max-width: 480px) {
  .game-page {
    padding: 10px;
  }

  .tabs button {
    padding: 10px 15px;
    font-size: 12px;
  }

  .result-message {
    font-size: 14px;
    padding: 12px 20px;
    min-width: 240px;
  }

  .back-button {
    width: 100%;
    justify-content: center;
  }
}
</style>
