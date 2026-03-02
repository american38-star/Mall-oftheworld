<template>
  <div class="container">
    <div class="card">
      <h2 class="title">تسجيل الدخول</h2>

      <!-- اختيار طريقة تسجيل الدخول -->
      <div class="login-type-selector">
        <button 
          class="type-btn" 
          :class="{ active: loginType === 'email' }"
          @click="loginType = 'email'"
        >
          <i class="fas fa-envelope"></i>
          البريد الإلكتروني
        </button>
        <button 
          class="type-btn" 
          :class="{ active: loginType === 'phone' }"
          @click="loginType = 'phone'"
        >
          <i class="fas fa-phone"></i>
          رقم الهاتف
        </button>
      </div>

      <!-- تسجيل الدخول بالبريد الإلكتروني -->
      <template v-if="loginType === 'email'">
        <label class="label">البريد الإلكتروني</label>
        <input
          type="email"
          v-model="email"
          placeholder="example@email.com"
          class="input"
          @keyup.enter="loginUser"
        />
      </template>

      <!-- تسجيل الدخول برقم الهاتف -->
      <template v-if="loginType === 'phone'">
        <label class="label">رقم الهاتف مع رمز الدولة</label>
        <div class="phone-input-container">
          <select v-model="countryCode" class="country-select">
            <option value="">اختر الرمز</option>
            <option value="+964">🇮🇶 العراق (+964)</option>
            <option value="+966">🇸🇦 السعودية (+966)</option>
            <option value="+971">🇦🇪 الإمارات (+971)</option>
            <option value="+965">🇰🇼 الكويت (+965)</option>
            <option value="+974">🇶🇦 قطر (+974)</option>
            <option value="+973">🇧🇭 البحرين (+973)</option>
            <option value="+968">🇴🇲 عمان (+968)</option>
            <option value="+962">🇯🇴 الأردن (+962)</option>
            <option value="+20">🇪🇬 مصر (+20)</option>
            <option value="+963">🇸🇾 سوريا (+963)</option>
            <option value="+961">🇱🇧 لبنان (+961)</option>
            <option value="+218">🇱🇾 ليبيا (+218)</option>
            <option value="+216">🇹🇳 تونس (+216)</option>
            <option value="+213">🇩🇿 الجزائر (+213)</option>
            <option value="+212">🇲🇦 المغرب (+212)</option>
            <option value="+222">🇲🇷 موريتانيا (+222)</option>
            <option value="+249">🇸🇩 السودان (+249)</option>
            <option value="+967">🇾🇪 اليمن (+967)</option>
            <option value="+970">🇵🇸 فلسطين (+970)</option>
            <option value="+90">🇹🇷 تركيا (+90)</option>
            <option value="+44">🇬🇧 بريطانيا (+44)</option>
            <option value="+1">🇺🇸 أمريكا (+1)</option>
            <option value="+49">🇩🇪 ألمانيا (+49)</option>
            <option value="+33">🇫🇷 فرنسا (+33)</option>
            <option value="+39">🇮🇹 إيطاليا (+39)</option>
            <option value="+34">🇪🇸 إسبانيا (+34)</option>
            <option value="+31">🇳🇱 هولندا (+31)</option>
            <option value="+46">🇸🇪 السويد (+46)</option>
            <option value="+47">🇳🇴 النرويج (+47)</option>
            <option value="+45">🇩🇰 الدنمارك (+45)</option>
            <option value="+358">🇫🇮 فنلندا (+358)</option>
            <option value="+41">🇨🇭 سويسرا (+41)</option>
            <option value="+43">🇦🇹 النمسا (+43)</option>
            <option value="+32">🇧🇪 بلجيكا (+32)</option>
            <option value="+48">🇵🇱 بولندا (+48)</option>
            <option value="+420">🇨🇿 التشيك (+420)</option>
            <option value="+36">🇭🇺 المجر (+36)</option>
            <option value="+40">🇷🇴 رومانيا (+40)</option>
            <option value="+359">🇧🇬 بلغاريا (+359)</option>
            <option value="+30">🇬🇷 اليونان (+30)</option>
            <option value="+351">🇵🇹 البرتغال (+351)</option>
            <option value="+7">🇷🇺 روسيا (+7)</option>
            <option value="+380">🇺🇦 أوكرانيا (+380)</option>
            <option value="+375">🇧🇾 بيلاروسيا (+375)</option>
            <option value="+995">🇬🇪 جورجيا (+995)</option>
            <option value="+994">🇦🇿 أذربيجان (+994)</option>
            <option value="+374">🇦🇲 أرمينيا (+374)</option>
            <option value="+998">🇺🇿 أوزبكستان (+998)</option>
            <option value="+996">🇰🇬 قرغيزستان (+996)</option>
            <option value="+992">🇹🇯 طاجيكستان (+992)</option>
            <option value="+993">🇹🇲 تركمانستان (+993)</option>
            <option value="+86">🇨🇳 الصين (+86)</option>
            <option value="+91">🇮🇳 الهند (+91)</option>
            <option value="+92">🇵🇰 باكستان (+92)</option>
            <option value="+93">🇦🇫 أفغانستان (+93)</option>
            <option value="+94">🇱🇰 سريلانكا (+94)</option>
            <option value="+95">🇲🇲 ميانمار (+95)</option>
            <option value="+66">🇹🇭 تايلاند (+66)</option>
            <option value="+84">🇻🇳 فيتنام (+84)</option>
            <option value="+60">🇲🇾 ماليزيا (+60)</option>
            <option value="+65">🇸🇬 سنغافورة (+65)</option>
            <option value="+62">🇮🇩 إندونيسيا (+62)</option>
            <option value="+63">🇵🇭 الفلبين (+63)</option>
            <option value="+82">🇰🇷 كوريا الجنوبية (+82)</option>
            <option value="+81">🇯🇵 اليابان (+81)</option>
          </select>
          <input
            type="tel"
            v-model="phoneNumber"
            placeholder="رقم الهاتف"
            class="phone-input"
            :disabled="!countryCode"
            @keyup.enter="loginUser"
          />
        </div>
        <span v-if="phoneError" class="error-message">{{ phoneError }}</span>
      </template>

      <!-- كلمة المرور -->
      <label class="label">كلمة المرور</label>
      <div class="input-box">
        <input
          :type="showPassword ? 'text' : 'password'"
          v-model="password"
          placeholder="كلمة المرور"
          class="input"
          @keyup.enter="loginUser"
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
      <h2>✨ إعلان ✨</h2>
      <div class="ad-content">
        <p>🎉🎉🎉🎉 مرحبا بالجميع! تأسست Mall of the world في سنغافورة في 20 أغسطس 2021 ومقرها حاليًا في منطقة الأعمال المركزية في سنغافورة. نحن شركة استثمار في التجارة الإلكترونية مع فريق تقني قوي وقوة مالية قوية.
          <br><br>
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
        </p>
      </div>
      <button @click="closeAd" class="ad-btn">أنا أعرف</button>
    </div>
  </div>
