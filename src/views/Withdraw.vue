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
      <div v-if="userVipLevel" class="vip-status-box">
        <div class="vip-badge">
          <i class="fas fa-crown"></i>
          مستوى VIP {{ userVipLevel }}
        </div>
        
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': balance >= minWithdrawAmount }"></i>
          <span>الحد الأدنى للسحب: <strong>{{ minWithdrawAmount }} USDT</strong></span>
        </div>
        
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': isAllowedDay }"></i>
          <span>يوم السحب: <strong>{{ withdrawDay }}</strong></span>
        </div>
        
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': balance >= minWithdrawAmount && isAllowedDay }"></i>
          <span>مرة واحدة فقط في اليوم المحدد</span>
        </div>
        
        <div class="progress-container">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
          </div>
          <div class="progress-text">
            {{ balance }} / {{ minWithdrawAmount }} USDT
          </div>
        </div>
      </div>

      <div v-else class="vip-status-box error">
        <i class="fas fa-exclamation-triangle"></i>
        <p>يجب أن يكون لديك اشتراك VIP للسحب</p>
      </div>

      <!-- رسائل الخطأ والنجاح -->
      <transition name="fade">
        <div v-if="errorMessage" class="message error">
          <i class="fas fa-exclamation-circle"></i>
          {{ errorMessage }}
        </div>
      </transition>

      <transition name="fade">
        <div v-if="successMessage" class="message success">
          <i class="fas fa-check-circle"></i>
          {{ successMessage }}
        </div>
      </transition>

      <!-- مبلغ السحب -->
      <div class="input-group">
        <label>
          <i class="fas fa-coins"></i>
          المبلغ
        </label>
        <div class="amount-input-wrapper">
          <input 
            type="number" 
            v-model.number="amount" 
            placeholder="0.00" 
            class="gold-input"
            min="0"
            step="0.01"
            :class="{ 'error-input': amountError }"
          />
          <span class="input-currency">USDT</span>
        </div>
        <span v-if="amountError" class="input-error">{{ amountError }}</span>
      </div>

      <!-- الشبكة -->
      <div class="input-group">
        <label>
          <i class="fas fa-network-wired"></i>
          اختر الشبكة
        </label>
        <div class="select-wrapper">
          <select v-model="network" class="gold-select" :class="{ 'error-input': networkError }">
            <option value="">اختر الشبكة المناسبة</option>
            <option v-for="net in networks" :key="net" :value="net">
              {{ net }}
            </option>
          </select>
          <i class="fas fa-chevron-down select-arrow"></i>
        </div>
        <span v-if="networkError" class="input-error">{{ networkError }}</span>
      </div>

      <!-- عنوان المحفظة -->
      <div class="input-group">
        <label>
          <i class="fas fa-qrcode"></i>
          عنوان المحفظة
        </label>
        <input 
          type="text" 
          v-model="walletAddress" 
          placeholder="أدخل عنوان محفظتك USDT" 
          class="gold-input"
          :class="{ 'error-input': walletError }"
        />
        <span v-if="walletError" class="input-error">{{ walletError }}</span>
      </div>

      <!-- ملخص السحب -->
      <div class="summary-box" v-if="amount && network && walletAddress">
        <h3>📋 ملخص الطلب</h3>
        <div class="summary-item">
          <span>المبلغ:</span>
          <span class="summary-value">{{ Number(amount).toFixed(2) }} USDT</span>
        </div>
        <div class="summary-item">
          <span>الشبكة:</span>
          <span class="summary-value">{{ network }}</span>
        </div>
        <div class="summary-item">
          <span>العنوان:</span>
          <span class="summary-value address">{{ walletAddress.substring(0, 15) }}...</span>
        </div>
        <div class="summary-item">
          <span>مستوى VIP:</span>
          <span class="summary-value">{{ userVipLevel || 'لا يوجد' }}</span>
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

      <!-- زر السحب -->
      <button 
        class="gold-button" 
        @click="submitWithdraw"
        :disabled="isLoading || !canWithdraw"
      >
        <i class="fas fa-paper-plane" v-if="!isLoading"></i>
        <i class="fas fa-spinner fa-spin" v-else></i>
        {{ isLoading ? 'جاري المعالجة...' : 'سحب الآن' }}
      </button>

      <!-- آخر طلبات السحب -->
      <div class="recent-withdraws" v-if="recentWithdraws.length > 0">
        <h4>آخر طلبات السحب</h4>
        <div class="withdraw-item" v-for="withdraw in recentWithdraws" :key="withdraw.id">
          <span>{{ withdraw.amount }} USDT</span>
          <span class="status" :class="withdraw.status">{{ withdraw.status }}</span>
        </div>
      </div>
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
  addDoc, 
  serverTimestamp,
  query,
  where,
  orderBy,
  limit,
  getDocs,
  Timestamp
} from "firebase/firestore";

