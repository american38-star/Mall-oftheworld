<template>
  <div class="home-container">

    <!-- الشريط العلوي -->
    <div class="top-bar">
      <div class="top-left">
        <i class="fas fa-headset icon"></i>
        <i class="fas fa-envelope icon"></i>
      </div>

      <div class="user-box">
        <div class="welcome">مرحباً، {{ username }}</div>
        <div class="balance">الرصيد: <strong>{{ balance }} USDT</strong></div>
      </div>
    </div>

    <!-- شريط الأزرار -->
    <div class="quick-buttons">
      <button class="quick-btn">
        <span class="btn-icon">⏱</span>
        وفر الوقت
      </button>
      <button class="quick-btn">
        <span class="btn-icon">💰</span>
        وفر المال
      </button>
    </div>

    <!-- شريط الإعلان -->
    <div class="notice-bar">
      <span class="notice-icon">🔊</span>
      شراكة عالمية مع Amazon, eBay, Etsy, Walmart وأكثر!
    </div>

    <!-- القائمة الرئيسية -->
    <div class="grid-menu">
      <div
        class="item"
        v-for="item in menu"
        :key="item.title"
        @click="go(item.route)"
      >
        <div class="icon-box">
          <i :class="item.icon"></i>
        </div>
        <p>{{ item.title }}</p>
      </div>

      <!-- زر الشركة لعرض الإعلان -->
      <div class="item company-item" @click="showCompanyAd">
        <div class="icon-box gold-glow">
          <i class="fas fa-building"></i>
        </div>
        <p>الشركة</p>
      </div>
    </div>

    <!-- تم إزالة البانر بالكامل -->

    <!-- إعلان Popup فاخر -->
    <div id="companyAd" class="ad-overlay" v-if="showAd" @click.self="closeAd">
      <div class="ad-box">
        <div class="ad-header">
          <h2>✨ إعلان الشركة ✨</h2>
          <button class="close-btn" @click="closeAd">×</button>
        </div>

        <div class="ad-content">
          <p>
            🎉🎉🎉🎉 مرحبا بالجميع! تأسست Mall of the world في سنغافورة في 20 أغسطس 2021 ومقرها حاليًا في منطقة الأعمال المركزية في سنغافورة. نحن شركة استثمار في التجارة الإلكترونية مع فريق تقني قوي وقوة مالية قوية.
            <br><br>
            يتعاون Mall of the world مع عشرات شركات التجارة الإلكترونية مثل Amazon و eBay و Tiktok و Aliexpress و Alibaba و Shopee ، إلخ. لمساعدة التجار على زيادة مبيعات المنتجات الخاصة بهم ، ويمكننا أيضًا تحقيق أرباح منه.
          </p>
          
          <div class="vip-table">
            <h3>🎯 خطط الاستثمار VIP</h3>
            <div class="vip-row" v-for="vip in vipPlans" :key="vip.level">
              <span class="vip-level">{{ vip.level }}</span>
              <span class="vip-recharge">💵 {{ vip.recharge }} USDT</span>
              <span class="vip-daily">📈 {{ vip.daily }} USDT/يوم</span>
            </div>
          </div>

          <div class="commission-info">
            <h3>🤝 نظام العمولات</h3>
            <p>المستوى 1: 6% | المستوى 2: 2% | المستوى 3: 1%</p>
          </div>
        </div>

        <button class="ad-btn" @click="closeAd">أنا أعرف</button>
      </div>
    </div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc } from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Home",

  data() {
    return {
      username: "جاري التحميل...",
      balance: 0,
      showAd: false,
      
      vipPlans: [
        { level: 'VIP1', recharge: '12', daily: '3' },
        { level: 'VIP2', recharge: '52', daily: '13' },
        { level: 'VIP3', recharge: '100', daily: '26' },
        { level: 'VIP4', recharge: '300', daily: '82' },
        { level: 'VIP5', recharge: '500', daily: '145' },
        { level: 'VIP6', recharge: '1500', daily: '479' },
        { level: 'VIP7', recharge: '3000', daily: '1078' },
        { level: 'VIP8', recharge: '5000', daily: '2000' },
        { level: 'VIP9', recharge: '10000', daily: '5000' },
        { level: 'VIP10', recharge: '30000', daily: '17699' },
        { level: 'VIP11', recharge: '90000', daily: '81818' }
      ],

      menu: [
        { title: "تعبئة رصيد", icon: "fas fa-coins", route: "/recharge" },
        { title: "سحب", icon: "fas fa-wallet", route: "/withdraw" },
        { title: "المعاملات", icon: "fas fa-history", route: "/transactions" },
        { title: "برنامج", icon: "fas fa-download", route: "/program" },
        { title: "الأصدقاء", icon: "fas fa-users", route: "/team" },
        { title: "الوكالات", icon: "fas fa-id-card", route: "/agency" }
      ]
    };
  },

  created() {
    this.watchUser();
  },

  methods: {
    watchUser() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.username = "غير مسجل";
          this.balance = 0;
          this.$router.push("/login");
          return;
        }

        await this.loadUserData(user.uid, user.email);
      });
    },

    async loadUserData(uid, email) {
      try {
        const snap = await getDoc(doc(db, "users", uid));

        if (snap.exists()) {
          const data = snap.data();
          this.username = data.username || data.email || email;
          this.balance = data.balance ?? 0;
        } else {
          this.username = email;
          this.balance = 0;
        }
      } catch (err) {
        console.error("Error loading user data:", err);
        this.username = email;
        this.balance = 0;
      }
    },

    go(route) {
      this.$router.push(route);
    },

    showCompanyAd() {
      this.showAd = true;
      document.body.style.overflow = 'hidden';
    },

    closeAd() {
      this.showAd = false;
      document.body.style.overflow = 'auto';
    }
  },

  beforeUnmount() {
    document.body.style.overflow = 'auto';
  }
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.home-container {
  direction: rtl;
  padding: 16px;
  background: #0A0C10;
  min-height: 100vh;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
  padding-bottom: 80px; /* إضافة مساحة للشريط السفلي */
}

