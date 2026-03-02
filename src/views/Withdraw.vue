<template>
  <div class="withdraw-page">
    <div class="card">
      <div class="card-header">
        <h2 class="title">
          <i class="fas fa-hand-holding-usd"></i>
          سحب الأرباح
          <span class="title-glow">USDT</span>
        </h2>
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

      <!-- حالة VIP -->
      <div v-if="userVipLevel" class="vip-status-box">
        <div class="vip-badge">
          <i class="fas fa-crown"></i>
          مستوى VIP {{ userVipLevel }}
        </div>
        <div class="user-email">
          <i class="fas fa-envelope"></i>
          {{ userEmail }}
        </div>
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': balance >= minWithdrawAmount }"></i>
          <span>الحد الأدنى: <strong>{{ minWithdrawAmount }} USDT</strong></span>
        </div>
        <div class="withdraw-condition">
          <i class="fas fa-check-circle" :class="{ 'condition-met': isAllowedDay }"></i>
          <span>يوم السحب: <strong>{{ withdrawDay }}</strong></span>
        </div>
      </div>

      <div v-else class="vip-status-box error">
        <i class="fas fa-exclamation-triangle"></i>
        <p>يجب أن يكون لديك اشتراك VIP للسحب</p>
      </div>

      <!-- رسائل الخطأ والنجاح -->
      <transition name="fade">
        <div v-if="message" class="message" :class="messageType">
          <i :class="messageType === 'error' ? 'fas fa-exclamation-circle' : 'fas fa-check-circle'"></i>
          {{ message }}
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
            @input="validateAmount"
          />
          <span class="input-currency">USDT</span>
        </div>
        <span v-if="amountError" class="input-error">{{ amountError }}</span>
      </div>

      <!-- الشبكة -->
      <div class="input-group">
        <label>
          <i class="fas fa-network-wired"></i>
          الشبكة
        </label>
        <div class="select-wrapper">
          <select v-model="network" class="gold-select" @change="validateNetwork">
            <option value="">اختر الشبكة</option>
            <option value="TRC20">TRC20</option>
            <option value="ERC20">ERC20</option>
            <option value="BEP20">BEP20</option>
            <option value="SOL">SOL</option>
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
          v-model="wallet" 
          placeholder="أدخل عنوان محفظتك USDT" 
          class="gold-input"
          @input="validateWallet"
        />
        <span v-if="walletError" class="input-error">{{ walletError }}</span>
      </div>

      <!-- ملخص الطلب -->
      <div v-if="showSummary" class="summary-box">
        <h3>📋 ملخص طلب السحب</h3>
        
        <div class="summary-item">
          <span>البريد الإلكتروني:</span>
          <span class="summary-value">{{ userEmail }}</span>
        </div>
        
        <div class="summary-item">
          <span>مستوى VIP:</span>
          <span class="summary-value">{{ userVipLevel || 'لا يوجد' }}</span>
        </div>
        
        <div class="summary-item">
          <span>المبلغ:</span>
          <span class="summary-value">{{ Number(amount).toFixed(2) }} USDT</span>
        </div>
        
        <div class="summary-item">
          <span>الشبكة:</span>
          <span class="summary-value">{{ network }}</span>
        </div>
        
        <div class="summary-item">
          <span>عنوان المحفظة:</span>
          <span class="summary-value address">{{ wallet.substring(0, 10) }}...{{ wallet.substring(wallet.length - 10) }}</span>
        </div>
        
        <div class="summary-item">
          <span>يوم السحب:</span>
          <span class="summary-value">{{ withdrawDay }}</span>
        </div>
        
        <div class="summary-item">
          <span>الحد الأدنى:</span>
          <span class="summary-value">{{ minWithdrawAmount }} USDT</span>
        </div>
        
        <div class="summary-item total">
          <span>سيتم خصم:</span>
          <span class="summary-value">{{ Number(amount).toFixed(2) }} USDT</span>
        </div>
        
        <div class="summary-item">
          <span>الرصيد بعد السحب:</span>
          <span class="summary-value">{{ (balance - Number(amount)).toFixed(2) }} USDT</span>
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
        :disabled="isLoading || !isFormValid"
      >
        <i class="fas fa-paper-plane" v-if="!isLoading"></i>
        <i class="fas fa-spinner fa-spin" v-else></i>
        {{ isLoading ? 'جاري المعالجة...' : 'تأكيد السحب' }}
      </button>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc, runTransaction, collection, addDoc, serverTimestamp } from "firebase/firestore";

