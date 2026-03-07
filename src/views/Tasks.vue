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
        <!-- عرض الصورة إذا وجدت، وإلا عرض الأيقونة -->
        <div class="game-image-container">
          <img 
            v-if="game.image" 
            :src="game.image" 
            :alt="game.name"
            class="game-image"
            @error="handleImageError"
          >
          <div v-else class="game-icon">{{ game.icon || '🎮' }}</div>
        </div>
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
import { auth, getUserBalance, updateUserBalance, addTransaction } from "../firebase"
import { shallowRef } from 'vue'

export default {
  name: "Tasks",
  
  setup() {
    // استيراد جميع الألعاب تلقائياً من مجلد games
    const gameModules = import.meta.glob('../components/games/*.vue', { eager: true })
    
    // استيراد جميع الصور من مجلد games-images
    let gameImages = {}
    try {
      gameImages = import.meta.glob('../components/games-images/*.{png,jpg,jpeg,svg,gif}', { 
        eager: true,
        as: 'url'
      })
    } catch (error) {
      console.warn("⚠️ مجلد الصور غير موجود:", error)
    }
    
    const gamesComponents = []
    
    // تحويل الملفات المستوردة إلى مصفوفة من الكائنات
    for (const path in gameModules) {
      const component = gameModules[path].default
      const fileName = path.split('/').pop().replace('.vue', '')
      
      // البحث عن صورة بنفس اسم اللعبة
      let gameImage = null
      const imageExtensions = ['.png', '.jpg', '.jpeg', '.svg', '.gif']
      
      for (const ext of imageExtensions) {
        const imagePath = `../components/games-images/${fileName}${ext}`
        if (gameImages[imagePath]) {
          gameImage = gameImages[imagePath]
          break
        }
      }
      
      // استخراج اسم اللعبة من الملف أو استخدام اسم الملف
      let gameName = fileName.replace(/([A-Z])/g, ' $1').replace(/^./, str => str.toUpperCase())
      let gameIcon = '🎮'
      let gameDescription = 'اضغط للعب'
      
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
        image: gameImage, // إضافة مسار الصورة
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
        console.log("⚠️ لا يوجد مستخدم مسجل الدخول")
        this.loading = false
        return
      }

      try {
        this.balance = await getUserBalance(user.uid)
        console.log("✅ تم جلب الرصيد:", this.balance)
      } catch (error) {
        console.error("❌ خطأ في جلب الرصيد:", error)
      } finally {
        this.loading = false
      }
    },

    async updateBalance(newBalance, gameName = '', betAmount = 0, won = false) {
      const oldBalance = this.balance
      this.balance = newBalance
      
      const user = auth.currentUser
      if (!user) return

      try {
        // تحديث الرصيد في Firebase
        const success = await updateUserBalance(user.uid, newBalance)
        
        if (success) {
          console.log(`✅ تم تحديث الرصيد: ${oldBalance} -> ${newBalance}`)
          
          // تسجيل المعاملة
          if (gameName && betAmount > 0) {
            await addTransaction(user.uid, {
              gameName,
              betAmount,
              won,
              oldBalance,
              newBalance,
              profit: newBalance - oldBalance
            })
          }
        } else {
          // إعادة الرصيد القديم في حالة الفشل
          this.balance = oldBalance
          this.showResult('فشل تحديث الرصيد!', false)
        }
      } catch (error) {
        console.error("❌ خطأ في تحديث الرصيد:", error)
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
    },

    handleImageError(event) {
      // في حالة فشل تحميل الصورة، نظهر الأيقونة بدلاً منها
      event.target.style.display = 'none'
      event.target.parentElement.innerHTML = '<div class="game-icon">🎮</div>'
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
  position: sticky;
  top: 20px;
  z-index: 100;
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
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 215, 0, 0.3);
}

.balance-card i {
  color: #f1c40f;
  font-size: 28px;
  filter: drop-shadow(0 0 5px rgba(241, 196, 15, 0.5));
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
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  color: white;
  border: 2px solid #f1c40f;
}

.result-message.lose {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  color: white;
  border: 2px solid #f1c40f;
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
  color: #f1c40f;
}

/* شبكة الألعاب */
.games-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 25px;
  max-width: 1200px;
  margin: 0 auto;
}

.game-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 15px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  text-align: center;
  backdrop-filter: blur(10px);
  border: 2px solid transparent;
}

.game-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.2);
  background: white;
  border-color: #f1c40f;
}

.game-image-container {
  width: 100%;
  height: 180px;
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-radius: 10px;
  background: linear-gradient(135deg, #f6f9fc 0%, #e6f0f5 100%);
}

.game-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.game-card:hover .game-image {
  transform: scale(1.05);
}

.game-icon {
  font-size: 64px;
  margin: 30px 0;
  filter: drop-shadow(0 5px 10px rgba(0, 0, 0, 0.2));
}

.game-name {
  font-size: 20px;
  color: #333;
  margin-bottom: 10px;
  font-weight: bold;
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
  background: rgba(0, 0, 0, 0.95);
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
  background: linear-gradient(135deg, #667eea, #764ba2);
  padding: 15px 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  color: white;
}

.close-button {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: white;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.close-button:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: rotate(90deg);
}

.game-modal-header h2 {
  margin: 0;
  font-size: 20px;
  flex: 1;
}

.game-modal-content {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

/* تحسينات الجوال */
@media (max-width: 768px) {
  .games-page {
    padding: 10px;
  }

  .games-grid {
    grid-template-columns: 1fr;
    gap: 15px;
  }

  .game-image-container {
    height: 150px;
  }

  .balance-card {
    font-size: 18px;
    padding: 12px 25px;
  }

  .result-message {
    font-size: 14px;
    padding: 10px 20px;
    top: 80px;
  }

  .game-modal-header h2 {
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .balance-card {
    font-size: 16px;
    padding: 10px 20px;
  }

  .game-image-container {
    height: 130px;
  }

  .game-icon {
    font-size: 48px;
    margin: 20px 0;
  }

  .game-name {
    font-size: 16px;
  }

  .game-description {
    font-size: 12px;
  }
}
</style>
