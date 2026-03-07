<!-- src/views/Tasks.vue -->
<template>
  <div class="games-page">
    <!-- الرصيد - بنفس تصميم الصورة -->
    <div class="balance-container">
      <div class="balance-card">
        <i class="fas fa-coins"></i>
        <span>Balance : <strong>${{ balance.toFixed(2) }}</strong></span>
        <span class="emoji">😊</span>
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

    <!-- شبكة الألعاب - بنفس ترتيب الصورة -->
    <div v-else class="games-container">
      <!-- لعبة Chicken Crash -->
      <div v-if="getGameByName('Chicken Crash')" class="game-section">
        <div class="game-header">
          <h2 class="game-title-large">CHICKEN CRASH</h2>
          <div class="game-price">$999,99</div>
        </div>
        
        <div class="game-row">
          <div 
            class="game-card-large"
            @click="openGame(getGameByName('Chicken Crash'))"
          >
            <div class="game-image-wrapper">
              <img 
                v-if="getGameByName('Chicken Crash').image" 
                :src="getGameByName('Chicken Crash').image" 
                :alt="getGameByName('Chicken Crash').name"
                class="game-image-large"
                @error="handleImageError($event, getGameByName('Chicken Crash'))"
              >
              <div v-else class="game-icon-large">{{ getGameByName('Chicken Crash').icon || '🐔' }}</div>
            </div>
            <div class="game-stats">
              <span class="stat-value">0.000</span>
            </div>
          </div>

          <!-- لعبة Big Wheel بجانبها -->
          <div 
            v-if="getGameByName('Big Wheel')"
            class="game-card-large secondary"
            @click="openGame(getGameByName('Big Wheel'))"
          >
            <div class="game-image-wrapper">
              <img 
                v-if="getGameByName('Big Wheel').image" 
                :src="getGameByName('Big Wheel').image" 
                :alt="getGameByName('Big Wheel').name"
                class="game-image-large"
                @error="handleImageError($event, getGameByName('Big Wheel'))"
              >
              <div v-else class="game-icon-large">{{ getGameByName('Big Wheel').icon || '🎡' }}</div>
            </div>
            <h3 class="game-title-small">BIG WHEEL BONUS</h3>
          </div>
        </div>
      </div>

      <!-- ألعاب إضافية في شبكة -->
      <div v-if="otherGames.length > 0" class="extra-games-grid">
        <div 
          v-for="game in otherGames" 
          :key="game.id"
          class="game-card-small"
          @click="openGame(game)"
        >
          <div class="game-image-wrapper-small">
            <img 
              v-if="game.image" 
              :src="game.image" 
              :alt="game.name"
              class="game-image-small"
              @error="handleImageError($event, game)"
            >
            <div v-else class="game-icon-small">{{ game.icon || '🎮' }}</div>
          </div>
          <h4 class="game-title-small">{{ game.name }}</h4>
        </div>
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
import { shallowRef, computed } from 'vue'

