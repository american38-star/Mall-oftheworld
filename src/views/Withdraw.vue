<template>
  <div class="withdraw-page">
    <div class="card">
      <div class="card-header">
        <h2 class="title">
          <i class="fas fa-hand-holding-usd"></i>
          سحب الأرباح
          <span class="title-glow">USDT</span>
        </h2>
        <div class="header-glow"></div>
        <p class="sub">قم بإدخال معلومات السحب الخاصة بك</p>
      </div>

      <!-- رصيد المستخدم -->
      <div class="balance-box">
        <div class="balance-icon">
          <i class="fas fa-wallet"></i>
        </div>
        <div class="balance-info">
          <span class="balance-label">رصيدك الحالي</span>
          <span class="balance-value">{{ balance }} <span class="balance-currency">USDT</span></span>
        </div>
      </div>

      <!-- حالة VIP والمتطلبات -->
      <div v-if="userVip" class="vip-status-box">
        <div class="vip-badge">
          <i class="fas fa-crown"></i>
          مستوى VIP {{ userVip.level }}
        </div>
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': balance >= getWithdrawLimit }"></i>
          <span>الحد الأدنى للسحب: <strong>{{ getWithdrawLimit }} USDT</strong></span>
        </div>
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': isAllowedDay }"></i>
          <span>يوم السحب: <strong>{{ getWithdrawDay }}</strong></span>
        </div>
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': balance >= getWithdrawLimit && isAllowedDay }"></i>
          <span>الحد الأقصى للسحب: <strong>{{ getWithdrawLimit }} USDT</strong> (مرة واحدة فقط)</span>
        </div>
        
        <div class="progress-container">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
          </div>
          <div class="progress-text">
            {{ balance }} / {{ getWithdrawLimit }} USDT
          </div>
        </div>
      </div>

      <div v-else class="vip-status-box error">
        <i class="fas fa-exclamation-triangle"></i>
        يجب أن يكون لديك اشتراك VIP للسحب
      </div>

      <!-- مبلغ السحب -->
      <div class="input-group">
        <label>
          <i class="fas fa-coins"></i>
          المبلغ
        </label>
        <div class="amount-input-wrapper">
          <input 
            type="number" 
            v-model="amount" 
            placeholder="0.00" 
            class="gold-input"
            min="0"
            step="0.01"
          />
          <span class="input-currency">USDT</span>
        </div>
      </div>

      <!-- الشبكة -->
      <div class="input-group">
        <label>
          <i class="fas fa-network-wired"></i>
          اختر الشبكة
        </label>
        <div class="select-wrapper">
          <select v-model="selectedNetwork" class="gold-select">
            <option disabled value="">اختر الشبكة المناسبة</option>
            <option v-for="net in networks" :key="net" :value="net">
              {{ net }}
            </option>
          </select>
          <i class="fas fa-chevron-down select-arrow"></i>
        </div>
      </div>

      <!-- عنوان المحفظة -->
      <div class="input-group">
        <label>
          <i class="fas fa-qrcode"></i>
          عنوان المحفظة
        </label>
        <input 
          type="text" 
          v-model="wallet" 
          placeholder="أدخل عنوان محفظتك USDT" 
          class="gold-input"
        />
      </div>

      <!-- ملخص السحب -->
      <div class="summary-box" v-if="amount && selectedNetwork && wallet">
        <h3>📋 ملخص الطلب</h3>
        <div class="summary-item">
          <span>المبلغ:</span>
          <span class="summary-value">{{ Number(amount).toFixed(2) }} USDT</span>
        </div>
        <div class="summary-item">
          <span>الشبكة:</span>
          <span class="summary-value">{{ selectedNetwork }}</span>
        </div>
        <div class="summary-item">
          <span>العنوان:</span>
          <span class="summary-value address">{{ wallet.substring(0, 15) }}...</span>
        </div>
        <div class="summary-item">
          <span>مستوى VIP:</span>
          <span class="summary-value">{{ userVip ? userVip.level : 'لا يوجد' }}</span>
        </div>
        <div class="summary-item total">
          <span>سيتم خصم:</span>
          <span class="summary-value">{{ Number(amount).toFixed(2) }} USDT</span>
        </div>
      </div>

      <!-- تحذيرات -->
      <div class="warning-box">
        <i class="fas fa-shield-alt"></i>
        <div class="warning-text">
          <p>يرجى التأكد من صحة المعلومات قبل الإرسال</p>
          <p class="small">سيتم خصم المبلغ من رصيدك فوراً عند تقديم الطلب</p>
        </div>
      </div>

      <!-- رسائل الخطأ/التحذير -->
      <div v-if="withdrawError" class="message error">
        <i class="fas fa-exclamation-circle"></i>
        {{ withdrawError }}
      </div>

      <!-- زر السحب -->
      <button 
        class="gold-button" 
        @click="submitWithdraw"
        :disabled="!canWithdraw"
      >
        <i class="fas fa-paper-plane"></i>
        {{ isProcessing ? 'جاري المعالجة...' : 'سحب الآن' }}
      </button>

      <!-- رسائل النجاح -->
      <transition name="fade">
        <div v-if="message" class="message" :class="messageType">
          <i :class="messageType === 'error' ? 'fas fa-exclamation-circle' : 'fas fa-check-circle'"></i>
          {{ message }}
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  doc,
  getDoc,
  runTransaction,
  collection,
  serverTimestamp,
  addDoc,
  query,
  where,
  getDocs
} from "firebase/firestore";

