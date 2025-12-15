<template>
  <div class="transactions-wrapper">
    <h2 class="title">المعاملات</h2>

    <!-- إشعار للمستخدم -->
    <div v-if="successMessage" class="success-notification">
      ✅ {{ successMessage }}
      <button @click="successMessage = ''" class="close-notification">✕</button>
    </div>

    <div v-if="loading" class="loading">جاري التحميل...</div>

    <div v-else-if="indexError" class="error-box">
      <h3>⚠️ تحتاج إلى إنشاء فهرس في Firebase</h3>
      <p>لإصلاح المشكلة الدائمة:</p>
      <ol>
        <li>اذهب لـ <a href="https://console.firebase.google.com/" target="_blank">Firebase Console</a></li>
        <li>اختر مشروعك: <strong>american-54cbd</strong></li>
        <li>اذهب لـ Firestore Database → Indexes</li>
        <li>أنشئ فهرس لـ collection "transactions" مع الحقول: userId (Ascending), createdAt (Descending)</li>
        <li>انتظر دقيقتين ثم أعد تحميل هذه الصفحة</li>
      </ol>
      <button @click="loadTransactionsWithoutIndex" class="retry-btn">
        🔄 محاولة التحميل بدون فهرس (مؤقت)
      </button>
    </div>

    <div v-else>
      <div v-if="transactions.length === 0" class="empty">
        <p>لا توجد معاملات</p>
        <p class="uid-info">UID الحالي: {{ currentUserId }}</p>
      </div>

      <div v-else>
        <p class="count-info">عدد المعاملات: {{ transactions.length }}</p>
        
        <div
          v-for="tx in transactions"
          :key="tx.id"
          class="tx-card"
        >
          <!-- ✅ هنا أضفت رمز الموافقة عند حالة "approved" -->
          <div v-if="tx.status === 'approved'" class="approved-badge">
            ✅ تمت الموافقة على طلبك
          </div>

          <div class="row">
            <span class="label">النوع</span>
            <span class="value">{{ typeLabel(tx.type) }}</span>
          </div>

          <div class="row">
            <span class="label">المبلغ</span>
            <span class="value">{{ tx.amount }} USDT</span>
          </div>

          <div class="row">
            <span class="label">الحالة</span>
            <span :class="['status', tx.status]">
              {{ statusLabel(tx.status) }}
            </span>
          </div>

          <div class="row">
            <span class="label">التاريخ</span>
            <span class="value">{{ formatDate(tx.createdAt) }}</span>
          </div>

          <!-- ✅ رسالة تأكيد للموافقة -->
          <div v-if="tx.status === 'approved' && tx.adminMessage" class="approved-message">
            <strong>🎉 تمت الموافقة!</strong>
            <div>{{ tx.adminMessage }}</div>
            <small>تم إضافة المبلغ إلى رصيدك</small>
          </div>

          <div
            v-if="tx.status === 'rejected' && tx.reason"
            class="reject-box"
          >
            <strong>سبب الرفض:</strong>
            <div>{{ tx.reason }}</div>
          </div>

          <div
            v-if="tx.status === 'pending' && tx.adminMessage"
            class="admin-box"
          >
            <strong>رسالة الإدارة:</strong>
            <div>{{ tx.adminMessage }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { auth, db } from "../firebase";
import {
  collection,
  query,
  where,
  orderBy,
  getDocs,
  doc,
  getDoc
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Transactions",

  data() {
    return {
      loading: true,
      transactions: [],
      indexError: false,
      currentUserId: "",
      useIndex: true,
      successMessage: "" // رسالة نجاح
    };
  },

  created() {
    this.loadTransactions();
    this.checkForApprovedTransactions();
  },

  methods: {
    async loadTransactions() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.loading = false;
          console.log("❌ لا يوجد مستخدم مسجل دخول");
          return;
        }

        this.currentUserId = user.uid;
        console.log("🔍 جاري تحميل معاملات المستخدم:", user.uid);

        try {
          // المحاولة الأولى: مع الفهرس (إذا كان موجوداً)
          if (this.useIndex) {
            try {
              const q = query(
                collection(db, "transactions"),
                where("userId", "==", user.uid),
                orderBy("createdAt", "desc")
              );
              
              const snapshot = await getDocs(q);
              this.transactions = snapshot.docs.map(doc => ({
                id: doc.id,
                ...doc.data()
              }));
              
              console.log(`✅ تم تحميل ${this.transactions.length} معاملة باستخدام الفهرس`);
              this.loading = false;
              return;
              
            } catch (indexError) {
              console.log("⚠️ خطأ في الفهرس:", indexError.message);
              this.indexError = true;
              this.useIndex = false;
              // استمر للطريقة البديلة
            }
          }

          // الطريقة البديلة: بدون orderBy (لا تحتاج فهرس)
          try {
            const q = query(
              collection(db, "transactions"),
              where("userId", "==", user.uid)
            );
            
            const snapshot = await getDocs(q);
            let transactions = snapshot.docs.map(doc => ({
              id: doc.id,
              ...doc.data()
            }));
            
            // ترتيب يدوي حسب التاريخ
            transactions.sort((a, b) => {
              const dateA = a.createdAt?.toDate ? a.createdAt.toDate() : new Date(a.createdAt || 0);
              const dateB = b.createdAt?.toDate ? b.createdAt.toDate() : new Date(b.createdAt || 0);
              return dateB - dateA;
            });
            
            this.transactions = transactions;
            console.log(`✅ تم تحميل ${transactions.length} معاملة بدون فهرس`);
            
          } catch (error) {
            console.error("❌ خطأ في الطريقة البديلة:", error);
            this.transactions = [];
          }
          
        } catch (err) {
          console.error("❌ خطأ عام في تحميل المعاملات:", err);
          this.transactions = [];
        }

        this.loading = false;
      });
    },

    // ✅ التحقق إذا كانت هناك معاملات تمت الموافقة عليها حديثاً
    async checkForApprovedTransactions() {
      const user = auth.currentUser;
      if (!user) return;

      try {
        const q = query(
          collection(db, "transactions"),
          where("userId", "==", user.uid),
          where("status", "==", "approved"),
          orderBy("createdAt", "desc")
        );
        
        const snapshot = await getDocs(q);
        const approvedCount = snapshot.size;
        
        if (approvedCount > 0) {
          const latest = snapshot.docs[0].data();
          this.successMessage = `✅ تمت الموافقة على طلبك! المبلغ: ${latest.amount} USDT`;
          
          // تختفي الرسالة بعد 10 ثواني
          setTimeout(() => {
            this.successMessage = "";
          }, 10000);
        }
      } catch (error) {
        console.log("⚠️ لا يمكن التحقق من المعاملات الموافق عليها:", error.message);
      }
    },

    // دالة للتحميل بدون فهرس
    async loadTransactionsWithoutIndex() {
      this.loading = true;
      this.indexError = false;
      this.useIndex = false;
      await this.loadTransactions();
    },

    typeLabel(type) {
      const types = {
        recharge: "تعبئة رصيد",
        withdraw: "سحب رصيد",
        deposit: "إيداع",
        vip: "VIP"
      };
      return types[type] || type;
    },

    statusLabel(status) {
      const statuses = {
        pending: "قيد الانتظار",
        approved: "✅ موافق",
        rejected: "مرفوض"
      };
      return statuses[status] || status;
    },

    formatDate(ts) {
      if (!ts) return "غير متوفر";
      
      try {
        let date;
        if (ts.toDate) {
          date = ts.toDate();
        } else if (ts.seconds) {
          date = new Date(ts.seconds * 1000);
        } else {
          date = new Date(ts);
        }
        
        return date.toLocaleString("ar-EG", {
          year: 'numeric',
          month: 'long',
          day: 'numeric',
          hour: '2-digit',
          minute: '2-digit'
        });
      } catch (error) {
        console.error("خطأ في تنسيق التاريخ:", error, ts);
        return "تاريخ غير صالح";
      }
    },
  },
};
</script>

