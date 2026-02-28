<template>
  <div class="team-page">
    <!-- حالة التحميل -->
    <div v-if="loading" class="loading-box">
      <div class="gold-spinner"></div>
      <p>جاري تحميل بيانات الفريق...</p>
    </div>

    <!-- حالة الخطأ -->
    <div v-if="error" class="error-box">
      <i class="fas fa-exclamation-triangle"></i>
      <p>حدث خطأ أثناء جلب بيانات الفريق:</p>
      <strong>{{ error }}</strong>
      <button @click="$router.go()" class="retry-btn">إعادة المحاولة</button>
    </div>

    <!-- المحتوى الرئيسي -->
    <template v-if="!loading && !error">
      <!-- كود الدعوة -->
      <div class="invite-section">
        <h2>🔗 كود الدعوة الخاص بك</h2>
        
        <div class="ref-box">
          <label>كود الإحالة:</label>
          <div class="ref-code">{{ referralCode || "غير متوفر" }}</div>
          <button @click="copyText(referralCode)" class="gold-btn">
            <i class="fas fa-copy"></i> نسخ
          </button>
        </div>

        <div class="ref-box">
          <label>رابط الدعوة:</label>
          <div class="ref-code">{{ inviteLink || "غير متوفر" }}</div>
          <button @click="copyText(inviteLink)" class="gold-btn">
            <i class="fas fa-copy"></i> نسخ
          </button>
        </div>

        <div class="share-buttons">
          <button @click="shareViaWhatsApp" class="share-btn whatsapp">
            <i class="fab fa-whatsapp"></i> واتساب
          </button>
          <button @click="shareViaTelegram" class="share-btn telegram">
            <i class="fab fa-telegram"></i> تليجرام
          </button>
        </div>
      </div>

      <!-- إحصائيات الفريق -->
      <div class="team-stats-box">
        <h2>📊 إحصائيات الفريق</h2>
        <div class="stats-grid">
          <div class="stat-item">
            <div class="stat-icon">💰</div>
            <div class="stat-title">إجمالي الشحن</div>
            <div class="stat-value">{{ teamStats.recharge }} USDT</div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">💸</div>
            <div class="stat-title">إجمالي السحب</div>
            <div class="stat-value">{{ teamStats.withdraw }} USDT</div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">👥</div>
            <div class="stat-title">حجم الفريق</div>
            <div class="stat-value">{{ teamStats.totalMembers }}</div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">🆕</div>
            <div class="stat-title">أعضاء جدد</div>
            <div class="stat-value">{{ teamStats.newMembers }}</div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">💳</div>
            <div class="stat-title">أول شحن</div>
            <div class="stat-value">{{ teamStats.firstRecharge }}</div>
          </div>

          <div class="stat-item">
            <div class="stat-icon">🏧</div>
            <div class="stat-title">أول سحب</div>
            <div class="stat-value">{{ teamStats.firstWithdraw }}</div>
          </div>
        </div>
      </div>

      <!-- المستويات -->
      <div class="levels-container">
        <h2>📈 مستويات العمولة</h2>
        
        <div class="level-cards">
          <div class="level-card level1">
            <div class="level-header">
              <span class="level-badge">المستوى الأول</span>
              <span class="commission-rate">6%</span>
            </div>
            <div class="level-body">
              <div class="level-count">
                <i class="fas fa-users"></i>
                <span>{{ stats.l1.count }} عضو</span>
              </div>
              <div class="level-earnings">
                <i class="fas fa-coins"></i>
                <span>{{ stats.l1.earnings.toFixed(2) }} USDT</span>
              </div>
            </div>
          </div>

          <div class="level-card level2">
            <div class="level-header">
              <span class="level-badge">المستوى الثاني</span>
              <span class="commission-rate">2%</span>
            </div>
            <div class="level-body">
              <div class="level-count">
                <i class="fas fa-users"></i>
                <span>{{ stats.l2.count }} عضو</span>
              </div>
              <div class="level-earnings">
                <i class="fas fa-coins"></i>
                <span>{{ stats.l2.earnings.toFixed(2) }} USDT</span>
              </div>
            </div>
          </div>

          <div class="level-card level3">
            <div class="level-header">
              <span class="level-badge">المستوى الثالث</span>
              <span class="commission-rate">1%</span>
            </div>
            <div class="level-body">
              <div class="level-count">
                <i class="fas fa-users"></i>
                <span>{{ stats.l3.count }} عضو</span>
              </div>
              <div class="level-earnings">
                <i class="fas fa-coins"></i>
                <span>{{ stats.l3.earnings.toFixed(2) }} USDT</span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- معلومات إضافية -->
      <div class="info-box">
        <i class="fas fa-info-circle"></i>
        <p>يتم تحديث الإحصائيات تلقائياً. العمولات تُحتسب من شحنات الأعضاء المعتمدة فقط.</p>
      </div>

      <button class="btn-back" @click="$router.push('/home')">
        <i class="fas fa-arrow-right"></i> عودة للرئيسية
      </button>
    </template>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  doc,
  getDoc,
  collection,
  query,
  where,
  getDocs
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Team",

  data() {
    return {
      referralCode: "",
      inviteLink: "",
      loading: true,
      error: null,

      userIdFieldInLogs: "userId",
      txIdFieldInLogs: "txid",

      teamStats: {
        withdraw: "0.00",
        recharge: "0.00",
        totalMembers: 0,
        newMembers: 0,
        firstRecharge: 0,
        firstWithdraw: 0,
      },

      stats: {
        l1: { count: 0, earnings: 0 },
        l2: { count: 0, earnings: 0 },
        l3: { count: 0, earnings: 0 },
      },
    };
  },

  created() {
    onAuthStateChanged(auth, async (user) => {
      if (!user) {
        this.loading = false;
        this.$router.push("/login");
        return;
      }

      try {
        const uid = user.uid;
        const udoc = await getDoc(doc(db, "users", uid));
        if (udoc.exists()) {
          const data = udoc.data();
          this.referralCode = data.referralCode || uid.substring(0, 6);
          this.inviteLink = `${window.location.origin}/register?ref=${this.referralCode}`;
        } else {
          this.referralCode = uid.substring(0, 6);
          this.inviteLink = `${window.location.origin}/register?ref=${this.referralCode}`;
        }

        await this.loadTeamLevels(uid);
        await this.loadTeamStats(uid);
      } catch (err) {
        console.error("Team load error:", err);
        this.error = err.message || String(err);
      } finally {
        this.loading = false;
      }
    });
  },

  methods: {
    copyText(text) {
      if (!text) {
        alert("لا يوجد شيء للنسخ");
        return;
      }
      navigator.clipboard
        .writeText(text)
        .then(() => {
          alert("تم النسخ بنجاح!");
        })
        .catch(() => alert("فشل النسخ — انسخ يدويًا"));
    },

    shareViaWhatsApp() {
      const message = `انضم إليّ عبر هذا الرابط: ${this.inviteLink}`;
      window.open(`https://wa.me/?text=${encodeURIComponent(message)}`, '_blank');
    },

    shareViaTelegram() {
      const message = `انضم إليّ عبر هذا الرابط: ${this.inviteLink}`;
      window.open(`https://t.me/share/url?url=${encodeURIComponent(this.inviteLink)}&text=${encodeURIComponent(message)}`, '_blank');
    },

    chunkArray(arr, size = 10) {
      const res = [];
      for (let i = 0; i < arr.length; i += size) {
        res.push(arr.slice(i, i + size));
      }
      return res;
    },

    async loadTeamLevels(uid) {
      try {
        const usersRef = collection(db, "users");

        // L1
        const q1 = query(usersRef, where("invitedBy", "==", uid));
        const s1 = await getDocs(q1);
        const level1Ids = s1.docs.map((d) => d.id);
        this.stats.l1.count = level1Ids.length;

        // L2
        let level2Ids = [];
        if (level1Ids.length) {
          const chunks = this.chunkArray(level1Ids, 10);
          for (const ch of chunks) {
            const q2 = query(usersRef, where("invitedBy", "in", ch));
            const s2 = await getDocs(q2);
            level2Ids = level2Ids.concat(s2.docs.map((d) => d.id));
          }
        }
        this.stats.l2.count = level2Ids.length;

        // L3
        let level3Ids = [];
        if (level2Ids.length) {
          const chunks2 = this.chunkArray(level2Ids, 10);
          for (const ch of chunks2) {
            const q3 = query(usersRef, where("invitedBy", "in", ch));
            const s3 = await getDocs(q3);
            level3Ids = level3Ids.concat(s3.docs.map((d) => d.id));
          }
        }
        this.stats.l3.count = level3Ids.length;

        const allIds = [...level1Ids, ...level2Ids, ...level3Ids];
        const uniqueIds = Array.from(new Set(allIds));
        this.teamStats.totalMembers = uniqueIds.length;
        this.teamStats.newMembers = level1Ids.length;

        await this.loadReferralRewards(uid);
      } catch (err) {
        console.error("loadTeamLevels error:", err);
        throw err;
      }
    },

    async loadReferralRewards(uid) {
      try {
        const rewardsRef = collection(db, "referral_rewards");
        const calc = async (level) => {
          const q = query(rewardsRef, where("receiver", "==", uid), where("level", "==", level));
          const s = await getDocs(q);
          return s.docs.reduce((sum, d) => sum + Number(d.data().amount || 0), 0);
        };

        this.stats.l1.earnings = await calc(1);
        this.stats.l2.earnings = await calc(2);
        this.stats.l3.earnings = await calc(3);
      } catch (err) {
        console.warn("loadReferralRewards warning:", err);
        this.stats.l1.earnings = 0;
        this.stats.l2.earnings = 0;
        this.stats.l3.earnings = 0;
      }
    },

    async loadTeamStats(uid) {
      try {
        const usersRef = collection(db, "users");

        const q1 = query(usersRef, where("invitedBy", "==", uid));
        const s1 = await getDocs(q1);
        const level1Members = s1.docs.map((d) => d.id);

        let level2Members = [];
        if (level1Members.length) {
          for (const ch of this.chunkArray(level1Members, 10)) {
            const q2 = query(usersRef, where("invitedBy", "in", ch));
            const s2 = await getDocs(q2);
            level2Members = level2Members.concat(s2.docs.map((d) => d.id));
          }
        }

        let level3Members = [];
        if (level2Members.length) {
          for (const ch of this.chunkArray(level2Members, 10)) {
            const q3 = query(usersRef, where("invitedBy", "in", ch));
            const s3 = await getDocs(q3);
            level3Members = level3Members.concat(s3.docs.map((d) => d.id));
          }
        }

        const all = [...level1Members, ...level2Members, ...level3Members];
        const membersUnique = Array.from(new Set(all));

        let withdrawSum = 0;
        let rechargeSum = 0;
        let firstWithdrawCount = 0;
        let firstRechargeCount = 0;

        const seenRechargeTx = new Set();
        const seenWithdrawTx = new Set();

        const uidField = this.userIdFieldInLogs;
        const txField = this.txIdFieldInLogs;

        for (const memberId of membersUnique) {
          // withdraw_logs
          const withdrawQ = query(collection(db, "withdraw_logs"), where(uidField, "==", memberId));
          const wSnap = await getDocs(withdrawQ);

          let memberHadWithdraw = false;
          wSnap.forEach((d) => {
            const data = d.data() || {};
            if (data.type && String(data.type).toLowerCase() !== "approved") return;
            if (data.status && String(data.status).toLowerCase() !== "approved") return;

            const key = (data[txField] && String(data[txField])) || d.id;
            if (seenWithdrawTx.has(key)) return;
            seenWithdrawTx.add(key);

            const amt = Number(data.amount || 0);
            if (!isNaN(amt) && amt !== 0) {
              withdrawSum += amt;
              memberHadWithdraw = true;
            }
          });
          if (memberHadWithdraw) firstWithdrawCount++;

          // recharge_logs
          const rechargeQ = query(collection(db, "recharge_logs"), where(uidField, "==", memberId));
          const rSnap = await getDocs(rechargeQ);

          let memberHadRecharge = false;
          rSnap.forEach((d) => {
            const data = d.data() || {};
            if (data.type && String(data.type).toLowerCase() !== "approved") return;
            if (data.status && String(data.status).toLowerCase() !== "approved") return;

            const key = (data[txField] && String(data[txField])) || d.id;
            if (seenRechargeTx.has(key)) return;
            seenRechargeTx.add(key);

            const amt = Number(data.amount || 0);
            if (!isNaN(amt) && amt !== 0) {
              rechargeSum += amt;
              memberHadRecharge = true;
            }
          });
          if (memberHadRecharge) firstRechargeCount++;
        }

        this.teamStats.withdraw = parseFloat(withdrawSum || 0).toFixed(2);
        this.teamStats.recharge = parseFloat(rechargeSum || 0).toFixed(2);
        this.teamStats.firstWithdraw = firstWithdrawCount;
        this.teamStats.firstRecharge = firstRechargeCount;
        this.teamStats.totalMembers = membersUnique.length;
      } catch (err) {
        console.error("loadTeamStats error:", err);
        this.error = err.message || String(err);
      }
    },
  },
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.team-page {
  direction: rtl;
  min-height: 100vh;
  background: #0A0C10;
  padding: 20px;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
  padding-bottom: 100px; /* مساحة للشريط السفلي */
  max-width: 600px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
}