export default {
  name: "Withdraw",

  data() {
    return {
      balance: 0,
      amount: "",
      wallet: "",
      selectedNetwork: "",
      networks: ["TRC20", "ERC20", "BEP20", "SOL"],
      isProcessing: false,
      message: "",
      messageType: "info",
      userVip: null,
      withdrawError: "",
    };
  },

  computed: {
    canWithdraw() {
      if (!this.userVip) return false;
      if (!this.selectedNetwork) return false;
      if (!this.wallet || this.wallet.length < 20) return false;
      if (!this.amount || this.amount <= 0) return false;
      if (this.isProcessing) return false;
      if (this.balance < this.getWithdrawLimit) return false;
      if (Number(this.amount) !== this.getWithdrawLimit) return false;
      if (!this.isAllowedDay) return false;
      
      return true;
    },

    getWithdrawLimit() {
      if (!this.userVip) return 0;
      const limits = {
        1: 5,
        2: 25,
        3: 50,
        4: 160,
        5: 530,
        6: 820,
        7: 1120,
        8: 2400,
        9: 5300,
        10: 11300,
        11: 26000,
        12: 56000,
        13: 120000,
        14: 260000
      };
      return limits[this.userVip.level] || 0;
    },

    getWithdrawDay() {
      if (!this.userVip) return "";
      const days = {
        1: "السبت",
        2: "السبت",
        3: "الأحد",
        4: "الأحد",
        5: "الاثنين",
        6: "الاثنين",
        7: "الثلاثاء",
        8: "الثلاثاء",
        9: "الأربعاء",
        10: "الأربعاء",
        11: "الخميس",
        12: "الخميس",
        13: "الجمعة",
        14: "الجمعة"
      };
      return days[this.userVip.level] || "";
    },

    isAllowedDay() {
      if (!this.userVip) return false;
      
      const dayMapping = {
        "السبت": "Saturday",
        "الأحد": "Sunday",
        "الاثنين": "Monday",
        "الثلاثاء": "Tuesday",
        "الأربعاء": "Wednesday",
        "الخميس": "Thursday",
        "الجمعة": "Friday"
      };

      const todayEnglish = new Date().toLocaleDateString("en-US", { weekday: "long" });
      const allowedDayEnglish = dayMapping[this.getWithdrawDay];
      
      return todayEnglish === allowedDayEnglish;
    },

    progressPercentage() {
      if (!this.userVip || this.getWithdrawLimit === 0) return 0;
      return Math.min((this.balance / this.getWithdrawLimit) * 100, 100);
    }
  },

  async created() {
    await this.loadBalance();
    await this.loadUserVip();
  },

  methods: {
    async loadBalance() {
      const user = auth.currentUser;
      if (!user) {
        this.$router.push("/login");
        return;
      }

      const ref = doc(db, "users", user.uid);
      const snap = await getDoc(ref);

      if (snap.exists()) {
        this.balance = snap.data().balance ?? 0;
      }
    },

    async loadUserVip() {
      const user = auth.currentUser;
      if (!user) return;

      const vipDocRef = doc(db, "users", user.uid, "vip", "current");
      const vipSnap = await getDoc(vipDocRef);
      
      if (vipSnap.exists()) {
        const data = vipSnap.data();
        this.userVip = {
          level: data.level || null,
          daily: data.daily || 0,
        };
      }
    },

    async checkExistingWithdrawToday() {
      const user = auth.currentUser;
      if (!user) return false;

      const today = new Date();
      today.setHours(0, 0, 0, 0);
      
      const tomorrow = new Date(today);
      tomorrow.setDate(tomorrow.getDate() + 1);

      const withdrawQuery = query(
        collection(db, "withdraw_requests"),
        where("userId", "==", user.uid),
        where("createdAt", ">=", today),
        where("createdAt", "<", tomorrow)
      );

      const snapshot = await getDocs(withdrawQuery);
      return !snapshot.empty;
    },

    checkWithdrawConditions() {
      this.withdrawError = "";

      if (!this.userVip) {
        this.withdrawError = "يجب أن يكون لديك اشتراك VIP للسحب";
        return false;
      }

      if (this.balance < this.getWithdrawLimit) {
        this.withdrawError = `⚠️ يجب أن يصل رصيدك إلى ${this.getWithdrawLimit} USDT على الأقل للسحب`;
        return false;
      }

      if (Number(this.amount) !== this.getWithdrawLimit) {
        this.withdrawError = `⚠️ مبلغ السحب يجب أن يكون بالضبط ${this.getWithdrawLimit} USDT`;
        return false;
      }

      if (!this.isAllowedDay) {
        this.withdrawError = `⚠️ السحب متاح فقط يوم ${this.getWithdrawDay}`;
        return false;
      }

      return true;
    },

    async submitWithdraw() {
      this.message = "";
      this.withdrawError = "";

      if (!this.checkWithdrawConditions()) {
        return;
      }

      if (!this.selectedNetwork) {
        this.showMessage("الرجاء اختيار الشبكة", "error");
        return;
      }

      if (!this.wallet || this.wallet.length < 20) {
        this.showMessage("الرجاء إدخال عنوان محفظة صحيح", "error");
        return;
      }

      const user = auth.currentUser;
      if (!user) {
        this.showMessage("الرجاء تسجيل الدخول من جديد", "error");
        return;
      }

      const hasWithdrawToday = await this.checkExistingWithdrawToday();
      if (hasWithdrawToday) {
        this.showMessage("⚠️ لقد قمت بالسحب مسبقاً اليوم. يمكنك السحب مرة واحدة فقط في اليوم المحدد", "error");
        return;
      }

      this.isProcessing = true;

      const userRef = doc(db, "users", user.uid);
      const withdrawRef = collection(db, "withdraw_requests");

      try {
        await runTransaction(db, async (tx) => {
          const userSnap = await tx.get(userRef);
          if (!userSnap.exists()) throw new Error("User not found");

          const userData = userSnap.data();

          if (userData.blocked === true) {
            throw new Error("🚫 حسابك محظور من السحب!");
          }

          const currentBalance = Number(userData.balance || 0);
          const amountNum = Number(this.amount);

          if (amountNum > currentBalance) {
            throw new Error("المبلغ أكبر من رصيدك!");
          }

          if (amountNum !== this.getWithdrawLimit) {
            throw new Error(`مبلغ السحب يجب أن يكون بالضبط ${this.getWithdrawLimit} USDT`);
          }

          tx.update(userRef, {
            balance: currentBalance - amountNum
          });

          const newReq = doc(withdrawRef);
          tx.set(newReq, {
            userId: user.uid,
            email: user.email,
            amount: amountNum,
            wallet: this.wallet,
            network: this.selectedNetwork,
            status: "pending",
            createdAt: serverTimestamp(),
            oldBalance: currentBalance,
            vipLevel: this.userVip?.level || null,
            withdrawDay: this.getWithdrawDay,
            withdrawLimit: this.getWithdrawLimit
          });
        });

        await addDoc(collection(db, "transactions"), {
          userId: user.uid,
          email: user.email,
          type: "withdraw",
          amount: Number(this.amount),
          wallet: this.wallet,
          network: this.selectedNetwork,
          status: "pending",
          reason: "",
          adminMessage: "",
          createdAt: serverTimestamp(),
          vipLevel: this.userVip?.level || null
        });

        this.showMessage("✅ تم إرسال طلب السحب بنجاح", "success");
        
        this.balance -= Number(this.amount);
        
        setTimeout(() => {
          this.amount = "";
          this.wallet = "";
          this.selectedNetwork = "";
          this.message = "";
        }, 3000);

      } catch (e) {
        this.showMessage("خطأ: " + e.message, "error");
      } finally {
        this.isProcessing = false;
      }
    },

    showMessage(msg, type) {
      this.message = msg;
      this.messageType = type;
    },
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.withdraw-page {
  min-height: 100vh;
  background: #0A0C10;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  direction: rtl;
  font-family: 'Cairo', sans-serif;
}

/* البطاقة الرئيسية */
.card {
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

.card::before {
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

/* رأس البطاقة */
.card-header {
  position: relative;
  margin-bottom: 25px;
  text-align: center;
}

.title {
  font-size: 28px;
  font-weight: 800;
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  position: relative;
  z-index: 1;
  margin-bottom: 8px;
}

.title i {
  color: #D4AF37;
  font-size: 32px;
}

.title-glow {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

.header-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.1) 0%, transparent 70%);
  filter: blur(30px);
  z-index: 0;
}

.sub {
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  margin-top: 5px;
  position: relative;
  z-index: 1;
}

/* صندوق الرصيد */
.balance-box {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  gap: 15px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.balance-icon {
  width: 50px;
  height: 50px;
  background: rgba(212, 175, 55, 0.1);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #D4AF37;
}

.balance-icon i {
  font-size: 24px;
  color: #D4AF37;
}

.balance-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.balance-label {
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  margin-bottom: 4px;
}

.balance-value {
  font-size: 28px;
  font-weight: 800;
  color: #D4AF37;
}

.balance-currency {
  font-size: 16px;
  color: rgba(255, 255, 255, 0.5);
  margin-right: 5px;
}

/* صندوق حالة VIP */
.vip-status-box {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 25px;
  border: 1px solid #D4AF37;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.2);
}

.vip-status-box.error {
  border-color: #ef4444;
  text-align: center;
  color: #ef4444;
}

.vip-status-box.error i {
  font-size: 24px;
  margin-bottom: 10px;
}

.vip-badge {
  display: inline-block;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 8px 16px;
  border-radius: 50px;
  font-weight: 700;
  font-size: 14px;
  margin-bottom: 15px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.vip-badge i {
  margin-left: 5px;
}

.withdraw-condition {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 10px 0;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
}

.withdraw-condition i {
  color: #666;
  font-size: 16px;
}

.withdraw-condition i.condition-met {
  color: #22c55e;
}

.withdraw-condition strong {
  color: #D4AF37;
}

.progress-container {
  margin-top: 15px;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 5px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #D4AF37, #F6E27A);
  border-radius: 10px;
  transition: width 0.3s ease;
}

.progress-text {
  text-align: center;
  color: #D4AF37;
  font-size: 12px;
  font-weight: 600;
}

/* مجموعات الإدخال */
.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #D4AF37;
  font-size: 15px;
  font-weight: 600;
  margin-bottom: 8px;
}

.input-group label i {
  font-size: 16px;
}

/* الحقول الذهبية */
.gold-input {
  width: 100%;
  padding: 14px 20px;
  border-radius: 16px;
  background: #1A1F2A;
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 15px;
  transition: all 0.3s ease;
}

.gold-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
}

.gold-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

/* حقل المبلغ مع العملة */
.amount-input-wrapper {
  position: relative;
}

.input-currency {
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  color: #D4AF37;
  font-weight: 700;
  font-size: 15px;
  background: rgba(212, 175, 55, 0.1);
  padding: 4px 12px;
  border-radius: 20px;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

/* القائمة المنسدلة */
.select-wrapper {
  position: relative;
}

.gold-select {
  width: 100%;
  padding: 14px 20px;
  border-radius: 16px;
  background: #1A1F2A;
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  appearance: none;
}

.gold-select:hover, .gold-select:focus {
  border-color: #D4AF37;
  outline: none;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
}

.select-arrow {
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  color: #D4AF37;
  pointer-events: none;
  font-size: 14px;
}

/* صندوق الملخص */
.summary-box {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 20px;
  margin: 20px 0;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.summary-box h3 {
  color: #D4AF37;
  font-size: 16px;
  margin-bottom: 15px;
  text-align: center;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
  color: rgba(255, 255, 255, 0.8);
}

.summary-item:last-child {
  border-bottom: none;
}

.summary-item.total {
  margin-top: 10px;
  padding-top: 15px;
  border-top: 2px solid #D4AF37;
  font-weight: 700;
  color: #D4AF37;
}

.summary-value {
  font-weight: 600;
  color: #D4AF37;
}

.summary-value.address {
  font-family: monospace;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.7);
}

/* صندوق التحذير */
.warning-box {
  background: rgba(212, 175, 55, 0.05);
  border-right: 4px solid #D4AF37;
  padding: 15px;
  border-radius: 12px;
  margin: 20px 0;
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.warning-box i {
  color: #D4AF37;
  font-size: 20px;
  margin-top: 2px;
}

.warning-text p {
  color: rgba(255, 255, 255, 0.8);
  font-size: 14px;
  margin: 0 0 5px 0;
}

.warning-text .small {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
}

/* الرسائل */
.message {
  margin-top: 15px;
  padding: 15px 20px;
  border-radius: 16px;
  font-size: 14px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 10px;
  border: 1px solid;
}

.message.success {
  background: rgba(34, 197, 94, 0.1);
  color: #22c55e;
  border-color: #22c55e;
}

.message.error {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
  border-color: #ef4444;
}

.message i {
  font-size: 18px;
}

/* الزر الرئيسي */
.gold-button {
  width: 100%;
  padding: 16px;
  border-radius: 20px;
  border: none;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  font-size: 18px;
  font-weight: 800;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin: 20px 0;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.gold-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.4);
}

.gold-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(20%);
}

/* حركة الرسائل */
.fade-enter-active, .fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* تحسينات للجوال */
@media (max-width: 480px) {
  .card {
    padding: 20px;
  }

  .title {
    font-size: 24px;
  }

  .title i {
    font-size: 28px;
  }

  .balance-value {
    font-size: 24px;
  }

  .gold-input, .gold-select {
    padding: 12px 15px;
    font-size: 14px;
  }

  .input-currency {
    left: 15px;
    padding: 3px 10px;
    font-size: 13px;
  }

  .summary-box {
    padding: 15px;
  }

  .warning-box {
    padding: 12px;
  }

  .warning-box i {
    font-size: 18px;
  }

  .warning-text p {
    font-size: 13px;
  }

  .gold-button {
    padding: 14px;
    font-size: 16px;
  }
}

/* إزالة أزرار الزيادة/النقص من input number */
input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}
</style>
