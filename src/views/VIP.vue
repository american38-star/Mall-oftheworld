<template>
  <div class="vip-page">
    <div class="container">
      <h1 class="page-title">
        <span class="title-glow">💎</span>
        مستويات VIP الفاخرة
        <span class="title-glow">💎</span>
      </h1>

      <div v-if="loading" class="center">
        <div class="gold-spinner"></div>
        <div class="loading-text">جاري تحميل حالة VIP...</div>
      </div>

      <div v-else>
        <!-- حالة المستخدم الحالية -->
        <div v-if="userVip && userVip.level" class="current-vip">
          <div class="current-left">
            <div class="badge-gold">✨ مفعل الآن ✨</div>
            <div class="vip-name">
              <span class="vip-level-badge">VIP {{ userVip.level }}</span>
            </div>
            <div class="vip-stats">
              <div class="stat-item">
                <i class="fas fa-coins"></i>
                <span>ربح يومي: <strong>{{ userVip.daily }} USDT</strong></span>
              </div>
              <div class="stat-item">
                <i class="fas fa-calendar-week"></i>
                <span>ربح أسبوعي: <strong>{{ (userVip.daily * 7).toFixed(2) }} USDT</strong></span>
              </div>
              <div class="stat-item">
                <i class="fas fa-calendar-alt"></i>
                <span>ربح شهري: <strong>{{ (userVip.daily * 30).toFixed(2) }} USDT</strong></span>
              </div>
              <div class="stat-item">
                <i class="fas fa-calendar-alt"></i>
                <span>ربح سنوي: <strong>{{ (userVip.daily * 365).toLocaleString() }} USDT</strong></span>
              </div>
            </div>
            <div class="vip-remaining">
              <i class="fas fa-hourglass-half"></i>
              الوقت المتبقي: {{ remainingText }}
            </div>
          </div>

          <div class="current-right">
            <button class="btn-gold-outline" @click="goToDetails">
              <i class="fas fa-info-circle"></i>
              تفاصيل
            </button>
            <!-- زر الإلغاء تمت إزالته بالكامل -->
          </div>
        </div>

        <!-- قائمة المستويات VIP -->
        <div class="vip-grid">
          <div
            class="vip-card"
            v-for="plan in plans"
            :key="plan.level"
            :class="{ 
              'active': userVip && userVip.level === plan.level,
              'premium': plan.level >= 8
            }"
          >
            <div class="card-header" :class="`level-${plan.level}`">
              <div class="level-icon">
                <span v-if="plan.level === 1">🥉</span>
                <span v-else-if="plan.level === 2">🥈</span>
                <span v-else-if="plan.level === 3">🥇</span>
                <span v-else>💎</span>
              </div>
              <div class="level-number">VIP {{ plan.level }}</div>
              <div class="level-badge" v-if="plan.level >= 10">🏆 النخبة</div>
            </div>

            <div class="card-body">
              <div class="price-tag">
                <span class="price-label">سعر الاشتراك</span>
                <span class="price-value">{{ plan.price }} USDT</span>
              </div>

              <div class="benefits-list">
                <div class="benefit-item">
                  <i class="fas fa-check-circle"></i>
                  <span>ربح يومي: <strong>{{ plan.daily }} USDT</strong></span>
                </div>
                <div class="benefit-item">
                  <i class="fas fa-calendar-week"></i>
                  <span>ربح أسبوعي: <strong>{{ (plan.daily * 7).toFixed(2) }} USDT</strong></span>
                </div>
                <div class="benefit-item">
                  <i class="fas fa-calendar-alt"></i>
                  <span>ربح شهري: <strong>{{ (plan.daily * 30).toFixed(2) }} USDT</strong></span>
                </div>
                <div class="benefit-item">
                  <i class="fas fa-tasks"></i>
                  <span>مهام يومية: {{ plan.tasks }}</span>
                </div>
                <div class="benefit-item">
                  <i class="fas fa-calendar-alt"></i>
                  <span>المدة: 24 ساعة</span>
                </div>
                <div class="benefit-item total-earnings">
                  <i class="fas fa-chart-line"></i>
                  <span>الإجمالي السنوي: <strong>{{ (plan.daily * 365).toLocaleString() }} USDT</strong></span>
                </div>
              </div>

              <div class="card-footer">
                <div class="roi-badge" v-if="plan.level >= 5">
                  <i class="fas fa-fire"></i>
                  عائد استثنائي
                </div>
                
                <button
                  class="btn-buy"
                  v-if="!isActivePlan(plan)"
                  @click="buyPlan(plan)"
                  :disabled="processing"
                >
                  <i class="fas fa-crown"></i>
                  اشترِ الآن
                </button>

                <button class="btn-active" v-else disabled>
                  <i class="fas fa-check-circle"></i>
                  مفعل
                </button>
              </div>
            </div>

            <!-- علامة مميزة للخطط النشطة -->
            <div class="active-ribbon" v-if="userVip && userVip.level === plan.level">
              نشط الآن
            </div>
          </div>
        </div>

        <!-- معلومات إضافية -->
        <div class="info-panel">
          <div class="info-item">
            <i class="fas fa-clock"></i>
            <div class="info-content">
              <h4>توقيت المكافآت</h4>
              <p>يتم توزيع الأرباح يومياً في الساعة 03:30 UTC (السادسة صباحاً بتوقيت مكة المكرمة)</p>
            </div>
          </div>
          
          <div class="info-item">
            <i class="fas fa-shield-alt"></i>
            <div class="info-content">
              <h4>سياسة الاسترداد</h4>
              <p>الاشتراك غير قابل للاسترداد. يتم احتساب الأرباح تلقائياً عند اكتمال الدورة.</p>
            </div>
          </div>
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
  serverTimestamp,
  Timestamp,
  setDoc,
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "VIP",
  data() {
    return {
      loading: true,
      processing: false,
      userVip: null,
      remainingMs: 0,
      intervalId: null,

      plans: [
        // المستوى 1 - مجاني
        { level: 1, name: "VIP 1", price: 0, tasks: 1, daily: 0.30, durationSeconds: 86400 },
        // المستوى 2
        { level: 2, name: "VIP 2", price: 50, tasks: 1, daily: 1.60, durationSeconds: 86400 },
        // المستوى 3
        { level: 3, name: "VIP 3", price: 100, tasks: 1, daily: 3.25, durationSeconds: 86400 },
        // المستوى 4
        { level: 4, name: "VIP 4", price: 300, tasks: 1, daily: 10, durationSeconds: 86400 },
        // المستوى 5
        { level: 5, name: "VIP 5", price: 900, tasks: 1, daily: 33, durationSeconds: 86400 },
        // المستوى 6
        { level: 6, name: "VIP 6", price: 1350, tasks: 1, daily: 51, durationSeconds: 86400 },
        // المستوى 7
        { level: 7, name: "VIP 7", price: 1800, tasks: 1, daily: 70, durationSeconds: 86400 },
        // المستوى 8
        { level: 8, name: "VIP 8", price: 3600, tasks: 1, daily: 150, durationSeconds: 86400 },
        // المستوى 9
        { level: 9, name: "VIP 9", price: 7200, tasks: 1, daily: 330, durationSeconds: 86400 },
        // المستوى 10
        { level: 10, name: "VIP 10", price: 14400, tasks: 1, daily: 700, durationSeconds: 86400 },
        // المستوى 11
        { level: 11, name: "VIP 11", price: 18800, tasks: 1, daily: 1600, durationSeconds: 86400 },
        // المستوى 12
        { level: 12, name: "VIP 12", price: 37600, tasks: 1, daily: 3500, durationSeconds: 86400 },
        // المستوى 13
        { level: 13, name: "VIP 13", price: 75200, tasks: 1, daily: 7500, durationSeconds: 86400 },
        // المستوى 14
        { level: 14, name: "VIP 14", price: 150400, tasks: 1, daily: 16000, durationSeconds: 86400 },
      ],

      globalCycleHourUTC: 3,
      globalCycleMinuteUTC: 30,
    };
  },

  computed: {
    remainingText() {
      if (!this.userVip || !this.userVip.vipEnd) return "--:--:--";
      const ms = Math.max(0, this.remainingMs || 0);
      const sec = Math.floor(ms / 1000);
      const h = String(Math.floor(sec / 3600)).padStart(2, "0");
      const m = String(Math.floor((sec % 3600) / 60)).padStart(2, "0");
      const s = String(sec % 60).padStart(2, "0");
      return `${h}:${m}:${s}`;
    },
  },

  created() {
    onAuthStateChanged(auth, async (user) => {
      if (!user) {
        this.loading = false;
        this.userVip = null;
        this.$router.push("/login");
        return;
      }
      await this.init();
    });
  },

  beforeUnmount() {
    if (this.intervalId) clearInterval(this.intervalId);
  },

  methods: {
    isActivePlan(plan) {
      return this.userVip && this.userVip.level === plan.level;
    },

    getNextGlobalCycleFromMs(msRef) {
      const ref = msRef ? new Date(msRef) : new Date();
      const next = new Date(ref.getTime());
      next.setUTCHours(this.globalCycleHourUTC, this.globalCycleMinuteUTC, 0, 0);
      if (next.getTime() <= ref.getTime()) {
        next.setTime(next.getTime() + 24 * 3600 * 1000);
      }
      return next;
    },

    async init() {
      this.loading = true;
      this.userVip = null;
      try {
        const user = auth.currentUser;
        if (!user) {
          this.loading = false;
          return;
        }

        const vipDocRef = doc(db, "users", user.uid, "vip", "current");
        const vipSnap = await getDoc(vipDocRef);
        
        if (vipSnap.exists()) {
          const data = vipSnap.data();
          this.userVip = {
            level: data.level || null,
            price: data.price || 0,
            daily: data.daily || 0,
            vipStart: data.vipStart || null,
            vipEnd: data.vipEnd || null,
            durationSeconds: data.durationSeconds || 86400,
            lastRewardAt: data.lastRewardAt || null,
          };

          await this.settleMissedCyclesIfAny(user.uid, vipDocRef);

          const vipSnap2 = await getDoc(vipDocRef);
          if (vipSnap2.exists()) {
            const d2 = vipSnap2.data();
            this.userVip = {
              level: d2.level || null,
              price: d2.price || 0,
              daily: d2.daily || 0,
              vipStart: d2.vipStart || null,
              vipEnd: d2.vipEnd || null,
              durationSeconds: d2.durationSeconds || 86400,
              lastRewardAt: d2.lastRewardAt || null,
            };
          }
        }

        this.startTimer();
      } catch (err) {
        console.error("VIP init error:", err);
        this.userVip = null;
      } finally {
        this.loading = false;
      }
    },

    async settleMissedCyclesIfAny(uid, vipDocRef) {
      try {
        await runTransaction(db, async (tx) => {
          const vipSnap = await tx.get(vipDocRef);
          if (!vipSnap.exists()) return;

          const vipData = vipSnap.data();
          if (!vipData.vipEnd || !vipData.daily) return;

          const vipEndMs = vipData.vipEnd.toMillis ? vipData.vipEnd.toMillis() : vipData.vipEnd;
          const nowMs = Date.now();
          const durationMs = (vipData.durationSeconds || 86400) * 1000;

          if (nowMs <= vipEndMs) return;

          const diffMs = nowMs - vipEndMs;
          const extraCycles = Math.floor(diffMs / durationMs) + 1;
          const totalReward = Number(vipData.daily || 0) * extraCycles;

          const userRef = doc(db, "users", uid);
          const userSnap = await tx.get(userRef);
          const currentBal = userSnap.exists() ? Number(userSnap.data().balance || 0) : 0;
          tx.update(userRef, { balance: currentBal + totalReward });

          const newVipEnd = Timestamp.fromMillis(vipEndMs + extraCycles * durationMs);
          tx.update(vipDocRef, { vipEnd: newVipEnd, lastRewardAt: serverTimestamp() });

          const logsRef = collection(db, "vip_rewards");
          const logDocRef = doc(logsRef);
          tx.set(logDocRef, {
            uid,
            amount: totalReward,
            cycles: extraCycles,
            type: "missed_settlement",
            createdAt: serverTimestamp(),
          });
        });
      } catch (e) {
        console.error("settleMissedCycles error:", e);
      }
    },

    startTimer() {
      if (!this.userVip || !this.userVip.vipEnd) return;

      const vipEndMs = this.userVip.vipEnd.toMillis ? this.userVip.vipEnd.toMillis() : this.userVip.vipEnd;
      this.remainingMs = vipEndMs - Date.now();

      if (this.intervalId) clearInterval(this.intervalId);
      this.intervalId = setInterval(async () => {
        this.remainingMs -= 1000;
        if (this.remainingMs <= 0) {
          await this.onCycleComplete();
        }
      }, 1000);
    },

    async onCycleComplete() {
      if (this.processing) return;
      this.processing = true;
      const user = auth.currentUser;
      if (!user) {
        this.processing = false;
        return;
      }

      const vipDocRef = doc(db, "users", user.uid, "vip", "current");
      const userRef = doc(db, "users", user.uid);

      try {
        await runTransaction(db, async (tx) => {
          const vipSnap = await tx.get(vipDocRef);
          if (!vipSnap.exists()) return;

          const vipData = vipSnap.data();
          const daily = Number(vipData.daily || 0);
          const durationSeconds = Number(vipData.durationSeconds || 86400);
          const vipEndMs = vipData.vipEnd.toMillis ? vipData.vipEnd.toMillis() : vipData.vipEnd;
          const nowMs = Date.now();
          if (nowMs < vipEndMs) return;

          const userSnap = await tx.get(userRef);
          const curBal = userSnap.exists() ? Number(userSnap.data().balance || 0) : 0;
          tx.update(userRef, { balance: curBal + daily });

          const newVipEnd = Timestamp.fromMillis(vipEndMs + durationSeconds * 1000);
          tx.update(vipDocRef, { vipEnd: newVipEnd, lastRewardAt: serverTimestamp() });

          const logsRef = collection(db, "vip_rewards");
          const logDocRef = doc(logsRef);
          tx.set(logDocRef, {
            uid: user.uid,
            amount: daily,
            type: "daily",
            createdAt: serverTimestamp(),
            level: vipData.level || null,
          });
        });

        await this.init();
      } catch (e) {
        console.error("onCycleComplete error:", e);
      } finally {
        this.processing = false;
      }
    },

    async buyPlan(plan) {
      const user = auth.currentUser;
      if (!user) return alert("يرجى تسجيل الدخول أولاً.");
      if (this.processing) return;

      this.processing = true;
      try {
        const userRef = doc(db, "users", user.uid);
        const vipDocRef = doc(db, "users", user.uid, "vip", "current");
        const purchasesRef = collection(db, "vip_purchases");
        const rewardsRef = collection(db, "vip_rewards");

        await runTransaction(db, async (tx) => {
          const uSnap = await tx.get(userRef);
          if (!uSnap.exists()) throw new Error("لم يتم العثور على حساب المستخدم.");

          const balance = Number(uSnap.data().balance || 0);
          if (balance < plan.price) throw new Error("رصيد غير كافٍ لشراء هذا المستوى.");

          const newBalance = balance - plan.price + plan.daily;
          tx.update(userRef, { balance: newBalance });

          const nowMs = Date.now();
          const nextGlobalDate = this.getNextGlobalCycleFromMs(nowMs);
          const vipEndTs = Timestamp.fromDate(nextGlobalDate);

          tx.set(vipDocRef, {
            level: plan.level,
            price: plan.price,
            daily: plan.daily,
            durationSeconds: plan.durationSeconds || 86400,
            vipStart: Timestamp.fromMillis(nowMs),
            vipEnd: vipEndTs,
            lastRewardAt: serverTimestamp(),
            purchasedAt: serverTimestamp(),
          });

          const pDocRef = doc(purchasesRef);
          tx.set(pDocRef, {
            uid: user.uid,
            planId: plan.level,
            level: plan.level,
            price: plan.price,
            daily: plan.daily,
            createdAt: serverTimestamp(),
            type: "purchase",
          });

          const rDocRef = doc(rewardsRef);
          tx.set(rDocRef, {
            uid: user.uid,
            amount: plan.daily,
            type: "purchase_bonus",
            createdAt: serverTimestamp(),
            level: plan.level,
          });
        });

        await this.init();
        this.showSuccessMessage("✔ تم شراء VIP بنجاح! أُضيف الربح الأولي إلى رصيدك.");
      } catch (err) {
        console.error("buyPlan error:", err);
        this.showErrorMessage(err.message || "فشل شراء المستوى. تأكد من الرصيد وحاول لاحقًا.");
      } finally {
        this.processing = false;
      }
    },

    goToDetails() {
      this.$router.push("/vip");
    },

    showSuccessMessage(msg) {
      alert(msg);
    },

    showErrorMessage(msg) {
      alert(msg);
    },
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.vip-page {
  min-height: 100vh;
  background: #0A0C10;
  padding: 20px;
  direction: rtl;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
}

/* العنوان الرئيسي */
.page-title {
  text-align: center;
  font-size: 32px;
  font-weight: 800;
  margin-bottom: 30px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
}

.title-glow {
  font-size: 40px;
  filter: drop-shadow(0 0 10px rgba(212, 175, 55, 0.5));
}

/* ===== بطاقة VIP الحالية ===== */
.current-vip {
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border-radius: 24px;
  padding: 25px;
  margin-bottom: 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 2px solid #D4AF37;
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.2);
  position: relative;
  overflow: hidden;
}