/* العناوين الرئيسية */
h2 {
  color: #D4AF37;
  font-size: 22px;
  margin-bottom: 20px;
  text-align: center;
  position: relative;
  padding-bottom: 10px;
}

h2::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 3px;
  background: linear-gradient(90deg, transparent, #D4AF37, #F6E27A, #D4AF37, transparent);
  border-radius: 3px;
}

/* ===== قسم الدعوة ===== */
.invite-section {
  background: #11151C;
  border-radius: 24px;
  padding: 25px 20px;
  margin-bottom: 25px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  width: 100%;
  box-sizing: border-box;
}

.ref-box {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 15px 0;
  flex-wrap: wrap;
  width: 100%;
}

.ref-box label {
  color: #D4AF37;
  font-weight: 600;
  min-width: 100px;
  font-size: 14px;
}

.ref-code {
  flex: 1;
  background: #1A1F2A;
  padding: 12px 15px;
  border-radius: 12px;
  color: #ffffff;
  font-family: monospace;
  font-size: 14px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  word-break: break-all;
  min-width: 150px;
}

.gold-btn {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  border: none;
  color: #0A0C10;
  padding: 10px 20px;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  white-space: nowrap;
}

.gold-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.share-buttons {
  display: flex;
  gap: 15px;
  margin-top: 20px;
  justify-content: center;
  width: 100%;
}