</template>

<script>
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";
import { collection, query, where, getDocs, doc, getDoc } from "firebase/firestore";
import { db } from "../firebase";
import router from "../router";

export default {
  data() {
    return {
      loginType: 'email', // 'email' أو 'phone'
      email: "",
      countryCode: "",
      phoneNumber: "",
      fullPhoneNumber: "",
      password: "",
      showPassword: false,
      loading: false,
      showAd: false,
      adTimer: null,
      phoneError: "",
    };
  },

  mounted() {
    document.addEventListener('keydown', this.handleEscKey);
  },

  beforeUnmount() {
    document.removeEventListener('keydown', this.handleEscKey);
    if (this.adTimer) {
      clearTimeout(this.adTimer);
    }
  },

  methods: {
    togglePassword() {
      this.showPassword = !this.showPassword;
    },

    validateEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },

    validatePhoneNumber() {
      if (!this.countryCode) {
        this.phoneError = "الرجاء اختيار رمز الدولة";
        return false;
      }

      if (!this.phoneNumber) {
        this.phoneError = "الرجاء إدخال رقم الهاتف";
        return false;
      }

      const cleanPhone = this.phoneNumber.replace(/[^0-9]/g, '');
      if (cleanPhone.length < 7 || cleanPhone.length > 15) {
        this.phoneError = "رقم الهاتف يجب أن يكون بين 7 و 15 رقم";
        return false;
      }

      this.fullPhoneNumber = this.countryCode + cleanPhone;
      this.phoneError = "";
      return true;
    },

    async findUserByPhone(phoneNumber) {
      try {
        // البحث عن المستخدم برقم الهاتف الكامل
        const q = query(
          collection(db, "users"),
          where("phoneNumber", "==", phoneNumber)
        );
        const querySnapshot = await getDocs(q);
        
        if (!querySnapshot.empty) {
          return querySnapshot.docs[0].data().email;
        }

        // البحث بدون الصفر الأول بعد رمز الدولة
        const phoneWithoutZero = phoneNumber.replace(/^(\+\d+)0(\d+)$/, '$1$2');
        if (phoneWithoutZero !== phoneNumber) {
          const q2 = query(
            collection(db, "users"),
            where("phoneNumber", "==", phoneWithoutZero)
          );
          const querySnapshot2 = await getDocs(q2);
          if (!querySnapshot2.empty) {
            return querySnapshot2.docs[0].data().email;
          }
        }

        return null;
      } catch (error) {
        console.error("خطأ في البحث عن رقم الهاتف:", error);
        return null;
      }
    },

    getErrorMessage(error) {
      const errorMessages = {
        'auth/user-not-found': 'البريد الإلكتروني أو رقم الهاتف غير مسجل',
        'auth/wrong-password': 'كلمة المرور غير صحيحة',
        'auth/invalid-email': 'البريد الإلكتروني غير صالح',
        'auth/user-disabled': 'هذا الحساب معطل'
      };
      return errorMessages[error.code] || 'حدث خطأ غير متوقع: ' + error.message;
    },

    handleEscKey(event) {
      if (event.key === 'Escape' && this.showAd) {
        this.closeAd();
      }
    },

    async loginUser() {
      // التحقق من المدخلات
      if (this.loginType === 'email') {
        if (!this.email || !this.password) {
          return alert("يرجى تعبئة البريد الإلكتروني وكلمة المرور");
        }
        if (!this.validateEmail(this.email)) {
          return alert("البريد الإلكتروني غير صالح");
        }
      } else {
        if (!this.validatePhoneNumber()) {
          return alert(this.phoneError || "يرجى تعبئة رقم الهاتف وكلمة المرور");
        }
        if (!this.password) {
          return alert("يرجى تعبئة كلمة المرور");
        }
      }

      this.loading = true;

      try {
        const auth = getAuth();
        let loginEmail = this.email;

        // إذا كان تسجيل الدخول برقم الهاتف، نبحث عن البريد الإلكتروني المرتبط به
        if (this.loginType === 'phone') {
          const foundEmail = await this.findUserByPhone(this.fullPhoneNumber);
          if (!foundEmail) {
            alert("رقم الهاتف غير مسجل");
            this.loading = false;
            return;
          }
          loginEmail = foundEmail;
        }

        // محاولة تسجيل الدخول
        await signInWithEmailAndPassword(auth, loginEmail, this.password);

        const user = auth.currentUser;
        if (!user) {
          alert("حدث خطأ غير متوقع");
          return;
        }

        const userRef = doc(db, "users", user.uid);
        const userSnap = await getDoc(userRef);

        if (userSnap.exists() && userSnap.data().blocked === true) {
          alert("🚫 حسابك محظور. يرجى التواصل مع الدعم.");
          await auth.signOut();
          return;
        }

        // تسجيل دخول الأدمن
        if (loginEmail === "azad.333388@gmail.com" && this.password === "admin0997408001AZAD") {
          return router.push("/admin");
        }

        // تسجيل دخول المستخدم العادي
        router.push("/home");

        // إظهار الإعلان بعد تسجيل الدخول
        this.showAd = true;
        this.adTimer = setTimeout(() => {
          this.closeAd();
        }, 30000);

      } catch (error) {
        alert(this.getErrorMessage(error));
      } finally {
        this.loading = false;
      }
    },

    closeAd() {
      this.showAd = false;
      if (this.adTimer) {
        clearTimeout(this.adTimer);
        this.adTimer = null;
      }
    }
  }
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  min-height: 100vh;
  background: #0A0C10;
  direction: rtl;
}