.current-vip::before {
  content: '';
  position: absolute;
  top: -50%;
  right: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.1) 0%, transparent 70%);
  animation: rotate 20s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.current-left {
  position: relative;
  z-index: 1;
  width: 100%;
}

.badge-gold {
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

.vip-name {
  margin-bottom: 15px;
}

.vip-level-badge {
  font-size: 28px;
  font-weight: 900;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.vip-stats {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  margin: 15px 0;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  background: rgba(212, 175, 55, 0.1);
  padding: 8px 12px;
  border-radius: 10px;
}

.stat-item i {
  color: #D4AF37;
  width: 20px;
}

.stat-item strong {
  color: #D4AF37;
  font-size: 15px;
}

.vip-remaining {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 8px 0;
  color: #ff6b6b;
  font-size: 16px;
}

.vip-remaining i {
  color: #ff6b6b;
  width: 24px;
}

.current-right {
  display: flex;
  gap: 12px;
  position: relative;
  z-index: 1;
}

/* ===== شبكة بطاقات VIP ===== */
.vip-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 25px;
  margin-bottom: 30px;
}

.vip-card {
  background: #11151C;
  border-radius: 24px;
  overflow: hidden;
  border: 1px solid rgba(212, 175, 55, 0.2);
  transition: all 0.3s ease;
  position: relative;
}

.vip-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px rgba(212, 175, 55, 0.2);
  border-color: #D4AF37;
}