export default {
  name: "Withdraw",
  
  data() {
    return {
      balance: 0,
      amount: "",
      network: "",
      wallet: "",
      isLoading: false,
      message: "",
      messageType: "info",
      userVipLevel: null,
      userEmail: "",
      minWithdrawAmount: 5,
      
      // أخطاء الحقول
      amountError: "",
      networkError: "",
      walletError: "",
      
      vipLimits: {
        1: 5, 2: 25, 3: 50, 4: 160, 5: 530,
        6: 820, 7: 1120, 8: 2400, 9: 5300, 10: 11300,
        11: 26000, 12: 56000, 13: 120000, 14: 260000
      },
      
      withdrawDays: {
        1: "السبت", 2: "السبت", 3: "الأحد", 4: "الأحد",
        5: "الاثنين", 6: "الاثنين", 7: "الثلاثاء", 8: "الثلاثاء",
        9: "الأربعاء", 10: "الأربعاء", 11: "الخميس", 12: "الخميس",
        13: "الجمعة", 14: "الجمعة"
      }
    };
  },

  computed: {
    withdrawDay() {
      return this.withdrawDays[this.userVipLevel] || "";
    },

    isAllowedDay() {
      if (!this.userVipLevel) return false;
      
      const dayMap = {
        "السبت": "Saturday",
        "الأحد": "Sunday",
        "الاثنين": "Monday",
        "الثلاثاء": "Tuesday",
        "الأربعاء": "Wednesday",
        "الخميس": "Thursday",
        "الجمعة": "Friday"
      };
      
      const today = new Date().toLocaleDateString("en-US", { weekday: "long" });
      const allowedDay = this.withdrawDays[this.userVipLevel];
      
      return today === dayMap[allowedDay];
    },

    isFormValid() {
      return (
        this.amount && 
        !this.amountError &&
        this.network && 
        !this.networkError &&
        this.wallet && 
        !this.walletError &&
        this.userVipLevel &&
        this.isAllowedDay &&
        Number(this.amount) === this.minWithdrawAmount &&
        this.balance >= this.minWithdrawAmount
      );
    },

    showSummary() {
      return this.amount && this.network && this.wallet && this.userVipLevel;
    }
  },

  watch: {
    amount() {
      this.validateAmount();
    },
    network() {
      this.validateNetwork();
    },
    wallet() {
      this.validateWallet();
    },
    userVipLevel() {
      if (this.userVipLevel) {
        this.minWithdrawAmount = this.vipLimits[this.userVipLevel] || 5;
      }
    }
  },

  async created() {
    await this.loadUserData();
  },

  methods: {
    async loadUserData() {
      const user = auth.currentUser;
      if (!user) {
        this.$router.push("/login");
        return;
      }

      try {
        // حفظ البريد الإلكتروني
        this.userEmail = user.email || "لا يوجد بريد";
        
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
          this.showMessage("لا يوجد اشتراك VIP نشط", "error");
        }
      } catch (error) {
        console.error("خطأ:", error);
        this.showMessage("حدث خطأ في تحميل البيانات", "error");
      }
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
      if (!this.wallet) {
        this.walletError = "الرجاء إدخال عنوان المحفظة";
        return false;
      }
      
      if (this.wallet.length < 20) {
        this.walletError = "عنوان المحفظة غير صحيح (يجب أن يكون 20 حرف على الأقل)";
        return false;
      }
      
      this.walletError = "";
      return true;
    },

    showMessage(msg, type) {
      this.message = msg;
      this.messageType = type;
      setTimeout(() => {
        this.message = "";
      }, 5000);
    },

    async submitWithdraw() {
      if (this.isLoading || !this.isFormValid) return;
      
      this.message = "";

      const user = auth.currentUser;
      if (!user) {
        this.showMessage("الرجاء تسجيل الدخول مرة أخرى", "error");
        return;
      }

      this.isLoading = true;

      try {
        const userRef = doc(db, "users", user.uid);
        const withdrawAmount = Number(this.amount);
        const transactionId = "WITHDRAW_" + Date.now() + "_" + Math.random().toString(36).substring(2, 9);

        await runTransaction(db, async (transaction) => {
          const userDoc = await transaction.get(userRef);
          
          if (!userDoc.exists()) {
            throw new Error("المستخدم غير موجود");
          }

          const userData = userDoc.data();
          
          if (userData.balance < withdrawAmount) {
            throw new Error("الرصيد غير كاف");
          }

          if (userData.blocked) {
            throw new Error("حسابك محظور من السحب");
          }

          transaction.update(userRef, {
            balance: userData.balance - withdrawAmount
          });

          // إنشاء طلب سحب في withdraw_requests
          const withdrawRef = doc(collection(db, "withdraw_requests"));
          transaction.set(withdrawRef, {
            transactionId: transactionId,
            userId: user.uid,
            userEmail: this.userEmail,
            amount: withdrawAmount,
            network: this.network,
            wallet: this.wallet,
            walletAddress: this.wallet,
            status: "pending",
            createdAt: serverTimestamp(),
            vipLevel: this.userVipLevel,
            withdrawDay: this.withdrawDay,
            adminAction: "",
            adminMessage: "",
            userMessage: "",
            reason: ""
          });
        });

        // إنشاء معاملة في مجموعة transactions
        await addDoc(collection(db, "transactions"), {
          transactionId: transactionId,
          userId: user.uid,
          userEmail: this.userEmail,
          type: "withdraw",
          amount: withdrawAmount,
          currency: "USDT",
          network: this.network,
          wallet: this.wallet,
          walletAddress: this.wallet,
          status: "pending",
          vipLevel: this.userVipLevel,
          withdrawDay: this.withdrawDay,
          adminAction: "",
          adminMessage: "",
          userMessage: "",
          reason: "",
          createdAt: serverTimestamp(),
          updatedAt: serverTimestamp(),
          approvedAt: null
        });

        this.balance -= withdrawAmount;
        
        this.showMessage("✅ تم إرسال طلب السحب بنجاح", "success");
        
        // تفريغ الحقول
        this.amount = "";
        this.network = "";
        this.wallet = "";

      } catch (error) {
        console.error("خطأ:", error);
        this.showMessage(error.message || "حدث خطأ أثناء السحب", "error");
      } finally {
        this.isLoading = false;
      }
    }
  }
};
</script>