/* الشريط العلوي */
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #11151C;
  padding: 15px 20px;
  border-radius: 16px;
  margin-bottom: 20px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.top-left {
  display: flex;
  gap: 15px;
}

.icon {
  font-size: 24px;
  color: #D4AF37;
  cursor: pointer;
  transition: all 0.3s ease;
}

.icon:hover {
  color: #F6E27A;
  transform: scale(1.1);
}

.user-box {
  text-align: left;
}

.welcome {
  font-size: 16px;
  font-weight: 600;
  color: #D4AF37;
  margin-bottom: 5px;
}

.balance {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.8);
}

.balance strong {
  color: #D4AF37;
  font-size: 18px;
}

/* شريط الأزرار السريعة */
.quick-buttons {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.quick-btn {
  flex: 1;
  padding: 14px;
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border: 1px solid rgba(212, 175, 55, 0.3);
  border-radius: 14px;
  font-weight: 700;
  color: #D4AF37;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 15px;
}

.btn-icon {
  font-size: 20px;
}

.quick-btn:hover {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-color: #D4AF37;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.2);
}

/* شريط الإعلان */
.notice-bar {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  padding: 14px 20px;
  border-radius: 50px;
  text-align: center;
  margin-bottom: 25px;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.notice-icon {
  font-size: 20px;
}

/* القائمة الرئيسية */
.grid-menu {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 18px;
  margin-bottom: 20px; /* تقليل المسافة بعد إزالة البانر */
}

.item {
  background: #11151C;
  color: #ffffff;
  padding: 25px 15px;
  border-radius: 20px;
  text-align: center;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.item:hover {
  transform: translateY(-5px);
  border-color: #D4AF37;
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.2);
}

.icon-box {
  font-size: 32px;
  margin-bottom: 12px;
  color: #D4AF37;
  transition: all 0.3s ease;
}

.item:hover .icon-box {
  transform: scale(1.1);
  color: #F6E27A;
}

.gold-glow {
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
}

.company-item {
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border: 1px solid #D4AF37;
}

/* تم إزالة قسم البانر بالكامل */

/* ===== إعلان فاخر ===== */
.ad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 12, 16, 0.98);
  backdrop-filter: blur(10px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  direction: rtl;
}

.ad-box {
  background: #11151C;
  width: 90%;
  max-width: 500px;
  border-radius: 30px;
  overflow: hidden;
  border: 2px solid #D4AF37;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.7), 0 0 30px rgba(212, 175, 55, 0.3);
  animation: fadeInUp 0.5s ease;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ad-header {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
}

.ad-header h2 {
  color: #0A0C10;
  margin: 0;
  font-size: 22px;
  font-weight: 700;
  flex: 1;
  text-align: center;
}

.close-btn {
  background: rgba(10, 12, 16, 0.2);
  border: none;
  color: #0A0C10;
  font-size: 28px;
  cursor: pointer;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
  position: absolute;
  left: 10px;
  top: 10px;
}

.close-btn:hover {
  background: rgba(10, 12, 16, 0.4);
  transform: rotate(90deg);
}

.ad-content {
  max-height: 400px;
  overflow-y: auto;
  padding: 25px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  line-height: 1.8;
  scrollbar-width: thin;
  scrollbar-color: #D4AF37 #1A1F2A;
}

.ad-content::-webkit-scrollbar {
  width: 6px;
}

.ad-content::-webkit-scrollbar-track {
  background: #1A1F2A;
}

.ad-content::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #D4AF37, #C5A028);
  border-radius: 10px;
}

.vip-table {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 15px;
  margin: 20px 0;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

.vip-table h3 {
  color: #D4AF37;
  text-align: center;
  margin-bottom: 15px;
  font-size: 18px;
}

.vip-row {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
  color: #ffffff;
}

.vip-row:last-child {
  border-bottom: none;
}

.vip-level {
  color: #D4AF37;
  font-weight: 700;
}

.vip-recharge {
  color: #F6E27A;
}

.vip-daily {
  color: #ffffff;
}

.commission-info {
  text-align: center;
  margin: 20px 0;
}

.commission-info h3 {
  color: #D4AF37;
  margin-bottom: 10px;
}

.commission-info p {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
  font-size: 18px;
}

.ad-btn {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 16px 30px;
  margin: 0 25px 25px 25px;
  border-radius: 50px;
  cursor: pointer;
  font-size: 18px;
  font-weight: 700;
  transition: all 0.3s ease;
  width: calc(100% - 50px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
}

.ad-btn:hover {
  transform: scale(1.02);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.5);
  background: linear-gradient(135deg, #C5A028, #F6E27A, #D4AF37);
}

/* تحسين للهواتف */
@media (max-width: 480px) {
  .home-container {
    padding: 12px;
    padding-bottom: 70px;
  }
  
  .grid-menu {
    gap: 12px;
  }
  
  .item {
    padding: 20px 10px;
  }
  
  .icon-box {
    font-size: 28px;
  }
  
  .ad-box {
    width: 95%;
  }
  
  .ad-content {
    max-height: 350px;
    padding: 15px;
  }
  
  .vip-row {
    font-size: 13px;
  }
}

/* تأثيرات إضافية */
@keyframes goldPulse {
  0% { box-shadow: 0 0 5px rgba(212, 175, 55, 0.3); }
  50% { box-shadow: 0 0 20px rgba(212, 175, 55, 0.6); }
  100% { box-shadow: 0 0 5px rgba(212, 175, 55, 0.3); }
}

.company-item {
  animation: goldPulse 2s infinite;
}
</style>