<style scoped>
.transactions-wrapper {
  padding: 16px;
  min-height: 100vh;
  direction: rtl;
  background: linear-gradient(#0d6efd, #6bb4ff);
}

.title {
  text-align: center;
  color: white;
  margin-bottom: 16px;
  font-size: 24px;
}

.success-notification {
  background: #d4edda;
  color: #155724;
  padding: 12px 20px;
  border-radius: 8px;
  margin-bottom: 20px;
  border: 1px solid #c3e6cb;
  display: flex;
  justify-content: space-between;
  align-items: center;
  animation: slideIn 0.5s ease;
}

.close-notification {
  background: none;
  border: none;
  color: #155724;
  font-size: 18px;
  cursor: pointer;
  padding: 0 5px;
}

@keyframes slideIn {
  from { transform: translateY(-20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.loading {
  text-align: center;
  color: white;
  margin-top: 40px;
  font-size: 18px;
}

.error-box {
  background: #fff3cd;
  border: 1px solid #ffeaa7;
  padding: 20px;
  border-radius: 10px;
  margin: 20px 0;
  color: #856404;
  text-align: right;
}

.error-box ol {
  margin-right: 20px;
}

.error-box a {
  color: #007bff;
  font-weight: bold;
}

.retry-btn {
  background: #28a745;
  color: white;
  border: none;
  padding: 12px 20px;
  border-radius: 8px;
  margin-top: 15px;
  cursor: pointer;
  font-size: 14px;
  width: 100%;
}

.retry-btn:hover {
  background: #218838;
}

.empty {
  text-align: center;
  color: white;
  margin-top: 40px;
  background: rgba(255, 255, 255, 0.1);
  padding: 30px;
  border-radius: 16px;
}

.uid-info {
  font-size: 12px;
  opacity: 0.8;
  margin: 10px 0;
  direction: ltr;
  word-break: break-all;
  background: rgba(0, 0, 0, 0.2);
  padding: 8px;
  border-radius: 8px;
}

.count-info {
  color: white;
  text-align: center;
  margin-bottom: 15px;
  font-weight: bold;
}

.tx-card {
  background: #ffffffee;
  padding: 14px;
  border-radius: 16px;
  margin-bottom: 14px;
  color: #000;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  position: relative;
}

.approved-badge {
  background: #d4edda;
  color: #155724;
  padding: 6px 12px;
  border-radius: 20px;
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
  border: 2px solid #28a745;
}

.approved-message {
  background: #e8f5e9;
  padding: 12px;
  border-radius: 10px;
  margin-top: 12px;
  color: #2e7d32;
  border-right: 4px solid #4CAF50;
}

.approved-message strong {
  display: block;
  margin-bottom: 5px;
  font-size: 14px;
}

.approved-message small {
  display: block;
  margin-top: 5px;
  font-size: 11px;
  opacity: 0.8;
}

.row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 6px;
}

.label {
  font-size: 13px;
  color: #555;
}

.value {
  font-weight: bold;
  color: #333;
}

.status {
  font-weight: bold;
  padding: 2px 8px;
  border-radius: 4px;
}

.status.pending {
  color: #ff9800;
  background-color: #fff3e0;
}

.status.approved {
  color: #2e7d32;
  background-color: #e8f5e9;
  border: 1px solid #4CAF50;
}

.status.rejected {
  color: #d32f2f;
  background-color: #ffebee;
}

.reject-box {
  background: #ffe5e5;
  padding: 8px;
  border-radius: 10px;
  margin-top: 8px;
  color: #b00020;
  font-size: 13px;
  border-right: 4px solid #dc3545;
}

.admin-box {
  background: #e3f2fd;
  padding: 8px;
  border-radius: 10px;
  margin-top: 8px;
  font-size: 13px;
  border-right: 4px solid #2196F3;
}
</style>
