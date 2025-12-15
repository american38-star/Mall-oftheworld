<template>
  <div class="transactions-wrapper">
    <h2 class="title">المعاملات</h2>

    <div v-if="loading" class="loading">جاري التحميل...</div>
    
    <div v-else-if="indexError" class="error-box">
      <h3>⚠️ تحتاج إلى إنشاء فهرس في Firebase</h3>
      <p>لإصلاح المشكلة الدائمة:</p>
      <ol>
        <li>اذهب لـ <a href="https://console.firebase.google.com/" target="_blank">Firebase Console</a></li>
        <li>اختر مشروعك: <strong>american-54cbd</strong></li>
        <li>اذهب لـ Firestore Database → Indexes</li>
        <li>أنشئ فهرس لـ collection "transactions" مع الحقول: userId (Ascending), createdAt (Descending)</li>
      </ol>
      <button @click="loadWithoutOrder" class="retry-btn">
        👇 استعراض المعاملات بدون ترتيب (مؤقت)
      </button>
    </div>

    <div v-else>
      <div v-if="transactions.length === 0" class="empty">
        لا توجد معاملات
      </div>

      <div
        v-for="tx in transactions"
        :key="tx.id"
        class="tx-card"
      >
        <!-- نفس كود العرض السابق -->
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
  getDocs
} from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Transactions",

  data() {
    return {
      loading: true,
      transactions: [],
      indexError: false,
      currentUserId: ""
    };
  },

  created() {
    this.loadTransactions();
  },

  methods: {
    async loadTransactions() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.loading = false;
          return;
        }

        this.currentUserId = user.uid;
        
        try {
          // المحاولة الأولى: مع orderBy (المفترضة)
          const q = query(
            collection(db, "transactions"),
            where("userId", "==", user.uid),
            orderBy("createdAt", "desc") // ← هذا يحتاج فهرس
          );
          
          const snapshot = await getDocs(q);
          this.transactions = snapshot.docs.map(doc => ({
            id: doc.id,
            ...doc.data()
          }));
          
          this.loading = false;
          
        } catch (error) {
          console.error("❌ خطأ في تحميل المعاملات:", error);
          
          // إذا كان الخطأ بسبب الفهرس
          if (error.code === 'failed-precondition' || 
              error.message.includes('index')) {
            this.indexError = true;
          }
          
          this.loading = false;
        }
      });
    },

    // دالة بديلة بدون orderBy (تعمل بدون فهرس)
    async loadWithoutOrder() {
      this.loading = true;
      this.indexError = false;
      
      try {
        const user = auth.currentUser;
        if (!user) return;
        
        const q = query(
          collection(db, "transactions"),
          where("userId", "==", user.uid)
          // ⚠️ بدون orderBy - يعمل بدون فهرس
        );
        
        const snapshot = await getDocs(q);
        let transactions = snapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data()
        }));
        
        // ترتيب يدوي في الكود (بدون فهرس)
        transactions.sort((a, b) => {
          const dateA = a.createdAt?.toDate ? a.createdAt.toDate() : new Date(a.createdAt || 0);
          const dateB = b.createdAt?.toDate ? b.createdAt.toDate() : new Date(b.createdAt || 0);
          return dateB - dateA; // من الأحدث للأقدم
        });
        
        this.transactions = transactions;
        console.log(`✅ تم تحميل ${transactions.length} معاملة (بدون فهرس)`);
        
      } catch (error) {
        console.error("❌ خطأ في التحميل البديل:", error);
        alert("خطأ: " + error.message);
      }
      
      this.loading = false;
    }
  }
};
</script>

<style scoped>
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

/* باقي الـ styles كما هي */
</style>