/* كرت تسجيل الدخول - رمادي غامق */
.card {
  background: #11151C;
  width: 90%;
  max-width: 420px;
  padding: 35px 25px;
  border-radius: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 1px rgba(212, 175, 55, 0.1);
  border: 1px solid rgba(212, 175, 55, 0.2);
  position: relative;
}

.card::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: linear-gradient(135deg, #D4AF37, transparent, #C5A028);
  border-radius: 22px;
  z-index: -1;
  opacity: 0.3;
}

/* العنوان - ذهبي فاخر */
.title {
  text-align: center;
  margin-bottom: 25px;
  font-weight: 700;
  font-size: 28px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-shadow: 0 2px 10px rgba(212, 175, 55, 0.3);
}

/* محدد نوع تسجيل الدخول */
.login-type-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 25px;
}

.type-btn {
  flex: 1;
  padding: 12px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.type-btn i {
  font-size: 16px;
  color: #D4AF37;
}

.type-btn:hover {
  border-color: #D4AF37;
  color: #D4AF37;
}

.type-btn.active {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border-color: transparent;
}

.type-btn.active i {
  color: #0A0C10;
}

/* تسميات الحقول */
.label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #D4AF37;
  font-size: 14px;
  letter-spacing: 0.5px;
}

/* حاوية رقم الهاتف */
.phone-input-container {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

/* اختيار رمز الدولة */
.country-select {
  width: 40%;
  padding: 14px 10px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 14px;
  transition: all 0.3s ease;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='16' height='16' viewBox='0 0 24 24' fill='none' stroke='%23D4AF37' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: left 12px center;
  background-size: 16px;
  padding-left: 35px;
}

.country-select:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
}