export default {
  name: "Withdraw",
  
  data() {
    return {
      balance: 0,
      amount: "",
      network: "",
      walletAddress: "",
      isLoading: false,
      errorMessage: "",
      successMessage: "",
      userVipLevel: null,
      minWithdrawAmount: 5,
      networks: ["TRC20", "ERC20", "BEP20", "SOL"],
      recentWithdraws: [],
      
      // أخطاء الحقول
      amountError: "",
      networkError: "",
      walletError: "",
      
      // حدود VIP
      vipLimits: {
        1: 5, 2: 25, 3: 50, 4: 160, 5: 530,
        6: 820, 7: 1120, 8: 2400, 9: 5300, 10: 11300,
        11: 26000, 12: 56000, 13: 120000, 14: 260000
      },
      
      // أيام السحب
      withdrawDays: {
        1: "السبت", 2: "السبت", 3: "الأحد", 4: "الأحد",
        5: "الاثنين", 6: "الاثنين", 7: "الثلاثاء", 8: "الثلاثاء",
        9: "الأربعاء", 10: "الأربعاء", 11: "الخميس", 12: "الخميس",
        13: "الجمعة", 14: "الجمعة"
      },
      
      dayMapping: {
        "السبت": "Saturday",
        "الأحد": "Sunday",
        "الاثنين": "Monday",
        "الثلاثاء": "Tuesday",
        "الأربعاء": "Wednesday",
        "الخميس": "Thursday",
        "الجمعة": "Friday"
      }
    };
  },

  computed: {
    canWithdraw() {
      return (
        this.userVipLevel &&
        this.amount &&
        this.network &&
        this.walletAddress &&
        this.walletAddress.length >= 20 &&
        Number(this.amount) === this.minWithdrawAmount &&
        this.balance >= this.minWithdrawAmount &&
        this.isAllowedDay &&
        !this.isLoading
      );
    },

    withdrawDay() {
      return this.withdrawDays[this.userVipLevel] || "";
    },

    isAllowedDay() {
      if (!this.userVipLevel) return false;
      
      const today = new Date().toLocaleDateString("en-US", { weekday: "long" });
      const allowedDay = this.withdrawDays[this.userVipLevel];
      const allowedDayEnglish = this.dayMapping[allowedDay];
      
      return today === allowedDayEnglish;
    },

    progressPercentage() {
      if (!this.userVipLevel || this.minWithdrawAmount === 0) return 0;
      return Math.min((this.balance / this.minWithdrawAmount) * 100, 100);
    }
  },

  watch: {
    amount(newVal) {
      this.validateAmount();
    },
    network() {
      this.validateNetwork();
    },
    walletAddress() {
      this.validateWallet();
    },
    userVipLevel() {
      this.setMinWithdrawAmount();
    }
  },

  async created() {
    await this.loadUserData();
    await this.loadRecentWithdraws();
  },

  methods: {
    async loadUserData() {
      const user = auth.currentUser;
      if (!user) {
        this.$router.push("/login");
        return;
      }

      try {
        // تحميل رصيد المستخدم
        const userRef = doc(db, "users", user.uid);
        const userSnap = await getDoc(userRef);
        
        if (userSnap.exists()) {
          this.balance = userSnap.data().balance || 0;
        }

        // تحميل مستوى VIP
        const vipRef = doc(db, "users", user.uid, "vip", "current");
        const vipSnap = await getDoc(vipRef);
        
        if (vipSnap.exists()) {
          this.userVipLevel = vipSnap.data().level;
        } else {
          this.errorMessage = "لا يوجد اشتراك VIP نشط";
        }
      } catch (error) {
        console.error("خطأ في تحميل البيانات:", error);
        this.errorMessage = "حدث خطأ في تحميل البيانات";
      }
    },

    async loadRecentWithdraws() {
      const user = auth.currentUser;
      if (!user) return;

      try {
        const q = query(
          collection(db, "withdraw_requests"),
          where("userId", "==", user.uid),
          orderBy("createdAt", "desc"),
          limit(3)
        );
        
        const querySnapshot = await getDocs(q);
        this.recentWithdraws = querySnapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data(),
          status: doc.data().status || "pending"
        }));
      } catch (error) {
        console.error("خطأ في تحميل طلبات السحب:", error);
      }
    },

    setMinWithdrawAmount() {
      this.minWithdrawAmount = this.vipLimits[this.userVipLevel] || 5;
    },

    validateAmount() {
      if (!this.amount) {
        this.amountError = "الرجاء إدخال المبلغ";
        return false;
      }
      
      if (this.amount <= 0) {
        this.amountError = "المبلغ يجب أن يكون أكبر من صفر";
        return false;
      }
      
      if (Number(this.amount) !== this.minWithdrawAmount) {
        this.amountError = `مبلغ السحب يجب أن يكون بالضبط ${this.minWithdrawAmount} USDT`;
        return false;
      }
      
      if (this.amount > this.balance) {
        this.amountError = "المبلغ أكبر من رصيدك";
        return false;
      }
      
      this.amountError = "";
      return true;
    },

    validateNetwork() {
      if (!this.network) {
        this.networkError = "الرجاء اختيار الشبكة";
        return false;
      }
      this.networkError = "";
      return true;
    },

    validateWallet() {
      if (!this.walletAddress) {
        this.walletError = "الرجاء إدخال عنوان المحفظة";
        return false;
      }
      
      if (this.walletAddress.length < 20) {
        this.walletError = "عنوان المحفظة غير صحيح (يجب أن يكون 20 حرف على الأقل)";
        return false;
      }
      
      this.walletError = "";
      return true;
    },

    async checkExistingWithdrawToday() {
      const user = auth.currentUser;
      if (!user) return false;

      const today = new Date();
      today.setHours(0, 0, 0, 0);
      
      const tomorrow = new Date(today);
      tomorrow.setDate(tomorrow.getDate() + 1);

      const q = query(
        collection(db, "withdraw_requests"),
        where("userId", "==", user.uid),
        where("createdAt", ">=", Timestamp.fromDate(today)),
        where("createdAt", "<", Timestamp.fromDate(tomorrow))
      );

      const snapshot = await getDocs(q);
      return !snapshot.empty;
    },

    validateAll() {
      this.errorMessage = "";
      
      if (!this.userVipLevel) {
        this.errorMessage = "يجب أن يكون لديك اشتراك VIP للسحب";
        return false;
      }

      if (!this.isAllowedDay) {
        this.errorMessage = `⚠️ السحب متاح فقط يوم ${this.withdrawDay}`;
        return false;
      }

      if (!this.validateAmount()) {
        return false;
      }

      if (!this.validateNetwork()) {
        return false;
      }

      if (!this.validateWallet()) {
        return false;
      }

      if (this.balance < this.minWithdrawAmount) {
        this.errorMessage = `⚠️ رصيدك غير كاف. تحتاج إلى ${this.minWithdrawAmount} USDT على الأقل`;
        return false;
      }

      return true;
    },

    async submitWithdraw() {
      if (this.isLoading) return;
      
      this.errorMessage = "";
      this.successMessage = "";

      if (!this.validateAll()) {
        return;
      }

      const user = auth.currentUser;
      if (!user) {
        this.errorMessage = "الرجاء تسجيل الدخول مرة أخرى";
        return;
      }

      // التحقق من عدم وجود سحب سابق اليوم
      const hasWithdrawToday = await this.checkExistingWithdrawToday();
      if (hasWithdrawToday) {
        this.errorMessage = "⚠️ لقد قمت بالسحب مسبقاً اليوم. يمكنك السحب مرة واحدة فقط في اليوم المحدد";
        return;
      }

      this.isLoading = true;

      try {
        const userRef = doc(db, "users", user.uid);
        const withdrawAmount = Number(this.amount);

        // تنفيذ عملية السحب
        await runTransaction(db, async (transaction) => {
          const userDoc = await transaction.get(userRef);
          
          if (!userDoc.exists()) {
            throw new Error("المستخدم غير موجود");
          }

          const userData = userDoc.data();
          
          // التحقق النهائي من الرصيد
          if (userData.balance < withdrawAmount) {
            throw new Error("الرصيد غير كاف");
          }

          // التحقق من عدم وجود حظر
          if (userData.blocked === true) {
            throw new Error("حسابك محظور من السحب");
          }

          // تحديث الرصيد
          transaction.update(userRef, {
            balance: userData.balance - withdrawAmount
          });

          // إنشاء طلب سحب
          const withdrawRef = doc(collection(db, "withdraw_requests"));
          transaction.set(withdrawRef, {
            userId: user.uid,
            userEmail: user.email,
            userName: user.displayName || "",
            amount: withdrawAmount,
            network: this.network,
            walletAddress: this.walletAddress,
            status: "pending",
            createdAt: serverTimestamp(),
            vipLevel: this.userVipLevel,
            withdrawDay: this.withdrawDay,
            oldBalance: userData.balance,
            newBalance: userData.balance - withdrawAmount
          });
        });

        // تسجيل المعاملة
        await addDoc(collection(db, "transactions"), {
          userId: user.uid,
          userEmail: user.email,
          type: "withdraw",
          amount: withdrawAmount,
          network: this.network,
          walletAddress: this.walletAddress,
          status: "pending",
          createdAt: serverTimestamp(),
          vipLevel: this.userVipLevel
        });

        // تحديث الرصيد في الواجهة
        this.balance -= withdrawAmount;
        
        // عرض رسالة النجاح
        this.successMessage = "✅ تم إرسال طلب السحب بنجاح";
        
        // تفريغ الحقول
        this.amount = "";
        this.network = "";
        this.walletAddress = "";

        // تحديث قائمة طلبات السحب
        await this.loadRecentWithdraws();

        // إخفاء رسالة النجاح بعد 5 ثوان
        setTimeout(() => {
          this.successMessage = "";
        }, 5000);

      } catch (error) {
        console.error("خطأ في السحب:", error);
        this.errorMessage = error.message || "حدث خطأ أثناء السحب";
      } finally {
        this.isLoading = false;
      }
    }
  }
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

