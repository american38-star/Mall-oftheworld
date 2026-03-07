<!-- src/views/Tasks.vue -->
<template>
  <div class="games-page">
    <!-- الرصيد -->
    <div class="balance-container">
      <div class="balance-card">
        <i class="fas fa-coins"></i>
        <span>Balance: ${{ balance.toFixed(2) }}</span>
      </div>
    </div>

    <!-- رسالة النتيجة -->
    <transition name="fade">
      <div v-if="resultMessage.show" class="result-message" :class="resultMessage.type">
        <i :class="resultMessage.icon"></i>
        <span>{{ resultMessage.text }}</span>
      </div>
    </transition>

    <!-- تحميل الألعاب -->
    <div v-if="loading" class="loading">
      <i class="fas fa-spinner fa-spin"></i>
      <span>جاري تحميل الألعاب...</span>
    </div>

    <!-- شبكة الألعاب -->
    <div v-else class="games-grid">
      <div 
        v-for="game in gamesComponents" 
        :key="game.id"
        class="game-card"
        @click="openGame(game)"
      >
        <div class="game-icon">{{ game.icon || '🎮' }}</div>
        <h3 class="game-name">{{ game.name }}</h3>
        <p class="game-description">{{ game.description || 'اضغط للعب' }}</p>
      </div>
    </div>

    <!-- نافذة اللعبة -->
    <transition name="slide-up">
      <div v-if="selectedGame" class="game-modal">
        <div class="game-modal-header">
          <button class="close-button" @click="closeGame">
            <i class="fas fa-times"></i>
          </button>
          <h2>{{ selectedGame.name }}</h2>
        </div>
        <div class="game-modal-content">
          <component 
            :is="selectedGame.component"
            :balance="balance"
            @update-balance="updateBalance"
            @show-result="showResult"
          />
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { auth, db } from "../firebase"
import { doc, getDoc, updateDoc } from "firebase/firestore"
import { ref, shallowRef, onMounted } from 'vue'

export default {
  name: "Tasks",
  
  setup() {
    // استيراد جميع الألعاب تلقائياً من مجلد games
    const gameModules = import.meta.glob('../components/games/*.vue', { eager: true })
    
    const gamesComponents = []
    
    // تحويل الملفات المستوردة إلى مصفوفة من الكائنات
    for (const path in gameModules) {
      const component = gameModules[path].default
      const fileName = path.split('/').pop().replace('.vue', '')
      
      // استخراج اسم اللعبة من الملف أو استخدام اسم الملف
      let gameName = fileName
      let gameIcon = '🎮'
      let gameDescription = ''
      
      // إذا كان المكون يحتوي على خصائص meta، استخدمها
      if (component.meta) {
        gameName = component.meta.name || gameName
        gameIcon = component.meta.icon || gameIcon
        gameDescription = component.meta.description || gameDescription
      }
      
      gamesComponents.push({
        id: fileName,
        name: gameName,
        icon: gameIcon,
        description: gameDescription,
        component: shallowRef(component)
      })
    }
    
    return {
      gamesComponents
    }
  },

  data() {
    return {
      balance: 0,
      loading: true,
      selectedGame: null,
      resultMessage: {
        show: false,
        type: '',
        icon: '',
        text: ''
      },
      resultTimeout: null
    }
  },

  async created() {
    await this.fetchUserBalance()
  },

  methods: {
    async fetchUserBalance() {
      this.loading = true
      const user = auth.currentUser
      
      if (!user) {
        console.log("لا يوجد مستخدم مسجل الدخول")
        this.loading = false
        return
      }

      try {
        const userRef = doc(db, "users", user.uid)
        const userSnap = await getDoc(userRef)
        
        if (userSnap.exists()) {
          this.balance = Number(userSnap.data().balance || 0)
          console.log("تم جلب الرصيد:", this.balance)
        } else {
          // إذا لم يكن المستخدم موجوداً، قم بإنشائه برصيد 0
          await updateDoc(userRef, { balance: 0 })
          this.balance = 0
          console.log("تم إنشاء مستخدم جديد برصيد 0")
        }
      } catch (error) {
        console.error("خطأ في جلب الرصيد:", error)
      } finally {
        this.loading = false
      }
    },

    async updateBalance(newBalance) {
      const oldBalance = this.balance
      this.balance = newBalance
      
      const user = auth.currentUser
      if (!user) return

      try {
        const userRef = doc(db, "users", user.uid)
        await updateDoc(userRef, { balance: newBalance })
        console.log(`تم تحديث الرصيد: ${oldBalance} -> ${newBalance}`)
      } catch (error) {
        console.error("خطأ في تحديث الرصيد:", error)
        // إعادة الرصيد القديم في حالة الفشل
        this.balance = oldBalance
      }
    },

    openGame(game) {
      this.selectedGame = game
    },

    closeGame() {
      this.selectedGame = null
    },

    showResult(message, isWin) {
      if (this.resultTimeout) {
        clearTimeout(this.resultTimeout)
      }

      this.resultMessage = {
        show: true,
        type: isWin ? 'win' : 'lose',
        icon: isWin ? 'fas fa-trophy' : 'fas fa-skull',
        text: message
      }

      this.resultTimeout = setTimeout(() => {
        this.resultMessage.show = false
      }, 2000)
    }
  },

  beforeUnmount() {
    if (this.resultTimeout) {
      clearTimeout(this.resultTimeout)
    }
  }
}
</script>

<style scoped>
.games-page {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

/* حاوية الرصيد */
.balance-container {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.balance-card {
  background: rgba(255, 255, 255, 0.95);
  padding: 15px 40px;
  border-radius: 50px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.balance-card i {
  color: #f1c40f;
  font-size: 28px;
}

/* رسالة النتيجة */
.result-message {
  position: fixed;
  top: 100px;
  left: 50%;
  transform: translateX(-50%);
  padding: 15px 30px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 18px;
  font-weight: bold;
  z-index: 1000;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.2);
  animation: slideDown 0.3s ease;
}

.result-message.win {
  background: #2ecc71;
  color: white;
}

.result-message.lose {
  background: #e74c3c;
  color: white;
}

@keyframes slideDown {
  from {
    top: 50px;
    opacity: 0;
  }
  to {
    top: 100px;
    opacity: 1;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateX(-50%) translateY(-20px);
}

/* التحميل */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 300px;
  color: white;
  font-size: 18px;
  gap: 15px;
}

.loading i {
  font-size: 40px;
}

/* شبكة الألعاب */
.games-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.game-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  padding: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.game-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.2);
  background: white;
}

.game-icon {
  font-size: 48px;
  margin-bottom: 15px;
}

.game-name {
  font-size: 20px;
  color: #333;
  margin-bottom: 10px;
}

.game-description {
  color: #666;
  font-size: 14px;
  margin: 0;
}

/* نافذة اللعبة */
.game-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.9);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
  }
  to {
    transform: translateY(0);
  }
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.3s ease;
}

.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
}

.game-modal-header {
  background: white;
  padding: 15px 20px;
  display: flex;
  align-items: center;
  gap: 15px;
}

.close-button {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.close-button:hover {
  background: #f0f0f0;
  color: #e74c3c;
}

.game-modal-header h2 {
  margin: 0;
  color: #333;
  font-size: 20px;
}

.game-modal-content {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

/* تحسينات الجوال */
@media (max-width: 768px) {
  .games-grid {
    grid-template-columns: 1fr;
  }

  .balance-card {
    font-size: 18px;
    padding: 12px 25px;
  }

  .result-message {
    font-size: 14px;
    padding: 10px 20px;
  }
}
</style>
