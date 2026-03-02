<template>
  <div class="withdraw-page">
    <div class="card">
      <div class="card-header">
        <h2 class="title">
          <i class="fas fa-hand-holding-usd"></i>
          سحب الأرباح
        </h2>
      </div>

      <!-- رصيد المستخدم -->
      <div class="balance-box">
        <div class="balance-info">
          <span class="balance-label">رصيدك الحالي</span>
          <span class="balance-value">{{ balance }} USDT</span>
        </div>
      </div>

      <!-- رسائل الخطأ -->
      <div v-if="errorMessage" class="message error">
        <i class="fas fa-exclamation-circle"></i>
        {{ errorMessage }}
      </div>

      <!-- رسائل النجاح -->
      <div v-if="successMessage" class="message success">
        <i class="fas fa-check-circle"></i>
        {{ successMessage }}
      </div>

      <!-- مبلغ السحب -->
      <div class="input-group">
        <label>المبلغ (USDT)</label>
        <input 
          type="number" 
          v-model="amount" 
          class="gold-input"
          :placeholder="'الحد الأدنى: ' + minWithdrawAmount"
        />
      </div>

      <!-- الشبكة -->
      <div class="input-group">
        <label>الشبكة</label>
        <select v-model="network" class="gold-select">
          <option value="">اختر الشبكة</option>
          <option value="TRC20">TRC20</option>
          <option value="ERC20">ERC20</option>
          <option value="BEP20">BEP20</option>
        </select>
      </div>

      <!-- عنوان المحفظة -->
      <div class="input-group">
        <label>عنوان المحفظة</label>
        <input 
          type="text" 
          v-model="walletAddress" 
          class="gold-input"
          placeholder="أدخل عنوان محفظتك"
        />
      </div>

      <!-- زر السحب -->
      <button 
        class="gold-button" 
        @click="submitWithdraw"
        :disabled="isLoading"
      >
        <span v-if="!isLoading">سحب الآن</span>
        <span v-else>جاري المعالجة...</span>
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
      walletAddress: "",
      isLoading: false,
      errorMessage: "",
      successMessage: "",
      userVipLevel: null,
      minWithdrawAmount: 5 // القيمة الافتراضية
    };
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
          this.setMinWithdrawAmount();
        } else {
          this.errorMessage = "لا يوجد اشتراك VIP نشط";
        }
      } catch (error) {
        console.error("خطأ في تحميل البيانات:", error);
        this.errorMessage = "حدث خطأ في تحميل البيانات";
      }
    },

    setMinWithdrawAmount() {
      const limits = {
        1: 5, 2: 25, 3: 50, 4: 160, 5: 530,
        6: 820, 7: 1120, 8: 2400, 9: 5300, 10: 11300,
        11: 26000, 12: 56000, 13: 120000, 14: 260000
      };
      this.minWithdrawAmount = limits[this.userVipLevel] || 5;
    },

    validateForm() {
      this.errorMessage = "";

      // التحقق من VIP
      if (!this.userVipLevel) {
        this.errorMessage = "يجب أن يكون لديك اشتراك VIP للسحب";
        return false;
      }

      // التحقق من المبلغ
      if (!this.amount || this.amount <= 0) {
        this.errorMessage = "الرجاء إدخال مبلغ صحيح";
        return false;
      }

      // التحقق من أن المبلغ يساوي الحد الأدنى
      if (Number(this.amount) !== this.minWithdrawAmount) {
        this.errorMessage = `مبلغ السحب يجب أن يكون بالضبط ${this.minWithdrawAmount} USDT`;
        return false;
      }

      // التحقق من الرصيد
      if (this.balance < this.minWithdrawAmount) {
        this.errorMessage = `رصيدك غير كاف. الحد الأدنى للسحب ${this.minWithdrawAmount} USDT`;
        return false;
      }

      // التحقق من الشبكة
      if (!this.network) {
        this.errorMessage = "الرجاء اختيار الشبكة";
        return false;
      }

      // التحقق من عنوان المحفظة
      if (!this.walletAddress || this.walletAddress.length < 20) {
        this.errorMessage = "الرجاء إدخال عنوان محفظة صحيح";
        return false;
      }

      return true;
    },

    async submitWithdraw() {
      // منع النقر المتكرر
      if (this.isLoading) return;
      
      // إخفاء الرسائل السابقة
      this.errorMessage = "";
      this.successMessage = "";

      // التحقق من صحة المدخلات
      if (!this.validateForm()) {
        return;
      }

      const user = auth.currentUser;
      if (!user) {
        this.errorMessage = "الرجاء تسجيل الدخول مرة أخرى";
        return;
      }

      this.isLoading = true;

      try {
        const userRef = doc(db, "users", user.uid);
        const withdrawAmount = Number(this.amount);

        // تنفيذ عملية السحب
        await runTransaction(db, async (transaction) => {
          // قراءة بيانات المستخدم
          const userDoc = await transaction.get(userRef);
          
          if (!userDoc.exists()) {
            throw new Error("المستخدم غير موجود");
          }

          const userData = userDoc.data();
          
          // التحقق من الرصيد مرة أخرى
          if (userData.balance < withdrawAmount) {
            throw new Error("الرصيد غير كاف");
          }

          // التحقق من عدم وجود حظر
          if (userData.blocked) {
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
            amount: withdrawAmount,
            network: this.network,
            walletAddress: this.walletAddress,
            status: "pending",
            createdAt: serverTimestamp(),
            vipLevel: this.userVipLevel
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
          createdAt: serverTimestamp()
        });

        // تحديث الرصيد في الواجهة
        this.balance -= withdrawAmount;
        
        // عرض رسالة النجاح
        this.successMessage = "✅ تم إرسال طلب السحب بنجاح";
        
        // تفريغ الحقول
        this.amount = "";
        this.network = "";
        this.walletAddress = "";

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
}

.balance-box {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 25px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.balance-label {
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  display: block;
  margin-bottom: 5px;
}

.balance-value {
  font-size: 32px;
  font-weight: 800;
  color: #D4AF37;
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: block;
  color: #D4AF37;
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 8px;
}

.gold-input, .gold-select {
  width: 100%;
  padding: 14px 20px;
  border-radius: 16px;
  background: #1A1F2A;
  color: #ffffff;
  border: 2px solid rgba(212, 175, 55, 0.3);
  font-size: 15px;
  transition: all 0.3s ease;
}

.gold-input:focus, .gold-select:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
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
  margin-top: 20px;
}

.gold-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.4);
}

.gold-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
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
}

.message.success {
  background: rgba(34, 197, 94, 0.1);
  color: #22c55e;
  border: 1px solid #22c55e;
}

.message.error {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
  border: 1px solid #ef4444;
}
</style>
