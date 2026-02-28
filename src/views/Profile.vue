<template>
  <div class="profile-wrapper">
    <h2 class="title">
      <span class="title-gold">حسابي</span>
      <span class="title-icon">👤</span>
    </h2>

    <div v-if="loading" class="loading-container">
      <div class="gold-spinner"></div>
      <p class="loading-text">جاري تحميل بيانات الحساب...</p>
    </div>

    <div v-else class="profile-box">
      <!-- صورة المستخدم مع تأثير ذهبي -->
      <div class="avatar-container">
        <div class="avatar-glow"></div>
        <div class="avatar">
          {{ userData.username ? userData.username.charAt(0).toUpperCase() : 'U' }}
        </div>
        <div class="avatar-badge" v-if="userData.vipLevel">
          VIP {{ userData.vipLevel }}
        </div>
      </div>

      <h3 class="username">{{ userData.username || "المستخدم" }}</h3>

      <!-- البريد الإلكتروني -->
      <div class="info-card">
        <div class="info-header">
          <i class="fas fa-envelope"></i>
          <span class="info-label">البريد الإلكتروني</span>
        </div>
        <div class="info-content">
          <span class="info-value">{{ userData.email }}</span>
          <button class="copy-btn" @click="copy(userData.email)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- معرف المستخدم -->
      <div class="info-card">
        <div class="info-header">
          <i class="fas fa-id-card"></i>
          <span class="info-label">المعرّف (ID)</span>
        </div>
        <div class="info-content">
          <span class="info-value id-value">{{ userData.uid }}</span>
          <button class="copy-btn" @click="copy(userData.uid)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- كود الإحالة -->
      <div class="info-card" v-if="userData.referralCode">
        <div class="info-header">
          <i class="fas fa-link"></i>
          <span class="info-label">كود الإحالة</span>
        </div>
        <div class="info-content">
          <span class="info-value referral-code">{{ userData.referralCode }}</span>
          <button class="copy-btn" @click="copy(userData.referralCode)" title="نسخ">
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>

      <!-- تاريخ التسجيل -->
      <div class="info-card">
        <div class="info-header">
          <i class="fas fa-calendar-alt"></i>
          <span class="info-label">تاريخ التسجيل</span>
        </div>
        <div class="info-content">
          <span class="info-value">{{ formattedDate }}</span>
        </div>
      </div>

      <!-- الرصيد -->
      <div class="info-card balance-card">
        <div class="info-header">
          <i class="fas fa-coins"></i>
          <span class="info-label">الرصيد المتاح</span>
        </div>
        <div class="info-content">
          <span class="balance-value">{{ userData.balance }} USDT</span>
        </div>
      </div>

      <!-- إحصائيات سريعة -->
      <div class="stats-grid" v-if="userData.vipLevel || userData.totalReferrals">
        <div class="stat-item" v-if="userData.vipLevel">
          <i class="fas fa-crown"></i>
          <span class="stat-label">مستوى VIP</span>
          <span class="stat-number">{{ userData.vipLevel }}</span>
        </div>
        <div class="stat-item" v-if="userData.totalReferrals">
          <i class="fas fa-users"></i>
          <span class="stat-label">الإحالات</span>
          <span class="stat-number">{{ userData.totalReferrals }}</span>
        </div>
      </div>

      <!-- الأزرار -->
      <div class="actions">
        <button class="btn btn-gold" @click="changePassword">
          <i class="fas fa-key"></i>
          تغيير كلمة المرور
        </button>

        <button class="btn btn-gold-outline" @click="copyReferralLink" v-if="userData.referralCode">
          <i class="fas fa-share-alt"></i>
          نسخ رابط الدعوة
        </button>

        <button class="btn btn-danger" @click="logout">
          <i class="fas fa-sign-out-alt"></i>
          تسجيل الخروج
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc } from "firebase/firestore";
import { onAuthStateChanged, signOut } from "firebase/auth";

