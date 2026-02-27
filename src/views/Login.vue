<template>
  <div class="container">
    <div class="card">
      <h2 class="title">تسجيل الدخول</h2>

      <!-- البريد الإلكتروني -->
      <label class="label">البريد الإلكتروني</label>
      <input
        type="email"
        v-model="email"
        placeholder="البريد الإلكتروني"
        class="input"
      />

      <!-- كلمة المرور -->
      <label class="label">كلمة المرور</label>
      <div class="input-box">
        <input
          :type="showPassword ? 'text' : 'password'"
          v-model="password"
          placeholder="كلمة المرور"
          class="input"
        />
        <span class="toggle" @click="togglePassword">
          {{ showPassword ? "إخفاء" : "إظهار" }}
        </span>
      </div>

      <!-- زر تسجيل الدخول مع Loader -->
      <button class="btn" @click="loginUser" :disabled="loading">
        <span v-if="!loading">تسجيل الدخول</span>
        <span v-else class="loader"></span>
      </button>

      <p class="link">
        ليس لديك حساب؟
        <router-link to="/register">إنشاء حساب</router-link>
      </p>
    </div>
  </div>

  <!-- Popup إعلان -->
  <div id="companyAd" class="ad-overlay" v-if="showAd">
    <div class="ad-box">
      <h2>إعلان</h2>

      <div class="ad-content">
        <p>
          🎉🎉🎉🎉 مرحبا بالجميع! تأسست Mall of the world في سنغافورة في 20 أغسطس 2021 ومقرها حاليًا في منطقة الأعمال المركزية في سنغافورة. نحن شركة استثمار في التجارة الإلكترونية مع فريق تقني قوي وقوة مالية قوية.
          <br>
          يتعاون Mall of the world مع عشرات شركات التجارة الإلكترونية مثل Amazon و eBay و Tiktok و Aliexpress و Alibaba و Shopee ، إلخ. لمساعدة التجار على زيادة مبيعات المنتجات الخاصة بهم ، ويمكننا أيضًا تحقيق أرباح منه. عندما تتصاعد على منصتنا ، تشارك في مساعدة البائعين على زيادة المبيعات ، بحيث يمكنك أيضًا كسب المال منها. حتى يتمكن الجميع من إعادة الشحن بثقة ، هذا مشروع جيد لجني الأموال. 🔇🔇🔇
          <br><br>
          👍1: الحد الأدنى لمبلغ إعادة الشحن: 12 USDT ، الحد الأدنى للسحب النقدي: 3 USDT
          <br>
          💰2: تستثمر المنصة على مستوى العالم ، لذا فإن الاستثمار يدعم فقط إعادة شحن العملة المشفرة.
          <br>
          🌈3: وقت إعادة تعيين المهمة هو الساعة 12 ظهراً في سنغافورة. يمكنك الحصول على الربح من خلال استكمال أوامر التاجر كل يوم (مرة واحدة في اليوم ، صالحة لمدة 365 يومًا).
          <br>
          🕯4: يمكنك سحب النقد مرة واحدة فقط في اليوم ، لا يوجد حد زمني ، يمكنك سحب النقد في أي وقت ، ووقت الانسحاب هو 1 إلى 5 دقائق ، والحد الأدنى لمبلغ السحب هو 3 USDT ، ولا يوجد حد أعلى.
          <br><br>
          عندما يصل مبلغ إعادة الشحن إلى المبلغ المقابل التالي ، سيتم ترقية الحساب تلقائيًا إلى VIP. كلما زادت مبلغ إعادة الشحن ، كلما زاد عدد USDT في اليوم!
          <br><br>
          👍vip1: إعادة شحن 12 USDT ، الإيرادات اليومية 3 USDT
          <br>
          👍vip2: إعادة الشحن 52 USDT ، الدخل اليومي 13 USDT
          <br>
          👍vip3: إعادة شحن 100 USDT ، الدخل اليومي 26 USDT
          <br>
          👍vip4: إعادة شحن 300 USDT ، الدخل اليومي 82 USDT
          <br>
          👍VIP5: إعادة شحن 500 USDT ، الدخل اليومي 145 USDT
          <br>
          👍vip6: إعادة شحن 1500 USDT ، الدخل اليومي 479 USDT
          <br>
          👍VIP7: إعادة شحن 3000 USDT ، الدخل اليومي 1078 USDT
          <br>
          👍VIP8: إعادة شحن 5000 USDT ، الدخل اليومي 2000 USDT
          <br>
          👍vip9: إعادة شحن 10000 USDT ، الدخل اليومي USDT
          <br>
          👍VIP10: إعادة شحن 30000 USDT ، الدخل اليومي 17699 USDT
          <br>
          👍VIP11: إعادة شحن 90،000 دولار أمريكي ، الدخل اليومي 81،818 USDT
          <br><br>
          يمكن للمستخدمين Mall of the world الترويج لنظامنا الأساسي من خلال روابط التوصية ودعوة أصدقائك للانضمام إلينا على Facebook و Twitter و Instagram و YouTube و Tiktok و Kaokao و WhatsApp و Telegram. عندما يسجل شخص ما وينتهي من Top-Up ، تحصل على مكافأة تصل إلى 9 ٪.
          <br><br>
          🤝team المستوى 3 إعادة شحن مكافأة تصل إلى 9 ٪
          <br>
          🤝level 1 مكافأة إعادة شحن الأعضاء: 6 ٪
          <br>
          🤝level 2 مكافأة إعادة شحن الأعضاء: 2 ٪
          <br>
          🤝level 3 مكافأة إعادة شحن الأعضاء: 1 ٪
          <br>
          🤑team إعادة شحن المستوى الأول البالغ 1000 دولار أمريكي ، ويمكنك الحصول على 60 USDT
          <br>
          🤑team إعادة الشحن الثانوي البالغ 1000 دولار أمريكي ، يمكنك الحصول على 20 USDT
          <br>
          🤑team المستوى 3 إعادة شحن 1000 USDT ، ويمكنك الحصول على 10 USDT
        </p>
      </div>

      <button @click="closeAd">أنا أعرف</button>
    </div>
  </div>