export default {
  name: "Tasks",
  
  setup() {
    // استيراد جميع الألعاب تلقائياً من مجلد games
    const gameModules = import.meta.glob('../components/games/*.vue', { eager: true })
    
    // استيراد جميع الصور من مجلد assets
    const assetImages = import.meta.glob('../assets/*.{png,jpg,jpeg,svg,gif,webp}', { 
      eager: true,
      as: 'url'
    })
    
    console.log("📸 الصور الموجودة في assets:", Object.keys(assetImages))
    
    const gamesComponents = []
    
    // تحويل الملفات المستوردة إلى مصفوفة من الكائنات
    for (const path in gameModules) {
      const component = gameModules[path].default
      const fileName = path.split('/').pop().replace('.vue', '')
      const fileNameLower = fileName.toLowerCase()
      
      // البحث عن صورة بنفس اسم اللعبة في مجلد assets
      let gameImage = null
      
      // قائمة بامتدادات الصور المدعومة
      const imageExtensions = ['.png', '.jpg', '.jpeg', '.svg', '.gif', '.webp']
      
      // البحث عن صورة تطابق اسم اللعبة
      for (const imagePath in assetImages) {
        const imageFileName = imagePath.split('/').pop().toLowerCase()
        
        // التحقق مما إذا كان اسم الصورة يحتوي على اسم اللعبة
        if (imageFileName.includes(fileNameLower) || 
            fileNameLower.includes(imageFileName.replace(/\.[^/.]+$/, ""))) {
          gameImage = assetImages[imagePath]
          console.log(`✅ تم العثور على صورة للعبة ${fileName}: ${imagePath}`)
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
        image: gameImage,
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
      balance: 100.00, // قيمة افتراضية
      loading: true,
      selectedGame: null,
      resultMessage: {
        show: false,
        type: '',
        icon: '',
        text: ''
      },
      resultTimeout: null,
      failedImages: new Set()
    }
  },

  computed: {
    // الحصول على لعبة Chicken Crash
    getGameByName() {
      return (name) => {
        return this.gamesComponents.find(game => 
          game.name.toLowerCase().includes(name.toLowerCase())
        )
      }
    },
    
    // الألعاب الأخرى (باستثناء Chicken Crash و Big Wheel)
    otherGames() {
      return this.gamesComponents.filter(game => 
        !game.name.toLowerCase().includes('chicken') && 
        !game.name.toLowerCase().includes('wheel')
      )
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
        this.balance = 100.00 // قيمة افتراضية
        this.loading = false
        return
      }

      try {
        this.balance = await getUserBalance(user.uid)
        console.log("✅ تم جلب الرصيد:", this.balance)
      } catch (error) {
        console.error("❌ خطأ في جلب الرصيد:", error)
        this.balance = 100.00
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
        const success = await updateUserBalance(user.uid, newBalance)
        
        if (success) {
          console.log(`✅ تم تحديث الرصيد: ${oldBalance} -> ${newBalance}`)
          
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

    handleImageError(event, game) {
      if (this.failedImages.has(game.id)) return
      
      this.failedImages.add(game.id)
      console.warn(`⚠️ فشل تحميل صورة اللعبة: ${game.name}`)
      
      event.target.style.display = 'none'
      const iconDiv = document.createElement('div')
      iconDiv.className = game.id.includes('Chicken') ? 'game-icon-large' : 'game-icon-small'
      iconDiv.textContent = game.icon || '🎮'
      event.target.parentNode.appendChild(iconDiv)
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
/* ===== التصميم العام ===== */
.games-page {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1f2e 0%, #2a2f3f 100%);
  padding: 20px;
  font-family: 'Montserrat', 'Arial', sans-serif;
  color: #ffffff;
}

/* ===== حاوية الرصيد - بنفس تصميم الصورة ===== */
.balance-container {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
  padding: 0 15px;
}

.balance-card {
  background: #2a2f42;
  padding: 15px 40px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 24px;
  color: #ffffff;
  border: 1px solid #3a4055;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.balance-card i {
  color: #ffd700;
  font-size: 28px;
}

.balance-card strong {
  color: #ffd700;
  font-weight: 700;
  margin: 0 5px;
}

.balance-card .emoji {
  font-size: 28px;
}

/* ===== حاوية الألعاب ===== */
.games-container {
  max-width: 1200px;
  margin: 0 auto;
}

/* ===== قسم اللعبة الرئيسي ===== */
.game-section {
  background: #1e2335;
  border-radius: 20px;
  padding: 25px;
  margin-bottom: 30px;
  border: 1px solid #3a4055;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid #3a4055;
}

.game-title-large {
  font-size: 28px;
  font-weight: 800;
  color: #ffffff;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin: 0;
}

.game-price {
  font-size: 32px;
  font-weight: 700;
  color: #ffd700;
  text-shadow: 0 0 10px rgba(255, 215, 0, 0.3);
}

/* ===== صف الألعاب الرئيسي ===== */
.game-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 25px;
}

/* ===== بطاقة اللعبة الكبيرة ===== */
.game-card-large {
  background: #2a2f42;
  border-radius: 15px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  text-align: center;
}

.game-card-large:hover {
  transform: translateY(-5px);
  border-color: #ffd700;
  box-shadow: 0 10px 30px rgba(255, 215, 0, 0.2);
}

.game-card-large.secondary {
  background: #252a3c;
}

.game-image-wrapper {
  width: 100%;
  height: 200px;
  margin-bottom: 15px;
  border-radius: 10px;
  overflow: hidden;
  background: #1e2335;
}

.game-image-large {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.game-card-large:hover .game-image-large {
  transform: scale(1.05);
}

.game-icon-large {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 80px;
  background: linear-gradient(135deg, #ffd700, #ffa500);
  color: #1e2335;
}

.game-stats {
  margin-top: 10px;
  padding: 10px;
  background: #1e2335;
  border-radius: 8px;
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: #ffd700;
}

/* ===== عنوان اللعبة الصغير ===== */
.game-title-small {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
  margin: 15px 0 5px;
  text-transform: uppercase;
}

/* ===== شبكة الألعاب الإضافية ===== */
.extra-games-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
  margin-top: 30px;
}

.game-card-small {
  background: #2a2f42;
  border-radius: 12px;
  padding: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  text-align: center;
}

.game-card-small:hover {
  transform: translateY(-3px);
  border-color: #ffd700;
  box-shadow: 0 5px 20px rgba(255, 215, 0, 0.2);
}

.game-image-wrapper-small {
  width: 100%;
  height: 150px;
  border-radius: 8px;
  overflow: hidden;
  background: #1e2335;
  margin-bottom: 10px;
}

.game-image-small {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.game-icon-small {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 50px;
  background: linear-gradient(135deg, #3a4055, #2a2f42);
  color: #ffd700;
}

/* ===== رسالة النتيجة ===== */
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
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.3);
  animation: slideDown 0.3s ease;
}

.result-message.win {
  background: linear-gradient(135deg, #2ecc71, #27ae60);
  color: white;
}

.result-message.lose {
  background: linear-gradient(135deg, #e74c3c, #c0392b);
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

/* ===== التحميل ===== */
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 300px;
  color: #ffffff;
  font-size: 18px;
  gap: 15px;
}

.loading i {
  font-size: 40px;
  color: #ffd700;
}

/* ===== نافذة اللعبة ===== */
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
  background: #1e2335;
  padding: 15px 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  color: white;
  border-bottom: 2px solid #ffd700;
}

.close-button {
  background: #2a2f42;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #ffd700;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.close-button:hover {
  background: #ffd700;
  color: #1e2335;
  transform: rotate(90deg);
}

.game-modal-header h2 {
  margin: 0;
  font-size: 20px;
  flex: 1;
  color: #ffd700;
}

.game-modal-content {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

/* ===== تحسينات الجوال ===== */
@media (max-width: 768px) {
  .games-page {
    padding: 10px;
  }

  .balance-card {
    font-size: 18px;
    padding: 12px 25px;
  }

  .game-row {
    grid-template-columns: 1fr;
  }

  .game-title-large {
    font-size: 22px;
  }

  .game-price {
    font-size: 24px;
  }

  .game-image-wrapper {
    height: 150px;
  }

  .extra-games-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .balance-card {
    font-size: 16px;
    padding: 10px 20px;
  }

  .game-header {
    flex-direction: column;
    gap: 10px;
    text-align: center;
  }

  .game-image-wrapper {
    height: 120px;
  }

  .game-icon-large {
    font-size: 60px;
  }

  .extra-games-grid {
    grid-template-columns: 1fr;
  }

  .result-message {
    font-size: 14px;
    padding: 10px 20px;
    top: 80px;
  }
}
</style>