.vip-card.active {
  border: 2px solid #D4AF37;
  box-shadow: 0 0 30px rgba(212, 175, 55, 0.3);
}

.vip-card.premium {
  background: linear-gradient(135deg, #11151C, #1A1F2A);
}

/* رأس البطاقة */
.card-header {
  padding: 25px 20px;
  text-align: center;
  position: relative;
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-bottom: 1px solid rgba(212, 175, 55, 0.2);
}

.level-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.level-number {
  font-size: 24px;
  font-weight: 800;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.level-badge {
  position: absolute;
  top: 15px;
  left: 15px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 5px 12px;
  border-radius: 50px;
  font-size: 12px;
  font-weight: 700;
}

/* جسم البطاقة */
.card-body {
  padding: 25px;
}

.price-tag {
  text-align: center;
  margin-bottom: 20px;
  padding: 15px;
  background: #1A1F2A;
  border-radius: 16px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.price-label {
  display: block;
  color: rgba(255, 255, 255, 0.6);
  font-size: 14px;
  margin-bottom: 5px;
}

.price-value {
  font-size: 32px;
  font-weight: 900;
  color: #D4AF37;
}

.benefits-list {
  margin-bottom: 20px;
}

.benefit-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 0;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
  color: rgba(255, 255, 255, 0.9);
}

.benefit-item:last-child {
  border-bottom: none;
}

.benefit-item i {
  color: #D4AF37;
  width: 20px;
}

.benefit-item.total-earnings {
  background: rgba(212, 175, 55, 0.1);
  padding: 12px;
  border-radius: 12px;
  margin-top: 10px;
}

.benefit-item.total-earnings strong {
  color: #D4AF37;
  font-size: 16px;
}

/* تذييل البطاقة */
.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
}

.roi-badge {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 8px 12px;
  border-radius: 50px;
  font-size: 13px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 5px;
}

.btn-buy {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 12px 25px;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.btn-buy:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
}

.btn-active {
  background: rgba(212, 175, 55, 0.2);
  color: #D4AF37;
  border: 1px solid #D4AF37;
  padding: 12px 25px;
  border-radius: 12px;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: default;
}

/* شريط نشط */
.active-ribbon {
  position: absolute;
  top: 20px;
  right: -35px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  padding: 8px 40px;
  transform: rotate(45deg);
  font-size: 14px;
  font-weight: 700;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

/* ===== أزرار ===== */
.btn-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 12px 24px;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn-gold:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
}

.btn-gold-outline {
  background: transparent;
  border: 2px solid #D4AF37;
  color: #D4AF37;
  padding: 12px 24px;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn-gold-outline:hover {
  background: #D4AF37;
  color: #0A0C10;
}

/* ===== لوحة المعلومات ===== */
.info-panel {
  background: #11151C;
  border-radius: 24px;
  padding: 25px;
  margin-top: 30px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

.info-item {
  display: flex;
  align-items: flex-start;
  gap: 15px;
}

.info-item i {
  font-size: 24px;
  color: #D4AF37;
  background: rgba(212, 175, 55, 0.1);
  padding: 12px;
  border-radius: 12px;
}

.info-content h4 {
  color: #D4AF37;
  margin-bottom: 5px;
  font-size: 16px;
}

.info-content p {
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  line-height: 1.6;
  margin: 0;
}

/* ===== حالات التحميل ===== */
.center {
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
@media (max-width: 768px) {
  .vip-page {
    padding: 15px;
  }

  .page-title {
    font-size: 24px;
    flex-wrap: wrap;
  }

  .current-vip {
    flex-direction: column;
    text-align: center;
    gap: 20px;
  }

  .vip-stats {
    grid-template-columns: 1fr;
  }

  .current-right {
    width: 100%;
    justify-content: center;
  }

  .vip-grid {
    grid-template-columns: 1fr;
  }

  .info-panel {
    grid-template-columns: 1fr;
  }

  .btn-gold, .btn-gold-outline {
    padding: 10px 20px;
    font-size: 14px;
  }
}

/* ===== تأثيرات إضافية ===== */
.vip-card::after {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(212, 175, 55, 0.05) 0%, transparent 70%);
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.vip-card:hover::after {
  opacity: 1;
}

/* تخصيص ألوان المستويات */
.level-1 .level-icon { color: #CD7F32; } /* برونزي */
.level-2 .level-icon { color: #C0C0C0; } /* فضي */
.level-3 .level-icon { color: #FFD700; } /* ذهبي */
.level-4 .level-icon,
.level-5 .level-icon,
.level-6 .level-icon,
.level-7 .level-icon,
.level-8 .level-icon,
.level-9 .level-icon,
.level-10 .level-icon,
.level-11 .level-icon,
.level-12 .level-icon,
.level-13 .level-icon,
.level-14 .level-icon { color: #D4AF37; } /* ذهبي فاخر */
</style>