.share-btn {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 12px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.3s ease;
}

.share-btn.whatsapp {
  background: #25D366;
  color: white;
}

.share-btn.telegram {
  background: #0088cc;
  color: white;
}

.share-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

/* ===== إحصائيات الفريق ===== */
.team-stats-box {
  background: #11151C;
  border-radius: 24px;
  padding: 25px 20px;
  margin-bottom: 25px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  width: 100%;
  box-sizing: border-box;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  width: 100%;
}

.stat-item {
  background: #1A1F2A;
  padding: 15px 10px;
  border-radius: 16px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.1);
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.stat-item:hover {
  border-color: #D4AF37;
  transform: translateY(-3px);
}

.stat-icon {
  font-size: 28px;
  margin-bottom: 8px;
}

.stat-title {
  color: rgba(255, 255, 255, 0.7);
  font-size: 12px;
  margin-bottom: 5px;
  white-space: nowrap;
}

.stat-value {
  color: #D4AF37;
  font-size: 16px;
  font-weight: 700;
  word-break: break-word;
}

/* ===== المستويات ===== */
.levels-container {
  background: #11151C;
  border-radius: 24px;
  padding: 25px 20px;
  margin-bottom: 25px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  width: 100%;
  box-sizing: border-box;
}

.level-cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  width: 100%;
}