.vip-status-box.error p {
  margin: 0;
  font-size: 14px;
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

.gold-input.error-input {
  border-color: #ef4444;
  background: rgba(239, 68, 68, 0.05);
}

.gold-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

.input-error {
  display: block;
  color: #ef4444;
  font-size: 12px;
  margin-top: 5px;
  margin-right: 5px;
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

.gold-select.error-input {
  border-color: #ef4444;
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
  margin: 15px 0;
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

/* آخر طلبات السحب */
.recent-withdraws {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid rgba(212, 175, 55, 0.2);
}

.recent-withdraws h4 {
  color: #D4AF37;
  font-size: 14px;
  margin-bottom: 10px;
  text-align: center;
}

.withdraw-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: rgba(255, 255, 255, 0.02);
  border-radius: 10px;
  margin-bottom: 5px;
  font-size: 13px;
  color: rgba(255, 255, 255, 0.8);
}

.withdraw-item .status {
  padding: 4px 8px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
}

.withdraw-item .status.pending {
  background: rgba(212, 175, 55, 0.1);
  color: #D4AF37;
}

.withdraw-item .status.approved {
  background: rgba(34, 197, 94, 0.1);
  color: #22c55e;
}

.withdraw-item .status.rejected {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
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

/* أنيميشن للتحميل */
@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.fa-spinner {
  animation: spin 1s linear infinite;
}
</style>
