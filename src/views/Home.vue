<template>
  <div class="home-container">

    <!-- الشريط العلوي -->
    <div class="top-bar">
      <div class="top-left">
        <i class="fas fa-headset icon"></i>
        <i class="fas fa-envelope icon"></i>
      </div>

      <div class="user-box">
        <div class="welcome">مرحباً، {{ username }}</div>
        <div class="balance">الرصيد: <strong>{{ balance }} USDT</strong></div>
      </div>
    </div>

    <!-- شريط الأزرار -->
    <div class="quick-buttons">
      <button class="quick-btn">
        <span class="btn-icon">⏱</span>
        وفر الوقت
      </button>
      <button class="quick-btn">
        <span class="btn-icon">💰</span>
        وفر المال
      </button>
    </div>

    <!-- شريط الإعلان -->
    <div class="notice-bar">
      <span class="notice-icon">🌴</span>
      شراكة عالمية مع Amazon, eBay, TikTok, Aliexpress, Alibaba, Shopee
    </div>

    <!-- القائمة الرئيسية -->
    <div class="grid-menu">
      <div
        class="item"
        v-for="item in menu"
        :key="item.title"
        @click="go(item.route)"
      >
        <div class="icon-box">
          <i :class="item.icon"></i>
        </div>
        <p>{{ item.title }}</p>
      </div>

      <!-- زر الشركة لعرض الإعلان -->
      <div class="item company-item" @click="showCompanyAd">
        <div class="icon-box gold-glow">
          <i class="fas fa-building"></i>
        </div>
        <p>الشركة</p>
      </div>

      <!-- زر الشروط والأحكام الجديد -->
      <div class="item terms-item" @click="showTermsAd">
        <div class="icon-box gold-glow">
          <i class="fas fa-scroll"></i>
        </div>
        <p>الشروط والأحكام</p>
      </div>
    </div>

    <!-- إعلان Popup فاخر للشركة -->
    <div id="companyAd" class="ad-overlay" v-if="showAd" @click.self="closeAd">
      <div class="ad-box">
        <div class="ad-header">
          <h2>🌴 Palm Treasure</h2>
          <button class="close-btn" @click="closeAd">×</button>
        </div>

        <div class="ad-content">
          <div class="company-message">
            <p>
              مرحباً بالجميع 🌟<br><br>
              يسرّنا أن نعرّفكم بشركة Palm Treasure التي تأسست في سنغافورة بتاريخ 5 فبراير 2026، وهي شركة استثمارية متخصصة في مجال التجارة الإلكترونية. تمتلك الشركة فريقاً تقنياً محترفاً وخبرة مالية قوية، ويقع مقرها الرئيسي حالياً في منطقة الأعمال المركزية في سنغافورة.<br><br>
              وانطلاقاً من رؤيتنا للتوسع وبناء شبكة تعاون واسعة، قمنا بإنشاء فروع ووكالات في عدد من الدول العربية مثل لبنان، الجزائر، ليبيا، والعراق، ونسعى خلال المرحلة القادمة إلى توسيع نشاطنا وانتشارنا في مختلف دول الشرق الأوسط.<br><br>
              لقد جاء تأسيس هذه الشركة بعد دراسة عميقة لما حدث خلال عامي 2024 و2025، حيث ظهرت العديد من المنصات الوهمية التي خدعت الكثير من الناس ولم تستمر طويلاً. ومن هنا كان هدفنا واضحاً: تحويل هذا المجال إلى منصة حقيقية وموثوقة يستفيد منها الأعضاء كما تستفيد منها الشركة، ضمن نظام عادل وشفاف.<br><br>
              وقد تحقق هذا المشروع بجهود كبيرة من الفريق التقني تحت إشراف المهندس أليكس ديروب، الذي لعب دوراً أساسياً في تطوير النظام التقني للشركة. وتقديراً لجهوده المميزة ومساهمته في نجاح هذا المشروع، تمت ترقيته إلى منصب نائب المدير.<br><br>
              نحن في Palm Treasure نؤمن بأن النجاح الحقيقي يجب أن يكون متاحاً للجميع، لذلك تم تصميم هذا المشروع ليكون فرصة متاحة لكل الناس، وخاصة للطبقة المتوسطة وذوي الدخل المحدود، حتى يتمكنوا من تحسين أوضاعهم والمشاركة في فرص الاقتصاد الرقمي.<br><br>
              ولهذا السبب تم وضع نظام واضح وقوانين عادلة تضمن حماية حقوق الموظفين والأعضاء قبل حقوق المستثمرين، مع مراعاة الظروف الاقتصادية والاجتماعية في المجتمعات العربية.<br><br>
              وسيتم تطبيق هذا النظام المبرمج في معظم الدول العربية ابتداءً من 1 مارس 2026 وحتى نهاية عام 2028، وخلال هذه الفترة سيتم تقييم الأداء والنتائج. وبعد ذلك قد يتم تطوير النظام وإجراء بعض التعديلات بناءً على كفاءة الأعضاء وجهودهم وإخلاصهم في العمل.<br><br>
              كما نعمل منذ الآن على التخطيط لمجموعة من الفرص المستقبلية والمشاريع الجديدة التي ستوفر مزيداً من الإمكانيات لأعضاء الشركة في السنوات القادمة.<br><br>
              نتمنى للجميع التوفيق، ونسعد بانضمامكم إلى مجتمع Palm Treasure 🌴
            </p>
          </div>
          
          <div class="vip-section">
            <h3>📊 خطط العضوية والأرباح</h3>
            
            <div class="vip-grid">
              <div class="vip-card" v-for="vip in vipPlans" :key="vip.level">
                <div class="vip-header" :class="getVipClass(vip.level)">
                  <span class="vip-level-badge">{{ vip.level }}</span>
                </div>
                <div class="vip-details">
                  <div class="vip-detail-item">
                    <span class="detail-label">💰 الاشتراك:</span>
                    <span class="detail-value">{{ vip.recharge }} USDT</span>
                  </div>
                  <div class="vip-detail-item">
                    <span class="detail-label">📈 الربح اليومي:</span>
                    <span class="detail-value">{{ vip.daily }} USDT</span>
                  </div>
                  <div class="vip-detail-item">
                    <span class="detail-label">📅 الربح الشهري:</span>
                    <span class="detail-value">{{ vip.monthly }} USDT</span>
                  </div>
                  <div v-if="vip.yearly" class="vip-detail-item">
                    <span class="detail-label">📊 الإجمالي السنوي:</span>
                    <span class="detail-value">{{ vip.yearly }} USDT</span>
                  </div>
                  <div v-if="vip.percentage" class="vip-detail-item">
                    <span class="detail-label">📊 نسبة الربح:</span>
                    <span class="detail-value highlight">{{ vip.percentage }}</span>
                  </div>
                  <div v-if="vip.tasks" class="vip-detail-item">
                    <span class="detail-label">📝 مهمة يومية:</span>
                    <span class="detail-value">{{ vip.tasks }}</span>
                  </div>
                  <div v-if="vip.status" class="vip-detail-item">
                    <span class="detail-label">✅ الحالة:</span>
                    <span class="detail-value status-active">{{ vip.status }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="commission-section">
            <h3>🤝 نظام العمولات</h3>
            <div class="commission-levels">
              <div class="commission-level">المستوى 1: 6%</div>
              <div class="commission-level">المستوى 2: 2%</div>
              <div class="commission-level">المستوى 3: 1%</div>
            </div>
          </div>

          <div class="partners-section">
            <h3>🌐 شراكات عالمية</h3>
            <p>Amazon, eBay, TikTok, Aliexpress, Alibaba, Shopee</p>
          </div>

          <div class="cta-message">
            <p>🚀 اختر المستوى المناسب لك وابدأ اليوم، فكل خطوة صغيرة قد تكون بداية لنجاح كبير مع Palm Treasure 🌴</p>
          </div>
        </div>

        <button class="ad-btn" @click="closeAd">أنا أعرف</button>
      </div>
    </div>

    <!-- إعلان Popup جديد للشروط والأحكام -->
    <div id="termsAd" class="ad-overlay" v-if="showTerms" @click.self="closeTerms">
      <div class="ad-box">
        <div class="ad-header">
          <h2>📜 الشروط والأحكام</h2>
          <button class="close-btn" @click="closeTerms">×</button>
        </div>

        <div class="ad-content">
          <div class="terms-message">
            <h3>📊 عقود رجال الأعمال</h3>
            <p>
              في عالم الأعمال الحديث، لا يُقاس النجاح فقط بما يحققه الشخص لنفسه، بل بقدرته على بناء فريق قوي وفعّال يقوده نحو التقدم والازدهار. ومن هذا المنطلق، تعتمد الشركة نظاماً مميزاً لتقدير القادة وتحفيزهم على تطوير فرقهم وتحقيق نتائج أفضل.<br><br>
              عندما تتمكن من بناء فريق قوي وزيادة عدد الأعضاء النشطين، فإن الشركة تقوم بتقدير جهودك ومنحك مزايا إضافية تعكس مكانتك القيادية داخل الفريق.
            </p>
            
            <h4>📌 عقود رجال الأعمال</h4>
            <p>
              عند وصول عدد أعضاء فريقك إلى 70 عضواً نشطاً أو أكثر، تقوم الشركة بإبرام عقد رسمي معك، وذلك لضمان حقوقك وتقديراً لجهودك في بناء فريق ناجح وإدارته بكفاءة. ويعد هذا العقد دليلاً على ثقة الشركة بك كقائد قادر على تحقيق النجاح والتطوير المستمر.
            </p>
            
            <p>💼 إضافة إلى التقدير المعنوي، تمنحك الشركة راتباً شهرياً ثابتاً يتناسب مع حجم فريقك ونشاطه، وفق سلم الترقيات القيادية التالي:</p>
            
            <div class="promotion-table">
              <div class="promotion-row">
                <div class="promotion-cell">🔹 70 عضواً نشطاً</div>
                <div class="promotion-cell">رتبة مساعد فريق</div>
                <div class="promotion-cell highlight">💰 راتب شهري: 200 دولار</div>
              </div>
              <div class="promotion-row">
                <div class="promotion-cell">🔹 140 عضواً نشطاً</div>
                <div class="promotion-cell">رتبة مشرف فريق</div>
                <div class="promotion-cell highlight">💰 راتب شهري: 350 دولاراً</div>
              </div>
              <div class="promotion-row">
                <div class="promotion-cell">🔹 300 عضو نشط</div>
                <div class="promotion-cell">رتبة مدير فريق</div>
                <div class="promotion-cell highlight">💰 راتب شهري: 500 دولار</div>
              </div>
            </div>
            
            <p>
              ⭐ يعكس هذا النظام التحفيزي رؤية الشركة في تشجيع روح القيادة والعمل الجماعي، حيث يتم مكافأة كل من يعمل بجد على توسيع فريقه وتطويره.
            </p>
            
            <p>
              🚀 إن بناء فريق قوي ليس مجرد مهمة عادية، بل هو طريق حقيقي نحو النجاح في عالم ريادة الأعمال، حيث يتحول العمل الجاد والطموح إلى إنجازات حقيقية تعود بالنفع على القائد وفريقه والشركة معاً.
            </p>
            
            <h4>📢 مواعيد سحب الرواتب الخاصة بحسابات VIP</h4>
            <p>
              حرصًا منا على تنظيم عملية السحب وضمان سلاسة الإجراءات لجميع الأعضاء، تم اعتماد جدول أسبوعي ثابت لمواعيد سحب الرواتب الخاصة بحسابات VIP.<br><br>
              يرجى من جميع الأعضاء الالتزام باليوم المخصص لكل مستوى، حيث يتم تنفيذ عمليات السحب فقط في اليوم المحدد لكل فئة.
            </p>
            
            <h4>📅 جدول السحب الأسبوعي:</h4>
            
            <div class="withdraw-schedule">
              <div class="schedule-day">
                <div class="day-header">🔹 يوم السبت من كل أسبوع:</div>
                <div class="vip-levels">VIP1 💸 VIP2 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الأحد من كل أسبوع:</div>
                <div class="vip-levels">VIP3 💸 VIP4 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الاثنين من كل أسبوع:</div>
                <div class="vip-levels">VIP5 💸 VIP6 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الثلاثاء من كل أسبوع:</div>
                <div class="vip-levels">VIP7 💸 VIP8 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الأربعاء من كل أسبوع:</div>
                <div class="vip-levels">VIP9 💸 VIP10 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الخميس من كل أسبوع:</div>
                <div class="vip-levels">VIP11 💸 VIP12 </div>
              </div>
              <div class="schedule-day">
                <div class="day-header">🔹 يوم الجمعة من كل أسبوع:</div>
                <div class="vip-levels">VIP13 💸 VIP14 </div>
              </div>
            </div>
            
            <div class="important-note">
              <p>⚠️ ملاحظة مهمة:<br>
              يرجى من جميع أعضاء VIP الالتزام باليوم المحدد لمستواهم لضمان تنفيذ عمليات السحب بسرعة وتنظيم، وتجنب أي تأخير في معالجة الطلبات.</p>
            </div>
            
            <div class="cta-message">
              <p>✨ نتمنى لكم تجربة مالية ناجحة ومليئة بالأرباح ونتمنى لكم دوام النجاح معنا.</p>
            </div>
          </div>
        </div>

        <button class="ad-btn" @click="closeTerms">موافق</button>
      </div>
    </div>

  </div>
</template>

<script>
import { auth, db } from "../firebase";
import { doc, getDoc } from "firebase/firestore";
import { onAuthStateChanged } from "firebase/auth";

export default {
  name: "Home",

  data() {
    return {
      username: "جاري التحميل...",
      balance: 0,
      showAd: false,
      showTerms: false,
      
      vipPlans: [
        { level: 'VIP 1', recharge: '0', daily: '0.3', monthly: '9', yearly: '109.5', tasks: '1', status: 'مفعل الآن' },
        { level: 'VIP 2', recharge: '50', daily: '1.6', monthly: '48', percentage: '96% شهرياً', yearly: '584' },
        { level: 'VIP 3', recharge: '100', daily: '3.25', monthly: '97.5', percentage: '97.5% شهرياً', yearly: '1186.25' },
        { level: 'VIP 4', recharge: '300', daily: '10', monthly: '300', percentage: '100% شهرياً', yearly: '3650' },
        { level: 'VIP 5', recharge: '900', daily: '33', monthly: '990', percentage: '110% شهرياً' },
        { level: 'VIP 6', recharge: '1350', daily: '51', monthly: '1530', percentage: '113% شهرياً' },
        { level: 'VIP 7', recharge: '1800', daily: '70', monthly: '2100', percentage: '116% شهرياً' },
        { level: 'VIP 8', recharge: '3600', daily: '150', monthly: '4500', percentage: '125% شهرياً' },
        { level: 'VIP 9', recharge: '7200', daily: '330', monthly: '9900', percentage: '137% شهرياً' },
        { level: 'VIP 10', recharge: '14400', daily: '700', monthly: '21000', percentage: '146% شهرياً' },
        { level: 'VIP 11', recharge: '18800', daily: '1600', monthly: '48000', percentage: '255% شهرياً' },
        { level: 'VIP 12', recharge: '37600', daily: '3500', monthly: '105000', percentage: '279% شهرياً' },
        { level: 'VIP 13', recharge: '75200', daily: '7500', monthly: '225000', percentage: '299% شهرياً' },
        { level: 'VIP 14', recharge: '150400', daily: '16000', monthly: '480000', percentage: '319% شهرياً' }
      ],

      menu: [
        { title: "تعبئة رصيد", icon: "fas fa-coins", route: "/recharge" },
        { title: "سحب", icon: "fas fa-wallet", route: "/withdraw" },
        { title: "المعاملات", icon: "fas fa-history", route: "/transactions" },
        { title: "برنامج", icon: "fas fa-download", route: "/program" },
        { title: "الأصدقاء", icon: "fas fa-users", route: "/team" },
        { title: "الوكالات", icon: "fas fa-id-card", route: "/agency" }
      ]
    };
  },

  created() {
    this.watchUser();
  },

  methods: {
    watchUser() {
      onAuthStateChanged(auth, async (user) => {
        if (!user) {
          this.username = "غير مسجل";
          this.balance = 0;
          this.$router.push("/login");
          return;
        }

        await this.loadUserData(user.uid);
      });
    },

    async loadUserData(uid) {
      try {
        const snap = await getDoc(doc(db, "users", uid));

        if (snap.exists()) {
          const data = snap.data();
          
          if (data.phoneNumber) {
            this.username = data.phoneNumber;
          } else if (data.email) {
            this.username = data.email;
          } else {
            this.username = "مستخدم";
          }
          
          this.balance = data.balance ?? 0;
        } else {
          this.username = "مستخدم";
          this.balance = 0;
        }
      } catch (err) {
        console.error("Error loading user data:", err);
        this.username = "مستخدم";
        this.balance = 0;
      }
    },

    getVipClass(level) {
      if (level.includes('VIP 1')) return 'vip-bronze';
      if (level.includes('VIP 2') || level.includes('VIP 3')) return 'vip-silver';
      if (level.includes('VIP 4') || level.includes('VIP 5')) return 'vip-gold';
      if (level.includes('VIP 6') || level.includes('VIP 7') || level.includes('VIP 8') || level.includes('VIP 9')) return 'vip-platinum';
      if (level.includes('VIP 10') || level.includes('VIP 11') || level.includes('VIP 12') || level.includes('VIP 13') || level.includes('VIP 14')) return 'vip-elite';
      return '';
    },

    go(route) {
      this.$router.push(route);
    },

    showCompanyAd() {
      this.showAd = true;
      document.body.style.overflow = 'hidden';
    },

    closeAd() {
      this.showAd = false;
      document.body.style.overflow = 'auto';
    },

    showTermsAd() {
      this.showTerms = true;
      document.body.style.overflow = 'hidden';
    },

    closeTerms() {
      this.showTerms = false;
      document.body.style.overflow = 'auto';
    }
  },

  beforeUnmount() {
    document.body.style.overflow = 'auto';
  }
};
</script>

<style scoped>
/* الخلفية الرئيسية - أسود فاخر */
.home-container {
  direction: rtl;
  padding: 16px;
  background: #0A0C10;
  min-height: 100vh;
  color: #ffffff;
  font-family: 'Cairo', sans-serif;
  padding-bottom: 80px;
}

/* الشريط العلوي */
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #11151C;
  padding: 15px 20px;
  border-radius: 16px;
  margin-bottom: 20px;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.top-left {
  display: flex;
  gap: 15px;
}

.icon {
  font-size: 24px;
  color: #D4AF37;
  cursor: pointer;
  transition: all 0.3s ease;
}

.icon:hover {
  color: #F6E27A;
  transform: scale(1.1);
}

.user-box {
  text-align: left;
}

.welcome {
  font-size: 16px;
  font-weight: 600;
  color: #D4AF37;
  margin-bottom: 5px;
}

.balance {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.8);
}