.level-card {
  background: #1A1F2A;
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid rgba(212, 175, 55, 0.2);
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
}

.level-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.2);
}

.level-header {
  background: linear-gradient(135deg, #D4AF37, #C5A028);
  padding: 12px 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #0A0C10;
}

.level-badge {
  font-weight: 700;
  font-size: 12px;
}

.commission-rate {
  background: rgba(10, 12, 16, 0.2);
  padding: 4px 6px;
  border-radius: 20px;
  font-weight: 700;
  font-size: 11px;
}

.level-body {
  padding: 15px 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.level-count, .level-earnings {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #ffffff;
  font-size: 13px;
}

.level-count i, .level-earnings i {
  color: #D4AF37;
  width: 18px;
  font-size: 14px;
}

.level1 .level-header { background: linear-gradient(135deg, #D4AF37, #F6E27A); }
.level2 .level-header { background: linear-gradient(135deg, #C5A028, #D4AF37); }
.level3 .level-header { background: linear-gradient(135deg, #B8962E, #C5A028); }

/* ===== صندوق المعلومات ===== */
.info-box {
  background: rgba(212, 175, 55, 0.1);
  border-right: 4px solid #D4AF37;
  padding: 15px;
  border-radius: 12px;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
  box-sizing: border-box;
}

.info-box i {
  color: #D4AF37;
  font-size: 20px;
  flex-shrink: 0;
}

.info-box p {
  color: rgba(255, 255, 255, 0.9);
  font-size: 13px;
  margin: 0;
  line-height: 1.5;
}

/* ===== زر العودة ===== */
.btn-back {
  width: 100%;
  padding: 16px;
  background: transparent;
  border: 2px solid #D4AF37;
  color: #D4AF37;
  border-radius: 16px;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
  box-sizing: border-box;
}

.btn-back:hover {
  background: #D4AF37;
  color: #0A0C10;
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.3);
}

/* ===== حالات التحميل والخطأ ===== */
.loading-box, .error-box {
  background: #11151C;
  border-radius: 24px;
  padding: 40px 20px;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.2);
  max-width: 400px;
  margin: 50px auto;
  width: 90%;
}

.gold-spinner {
  width: 50px;
  height: 50px;
  border: 4px solid rgba(212, 175, 55, 0.1);
  border-top: 4px solid #D4AF37;
  border-radius: 50%;
  margin: 0 auto 20px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error-box i {
  font-size: 48px;
  color: #ff6b6b;
  margin-bottom: 15px;
}

.error-box strong {
  color: #ff6b6b;
  display: block;
  margin: 10px 0;
}

.retry-btn {
  background: #D4AF37;
  border: none;
  color: #0A0C10;
  padding: 12px 30px;
  border-radius: 12px;
  font-weight: 700;
  margin-top: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.retry-btn:hover {
  background: #F6E27A;
  transform: translateY(-2px);
}

/* ===== تحسينات للجوال ===== */
@media (max-width: 600px) {
  .team-page {
    padding: 12px;
    padding-bottom: 90px;
  }

  .ref-box {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .ref-box label {
    min-width: auto;
    margin-bottom: 5px;
  }

  .ref-code {
    width: 100%;
    font-size: 13px;
    padding: 10px;
  }

  .gold-btn {
    width: 100%;
    justify-content: center;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }

  .stat-item {
    padding: 12px 8px;
  }

  .stat-icon {
    font-size: 24px;
  }

  .stat-title {
    font-size: 11px;
  }

  .stat-value {
    font-size: 14px;
  }

  .level-cards {
    grid-template-columns: 1fr;
    gap: 12px;
  }

  .level-card {
    width: 100%;
  }

  .level-header {
    padding: 10px;
  }

  .level-badge {
    font-size: 13px;
  }

  .commission-rate {
    font-size: 12px;
    padding: 4px 8px;
  }

  .level-body {
    padding: 12px;
    flex-direction: row;
    justify-content: space-around;
  }

  .level-count, .level-earnings {
    font-size: 14px;
  }

  .share-buttons {
    flex-direction: column;
    gap: 10px;
  }

  .share-btn {
    width: 100%;
  }

  .info-box {
    padding: 12px;
  }

  .info-box i {
    font-size: 18px;
  }

  .info-box p {
    font-size: 12px;
  }

  .btn-back {
    padding: 14px;
    font-size: 16px;
  }
}

@media (max-width: 350px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .stat-item {
    flex-direction: row;
    justify-content: space-between;
    text-align: right;
    padding: 10px 15px;
  }

  .stat-icon {
    margin-bottom: 0;
    margin-left: 10px;
  }

  .stat-title, .stat-value {
    display: inline-block;
  }
}
</style>