</template>

<script>
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
import { doc, getDoc } from "firebase/firestore";

// ✔ المسار الصحيح
import { db } from "../firebase";

// ✔ المسار الصحيح للروتر
import router from "../router";

export default {
  data() {
    return {
      email: "",
      password: "",
      showPassword: false,
      loading: false,
      showAd: false,  // تحديد حالة ظهور الإعلان
    };
  },

  methods: {
    togglePassword() {
      this.showPassword = !this.showPassword;
    },

    async loginUser() {
      if (!this.email || !this.password) {
        return alert("يرجى تعبئة الحقول");
      }

      this.loading = true;

      try {
        const auth = getAuth();

        // تسجيل الدخول
        await signInWithEmailAndPassword(auth, this.email, this.password);

        const user = auth.currentUser;
        if (!user) {
          alert("حدث خطأ غير متوقع");
          return;
        }

        // جلب بيانات المستخدم
        const userRef = doc(db, "users", user.uid);
        const userSnap = await getDoc(userRef);

        // 🔥 التحقق من الحظر
        if (userSnap.exists() && userSnap.data().blocked === true) {
          alert("🚫 حسابك محظور. يرجى التواصل مع الدعم.");
          await auth.signOut();
          return;
        }

        // 🔥 تسجيل دخول الأدمن
        if (
          this.email === "azad.333388@gmail.com" &&
          this.password === "admin0997408001AZAD"
        ) {
          return router.push("/admin");
        }

        // 🔥 تسجيل دخول المستخدم العادي
        router.push("/home");

        // إظهار الإعلان بعد تسجيل الدخول
        this.showAd = true; // تفعيل عرض الإعلان

      } catch (error) {
        alert("خطأ: " + error.message);
      } finally {
        this.loading = false;
      }
    },

    closeAd() {
      this.showAd = false; // إغلاق الإعلان
    }
  },
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  min-height: 100vh;
  background: #1a1a1a; /* أسود احترافي غير مزعج للعين */
}

.card {
  background: white;
  width: 90%;
  max-width: 380px;
  padding: 25px;
  border-radius: 20px;
  box-shadow: 0 4px 12px #0003;
}

.title {
  text-align: center;
  margin-bottom: 20px;
  font-weight: bold;
}

.label {
  display: block;
  margin-bottom: 5px;
  font-weight: 600;
}

.input {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border-radius: 10px;
  border: 1px solid #ccc;
}

.input-box {
  position: relative;
}

.toggle {
  position: absolute;
  left: 15px;
  top: 12px;
  color: #0d6efd;
  cursor: pointer;
  font-size: 14px;
}

.btn {
  width: 100%;
  padding: 12px;
  border: none;
  background: #007bff;
  color: white;
  border-radius: 8px;
  font-size: 18px;
  cursor: pointer;
}

.link {
  text-align: center;
  margin-top: 15px;
}

.loader {
  width: 20px;
  height: 20px;
  border: 3px solid #fff;
  border-top: 3px solid transparent;
  border-radius: 50%;
  display: inline-block;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

/* إعلان */
.ad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.ad-box {
  background: #eaf2ff;
  width: 90%;
  max-width: 400px;
  margin: 15% auto;
  border-radius: 15px;
  padding: 15px;
  text-align: center;
}

.ad-box h2 {
  background: #3b82f6;
  color: white;
  padding: 10px;
  border-radius: 10px;
}

.ad-content {
  max-height: 250px;
  overflow-y: auto;
  margin: 10px 0;
  font-size: 14px;
}

.ad-box button {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
}
</style>