export default {
  name: "Profile",

  data() {
    return {
      loading: true,
      userData: {
        email: "",
        uid: "",
        createdAt: "",
        balance: 0,
        username: "",
        referralCode: "",
        vipLevel: 0,
        totalReferrals: 0,
      },
    };
  },

  computed: {
    formattedDate() {
      if (!this.userData.createdAt) return "غير متوفر";

      let date;

      if (this.userData.createdAt.toDate) {
        date = this.userData.createdAt.toDate();
      } else {
        date = new Date(this.userData.createdAt);
      }

      return isNaN(date.getTime())
        ? "غير متوفر"
        : date.toLocaleDateString("ar-EG", {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
          });
    },

    referralLink() {
      return `${window.location.origin}/register?ref=${this.userData.referralCode}`;
    }
  },

  created() {
    this.loadUserData();
  },

  methods: {
    async loadUserData() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.loading = false;
          this.$router.push("/login");
          return;
        }

        try {
          const snap = await getDoc(doc(db, "users", user.uid));

          if (snap.exists()) {
            const data = snap.data();

            this.userData = {
              email: data.email || user.email,
              uid: user.uid,
              createdAt: data.createdAt || user.metadata.creationTime,
              balance: data.balance ?? 0,
              username: data.username || user.email.split("@")[0],
              referralCode: data.referralCode || user.uid.substring(0, 6),
              vipLevel: data.vipLevel || 0,
              totalReferrals: data.totalReferrals || 0,
            };
          } else {
            // بيانات افتراضية إذا لم يوجد المستند
            this.userData = {
              email: user.email,
              uid: user.uid,
              createdAt: user.metadata.creationTime,
              balance: 0,
              username: user.email.split("@")[0],
              referralCode: user.uid.substring(0, 6),
              vipLevel: 0,
              totalReferrals: 0,
            };
          }
        } catch (err) {
          console.error("Error loading profile:", err);
          this.showErrorMessage("حدث خطأ في تحميل البيانات");
        }

        this.loading = false;
      });
    },

    copy(text) {
      navigator.clipboard.writeText(text);
      this.showSuccessMessage("تم النسخ ✓");
    },

    copyReferralLink() {
      if (this.referralLink) {
        this.copy(this.referralLink);
        this.showSuccessMessage("تم نسخ رابط الدعوة");
      }
    },

    changePassword() {
      // يمكن إضافة وظيفة تغيير كلمة المرور لاحقاً
      this.showInfoMessage("سيتم إضافة تغيير كلمة المرور قريباً");
    },

    async logout() {
      try {
        await signOut(auth);
        this.$router.push("/login");
      } catch (err) {
        console.error("Logout error:", err);
        this.showErrorMessage("حدث خطأ في تسجيل الخروج");
      }
    },

    // دوال مساعدة للإشعارات (يمكن تطويرها لاحقاً)
    showSuccessMessage(msg) {
      alert(msg);
    },

    showErrorMessage(msg) {
      alert(msg);
    },

    showInfoMessage(msg) {
      alert(msg);
    }
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.profile-wrapper {
  min-height: 100vh;
  background: #0A0C10;
  padding: 30px 20px;
  direction: rtl;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* العنوان */
.title {
  font-size: 32px;
  font-weight: 800;
  margin-bottom: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
}

.title-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.title-icon {
  font-size: 36px;
  filter: drop-shadow(0 0 10px rgba(212, 175, 55, 0.3));
}

/* ===== بطاقة الملف الشخصي ===== */
.profile-box {
  background: #11151C;
  width: 100%;
  max-width: 500px;
  border-radius: 30px;
  padding: 30px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba(212, 175, 55, 0.1);
  position: relative;
  overflow: hidden;
}

.profile-box::before {
  content: '';
  position: absolute;
  top: -50%;
  right: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.03) 0%, transparent 70%);
  animation: rotate 30s linear infinite;
  pointer-events: none;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* ===== صورة المستخدم ===== */
.avatar-container {
  position: relative;
  width: 120px;
  height: 120px;
  margin: 0 auto 20px;
}

.avatar-glow {
  position: absolute;
  top: -5px;
  left: -5px;
  right: -5px;
  bottom: -5px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  border-radius: 50%;
  filter: blur(10px);
  opacity: 0.5;
  animation: glowPulse 2s ease-in-out infinite;
}

@keyframes glowPulse {
  0%, 100% { opacity: 0.5; transform: scale(1); }
  50% { opacity: 0.8; transform: scale(1.05); }
}

.avatar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 48px;
  font-weight: 700;
  color: #D4AF37;
  border: 3px solid #D4AF37;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  z-index: 1;
}

.avatar-badge {
  position: absolute;
  bottom: 0;
  right: -10px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 5px 12px;
  border-radius: 50px;
  font-size: 12px;
  font-weight: 700;
  border: 2px solid #0A0C10;
  z-index: 2;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.3);
}