<style scoped>
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

.card {
  background: #11151C;
  width: 100%;
  max-width: 500px;
  border-radius: 30px;
  padding: 30px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
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

.card-header {
  text-align: center;
  margin-bottom: 25px;
}

.title {
  font-size: 28px;
  font-weight: 800;
  color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 8px;
}

.title i {
  color: #D4AF37;
  font-size: 32px;
}

.title-glow {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 900;
}

.sub {
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
}

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
}

.balance-label {
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  display: block;
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
  margin-bottom: 10px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.vip-badge i {
  margin-left: 5px;
}

.user-email {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #D4AF37;
  font-size: 14px;
  margin-bottom: 15px;
  padding: 8px;
  background: rgba(212, 175, 55, 0.05);
  border-radius: 10px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.user-email i {
  color: #D4AF37;
  font-size: 14px;
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
}

.input-error {
  display: block;
  color: #ef4444;
  font-size: 12px;
  margin-top: 5px;
  margin-right: 5px;
}

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
  appearance: none;
}

.gold-select:focus {
  outline: none;
  border-color: #D4AF37;
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

/* ملخص الطلب */
.summary-box {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 20px;
  padding: 20px;
  margin: 20px 0;
  border: 2px solid #D4AF37;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.2);
}

.summary-box h3 {
  color: #D4AF37;
  font-size: 18px;
  margin-bottom: 15px;
  text-align: center;
  position: relative;
  padding-bottom: 10px;
}

.summary-box h3::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 50px;
  height: 2px;
  background: #D4AF37;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid rgba(212, 175, 55, 0.2);
  color: rgba(255, 255, 255, 0.9);
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
  font-size: 16px;
}

.summary-value {
  font-weight: 600;
  color: #D4AF37;
}

.summary-value.address {
  font-family: monospace;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.8);
  background: rgba(212, 175, 55, 0.1);
  padding: 4px 8px;
  border-radius: 8px;
}

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

.gold-button {
  width: 100%;
  padding: 16px;
  border-radius: 20px;
  border: none;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
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

.fade-enter-active, .fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.fa-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

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
  
  .summary-box {
    padding: 15px;
  }
  
  .summary-item {
    font-size: 13px;
  }
}
</style>
