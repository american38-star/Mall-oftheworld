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
      // التحقق من وجود VIP أولاً
      if (!this.userVip) {
        console.log("لا يوجد VIP");
        return false;
      }
      
      // التحقق من اختيار الشبكة
      if (!this.selectedNetwork) {
        console.log("لم يتم اختيار الشبكة");
        return false;
      }
      
      // التحقق من المحفظة
      if (!this.wallet || this.wallet.length < 20) {
        console.log("المحفظة غير صحيحة");
        return false;
      }
      
      // التحقق من المبلغ
      if (!this.amount || this.amount <= 0) {
        console.log("المبلغ غير صحيح");
        return false;
      }
      
      // التحقق من عدم وجود عملية جارية
      if (this.isProcessing) {
        console.log("يوجد عملية جارية");
        return false;
      }
      
      // التحقق من الرصيد
      if (this.balance < this.getWithdrawLimit) {
        console.log("الرصيد غير كافي");
        return false;
      }
      
      // التحقق من تطابق المبلغ مع الحد المسموح
      if (Number(this.amount) !== this.getWithdrawLimit) {
        console.log("المبلغ غير مطابق للحد المسموح");
        return false;
      }
      
      // التحقق من اليوم المسموح
      if (!this.isAllowedDay) {
        console.log("اليوم غير مسموح");
        return false;
      }
      
      console.log("جميع الشروط مستوفاة - يمكن السحب");
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
      console.log("تم الضغط على زر السحب");
      this.message = "";
      this.withdrawError = "";

      // التحقق من الشروط الأساسية
      if (!this.checkWithdrawConditions()) {
        console.log("فشل في التحقق من الشروط:", this.withdrawError);
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
      console.log("بدء عملية السحب...");

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
        console.log("تم السحب بنجاح");
        
        this.balance -= Number(this.amount);
        
        setTimeout(() => {
          this.amount = "";
          this.wallet = "";
          this.selectedNetwork = "";
          this.message = "";
        }, 3000);

      } catch (e) {
        console.error("خطأ في السحب:", e);
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