.balance strong {
  color: #D4AF37;
  font-size: 18px;
}

/* شريط الأزرار السريعة */
.quick-buttons {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.quick-btn {
  flex: 1;
  padding: 14px;
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border: 1px solid rgba(212, 175, 55, 0.3);
  border-radius: 14px;
  font-weight: 700;
  color: #D4AF37;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 15px;
}

.btn-icon {
  font-size: 20px;
}

.quick-btn:hover {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-color: #D4AF37;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.2);
}

/* شريط الإعلان */
.notice-bar {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  padding: 14px 20px;
  border-radius: 50px;
  text-align: center;
  margin-bottom: 25px;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.notice-icon {
  font-size: 20px;
}

/* القائمة الرئيسية */
.grid-menu {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 18px;
  margin-bottom: 20px;
}

.item {
  background: #11151C;
  color: #ffffff;
  padding: 25px 15px;
  border-radius: 20px;
  text-align: center;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.item:hover {
  transform: translateY(-5px);
  border-color: #D4AF37;
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.2);
}

.icon-box {
  font-size: 32px;
  margin-bottom: 12px;
  color: #D4AF37;
  transition: all 0.3s ease;
}

.item:hover .icon-box {
  transform: scale(1.1);
  color: #F6E27A;
}

.gold-glow {
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
}

.company-item {
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border: 1px solid #D4AF37;
  animation: goldPulse 2s infinite;
}

.terms-item {
  background: linear-gradient(135deg, #11151C, #1A1F2A);
  border: 1px solid #D4AF37;
  animation: goldPulse 2s infinite;
}

/* ===== إعلان فاخر ===== */
.ad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 12, 16, 0.98);
  backdrop-filter: blur(10px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  direction: rtl;
}

.ad-box {
  background: #11151C;
  width: 95%;
  max-width: 900px;
  max-height: 90vh;
  border-radius: 30px;
  overflow: hidden;
  border: 2px solid #D4AF37;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.7), 0 0 30px rgba(212, 175, 55, 0.3);
  animation: fadeInUp 0.5s ease;
  display: flex;
  flex-direction: column;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ad-header {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
}

.ad-header h2 {
  color: #0A0C10;
  margin: 0;
  font-size: 24px;
  font-weight: 700;
  flex: 1;
  text-align: center;
}

.close-btn {
  background: rgba(10, 12, 16, 0.2);
  border: none;
  color: #0A0C10;
  font-size: 28px;
  cursor: pointer;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s ease;
  position: absolute;
  left: 10px;
  top: 10px;
}

.close-btn:hover {
  background: rgba(10, 12, 16, 0.4);
  transform: rotate(90deg);
}

.ad-content {
  flex: 1;
  overflow-y: auto;
  padding: 25px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  line-height: 1.8;
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

.company-message {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 25px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.company-message p {
  margin: 0;
  color: #ffffff;
  font-size: 14px;
  line-height: 1.8;
}

.terms-message {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 25px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.terms-message h3 {
  color: #D4AF37;
  text-align: center;
  margin-bottom: 20px;
  font-size: 22px;
}

.terms-message h4 {
  color: #F6E27A;
  margin: 20px 0 10px 0;
  font-size: 18px;
}

.terms-message p {
  margin: 0 0 15px 0;
  color: #ffffff;
  font-size: 14px;
  line-height: 1.8;
}

.promotion-table {
  background: #11151C;
  border-radius: 12px;
  padding: 15px;
  margin: 20px 0;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

.promotion-row {
  display: flex;
  justify-content: space-between;
  padding: 12px;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
}

.promotion-row:last-child {
  border-bottom: none;
}

.promotion-cell {
  color: #ffffff;
}

.promotion-cell.highlight {
  color: #F6E27A;
  font-weight: 700;
}

.withdraw-schedule {
  background: #11151C;
  border-radius: 12px;
  padding: 15px;
  margin: 20px 0;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

.schedule-day {
  padding: 10px;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
}

.schedule-day:last-child {
  border-bottom: none;
}

.day-header {
  color: #F6E27A;
  font-weight: 600;
  margin-bottom: 5px;
}

.vip-levels {
  color: #D4AF37;
  padding-right: 20px;
}

.important-note {
  background: rgba(212, 175, 55, 0.1);
  border-right: 3px solid #D4AF37;
  padding: 15px;
  margin: 20px 0;
  border-radius: 8px;
}

.important-note p {
  margin: 0;
  color: #F6E27A;
}

.vip-section {
  margin-bottom: 25px;
}

.vip-section h3 {
  color: #D4AF37;
  text-align: center;
  margin-bottom: 20px;
  font-size: 22px;
}

.vip-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 15px;
}

.vip-card {
  background: #1A1F2A;
  border-radius: 16px;
  overflow: hidden;
  border: 1px solid rgba(212, 175, 55, 0.2);
  transition: all 0.3s ease;
}

.vip-card:hover {
  transform: translateY(-5px);
  border-color: #D4AF37;
  box-shadow: 0 10px 20px rgba(212, 175, 55, 0.2);
}

.vip-header {
  padding: 12px;
  text-align: center;
  font-weight: 700;
}

.vip-level-badge {
  font-size: 18px;
}

.vip-bronze {
  background: linear-gradient(135deg, #CD7F32, #B87333);
}
.vip-silver {
  background: linear-gradient(135deg, #C0C0C0, #A8A8A8);
}
.vip-gold {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
}
.vip-platinum {
  background: linear-gradient(135deg, #E5E4E2, #B0C4DE);
}
.vip-elite {
  background: linear-gradient(135deg, #8A2BE2, #4B0082);
}

.vip-details {
  padding: 15px;
}

.vip-detail-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
}

.vip-detail-item:last-child {
  border-bottom: none;
}

.detail-label {
  color: rgba(255, 255, 255, 0.7);
}

.detail-value {
  color: #D4AF37;
  font-weight: 600;
}

.detail-value.highlight {
  color: #F6E27A;
  font-size: 16px;
}

.status-active {
  color: #4CAF50;
}

.commission-section {
  text-align: center;
  margin: 25px 0;
  padding: 20px;
  background: #1A1F2A;
  border-radius: 16px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.commission-section h3 {
  color: #D4AF37;
  margin-bottom: 15px;
  font-size: 20px;
}

.commission-levels {
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}

.commission-level {
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
  font-size: 18px;
}

.partners-section {
  text-align: center;
  margin: 25px 0;
  padding: 20px;
  background: #1A1F2A;
  border-radius: 16px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.partners-section h3 {
  color: #D4AF37;
  margin-bottom: 10px;
  font-size: 20px;
}

.partners-section p {
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
}

.cta-message {
  text-align: center;
  margin: 25px 0;
  padding: 20px;
  background: linear-gradient(135deg, rgba(212, 175, 55, 0.1), rgba(246, 226, 122, 0.1));
  border-radius: 16px;
  border: 1px solid #D4AF37;
}

.cta-message p {
  color: #F6E27A;
  font-size: 16px;
  font-weight: 700;
  margin: 0;
}

.ad-btn {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  padding: 16px 30px;
  margin: 0 25px 25px 25px;
  border-radius: 50px;
  cursor: pointer;
  font-size: 18px;
  font-weight: 700;
  transition: all 0.3s ease;
  width: calc(100% - 50px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
}

.ad-btn:hover {
  transform: scale(1.02);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.5);
  background: linear-gradient(135deg, #C5A028, #F6E27A, #D4AF37);
}

/* تحسين للهواتف */
@media (max-width: 768px) {
  .ad-box {
    width: 95%;
    max-height: 95vh;
  }
  
  .vip-grid {
    grid-template-columns: 1fr;
  }
  
  .commission-levels {
    flex-direction: column;
    gap: 10px;
  }
  
  .ad-content {
    padding: 15px;
  }
  
  .promotion-row {
    flex-direction: column;
    gap: 5px;
  }
}

@media (max-width: 480px) {
  .home-container {
    padding: 12px;
    padding-bottom: 70px;
  }
  
  .grid-menu {
    gap: 12px;
  }
  
  .item {
    padding: 20px 10px;
  }
  
  .icon-box {
    font-size: 28px;
  }
  
  .ad-header h2 {
    font-size: 20px;
  }
  
  .vip-section h3 {
    font-size: 18px;
  }
}

/* تأثيرات إضافية */
@keyframes goldPulse {
  0% { box-shadow: 0 0 5px rgba(212, 175, 55, 0.3); }
  50% { box-shadow: 0 0 20px rgba(212, 175, 55, 0.6); }
  100% { box-shadow: 0 0 5px rgba(212, 175, 55, 0.3); }
}
</style>