.country-select option {
  background: #1A1F2A;
  color: #ffffff;
  padding: 10px;
}

/* حقل رقم الهاتف */
.phone-input {
  width: 60%;
  padding: 14px 12px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 15px;
  transition: all 0.3s ease;
}

.phone-input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
  background: #1E2430;
}

.phone-input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.phone-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

/* حقول الإدخال العادية */
.input {
  width: 100%;
  padding: 14px 12px;
  margin-bottom: 20px;
  border-radius: 12px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  background: #1A1F2A;
  color: #ffffff;
  font-size: 15px;
  transition: all 0.3s ease;
  box-sizing: border-box;
}

.input:focus {
  outline: none;
  border-color: #D4AF37;
  box-shadow: 0 0 0 3px rgba(212, 175, 55, 0.1);
  background: #1E2430;
}

.input::placeholder {
  color: rgba(255, 255, 255, 0.3);
}

/* حقل كلمة المرور مع زر الإظهار/الإخفاء */
.input-box {
  position: relative;
  width: 100%;
  margin-bottom: 20px;
}

.toggle {
  position: absolute;
  left: 15px;
  top: 12px;
  color: #D4AF37;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.3s ease;
  padding: 2px 8px;
  border-radius: 6px;
  background: rgba(212, 175, 55, 0.1);
}

.toggle:hover {
  background: rgba(212, 175, 55, 0.2);
  color: #F6E27A;
}

/* رسالة خطأ الهاتف */
.error-message {
  color: #ff6b6b;
  font-size: 12px;
  margin-top: -15px;
  margin-bottom: 10px;
  text-align: right;
  display: block;
}

/* زر تسجيل الدخول - ذهبي متدرج */
.btn {
  width: 100%;
  padding: 14px;
  border: none;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 10px 0 15px;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.btn:hover:not(:disabled) {
  background: linear-gradient(135deg, #B8962E, #D4AF37, #B8962E);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(212, 175, 55, 0.4);
}

.btn:active:not(:disabled) {
  transform: translateY(0);
}

.btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  filter: grayscale(20%);
}

/* رابط إنشاء حساب */
.link {
  text-align: center;
  margin-top: 20px;
  color: rgba(255, 255, 255, 0.6);
  font-size: 15px;
}

.link a {
  color: #D4AF37;
  text-decoration: none;
  font-weight: 600;
  margin-right: 5px;
  transition: all 0.3s ease;
  border-bottom: 1px solid transparent;
}

.link a:hover {
  color: #F6E27A;
  border-bottom-color: #D4AF37;
}

/* Loader - ذهبي */
.loader {
  width: 22px;
  height: 22px;
  border: 3px solid #0A0C10;
  border-top: 3px solid #D4AF37;
  border-radius: 50%;
  display: inline-block;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* إعلان */
.ad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 12, 16, 0.95);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  direction: rtl;
}

.ad-box {
  background: #11151C;
  width: 90%;
  max-width: 450px;
  border-radius: 24px;
  padding: 0;
  text-align: center;
  border: 1px solid rgba(212, 175, 55, 0.3);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.7);
  overflow: hidden;
  animation: fadeInUp 0.5s ease;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ad-box h2 {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  padding: 18px;
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  border-bottom: 2px solid #D4AF37;
}

.ad-content {
  max-height: 350px;
  overflow-y: auto;
  padding: 20px;
  font-size: 14px;
  line-height: 1.8;
  color: rgba(255, 255, 255, 0.9);
  text-align: right;
  scrollbar-width: thin;
  scrollbar-color: #D4AF37 #1A1F2A;
}

.ad-content::-webkit-scrollbar {
  width: 6px;
}

.ad-content::-webkit-scrollbar-track {
  background: #1A1F2A;
}

.ad-content::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #D4AF37, #C5A028);
  border-radius: 10px;
}

.ad-btn {
  background: transparent;
  color: #D4AF37;
  border: 2px solid #D4AF37;
  padding: 14px 30px;
  margin: 0 20px 25px 20px;
  border-radius: 50px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 700;
  transition: all 0.3s ease;
  width: calc(100% - 40px);
}

.ad-btn:hover {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border-color: transparent;
  transform: scale(1.02);
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.4);
}

/* تحسين للهواتف */
@media (max-width: 480px) {
  .card {
    width: 95%;
    padding: 25px 15px;
  }
  
  .title {
    font-size: 24px;
  }
  
  .login-type-selector {
    flex-direction: column;
  }
  
  .phone-input-container {
    flex-direction: column;
    gap: 10px;
  }
  
  .country-select,
  .phone-input {
    width: 100%;
  }
  
  .ad-box {
    width: 95%;
  }
  
  .ad-content {
    max-height: 300px;
  }
}
</style>