/* اسم المستخدم */
.username {
  text-align: center;
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 25px;
  color: #D4AF37;
  border-bottom: 1px solid rgba(212, 175, 55, 0.2);
  padding-bottom: 15px;
}

/* ===== بطاقات المعلومات ===== */
.info-card {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 15px;
  margin-bottom: 15px;
  border: 1px solid rgba(212, 175, 55, 0.1);
  transition: all 0.3s ease;
}

.info-card:hover {
  border-color: #D4AF37;
  transform: translateX(-5px);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.1);
}

.info-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
  color: #D4AF37;
}

.info-header i {
  font-size: 18px;
}

.info-label {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
}

.info-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
}

.info-value {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  word-break: break-all;
  flex: 1;
}

.id-value {
  font-family: monospace;
  font-size: 14px;
  background: rgba(0, 0, 0, 0.2);
  padding: 5px 10px;
  border-radius: 8px;
}

.referral-code {
  font-family: monospace;
  font-size: 16px;
  color: #D4AF37;
  letter-spacing: 1px;
}

/* بطاقة الرصيد */
.balance-card {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border: 2px solid #D4AF37;
  margin: 20px 0;
}

.balance-value {
  font-size: 24px;
  font-weight: 800;
  color: #D4AF37;
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.3);
}

/* زر النسخ */
.copy-btn {
  background: rgba(212, 175, 55, 0.1);
  border: 1px solid rgba(212, 175, 55, 0.3);
  color: #D4AF37;
  width: 36px;
  height: 36px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.copy-btn:hover {
  background: #D4AF37;
  color: #0A0C10;
  transform: scale(1.1);
}

/* ===== شبكة الإحصائيات ===== */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin: 20px 0;
}

.stat-item {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 15px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.1);
}

.stat-item i {
  font-size: 24px;
  color: #D4AF37;
  margin-bottom: 5px;
}

.stat-label {
  display: block;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 5px;
}

.stat-number {
  display: block;
  font-size: 20px;
  font-weight: 800;
  color: #D4AF37;
}

/* ===== الأزرار ===== */
.actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 25px;
}

.btn {
  padding: 14px 20px;
  border-radius: 14px;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  width: 100%;
}

.btn-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.btn-gold:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
}

.btn-gold-outline {
  background: transparent;
  border: 2px solid #D4AF37;
  color: #D4AF37;
}

.btn-gold-outline:hover {
  background: #D4AF37;
  color: #0A0C10;
  transform: translateY(-2px);
}

.btn-danger {
  background: rgba(255, 75, 75, 0.1);
  border: 2px solid #ff4b4b;
  color: #ff4b4b;
}

.btn-danger:hover {
  background: #ff4b4b;
  color: #ffffff;
  transform: translateY(-2px);
}

/* ===== حالات التحميل ===== */
.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 60px 0;
}

.gold-spinner {
  width: 60px;
  height: 60px;
  border: 4px solid rgba(212, 175, 55, 0.1);
  border-top: 4px solid #D4AF37;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-text {
  color: #D4AF37;
  font-size: 18px;
  font-weight: 600;
}

/* ===== تحسينات للجوال ===== */
@media (max-width: 480px) {
  .profile-wrapper {
    padding: 20px 15px;
  }

  .title {
    font-size: 28px;
  }

  .profile-box {
    padding: 20px;
  }

  .avatar-container {
    width: 100px;
    height: 100px;
  }

  .avatar {
    font-size: 40px;
  }

  .username {
    font-size: 20px;
  }

  .info-value {
    font-size: 14px;
  }

  .balance-value {
    font-size: 20px;
  }

  .stats-grid {
    gap: 10px;
  }

  .stat-item {
    padding: 12px;
  }

  .btn {
    padding: 12px 15px;
    font-size: 15px;
  }
}

/* ===== تأثيرات إضافية ===== */
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.balance-card {
  animation: pulse 2s ease-in-out infinite;
}

/* تخصيص شريط التمرير */
.info-value::-webkit-scrollbar {
  height: 4px;
}

.info-value::-webkit-scrollbar-track {
  background: #1A1F2A;
  border-radius: 4px;
}

.info-value::-webkit-scrollbar-thumb {
  background: #D4AF37;
  border-radius: 4px;
}
</style>
