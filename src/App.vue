<template>
  <div id="app" :class="{ 'rtl': currentLang === 'AR' }">
    <!-- زر تغيير اللغة -->
    <div class="circle-btn lang-btn" @click.stop="toggleLanguageMenu" @mousedown="startDrag" @touchstart="startDrag">
      <i class="fas fa-globe"></i>
      <span class="lang-code">{{ currentLang }}</span>
    </div>

    <!-- زر الدعم - تم تحديث الرابط -->
    <a class="circle-btn support-btn"
       href="https://t.me/Palm_Treasure"
       target="_blank"
       @mousedown="startDrag"
       @touchstart="startDrag">
      <i class="fas fa-headset"></i>
    </a>

    <!-- زر انستغرام - تم تحديث الرابط -->
    <a class="circle-btn instagram-btn"
       href="https://www.instagram.com/palm__treasure?igsh=OXR1emp3N2k2d2Yz"
       target="_blank"
       @mousedown="startDrag"
       @touchstart="startDrag">
      <i class="fab fa-instagram"></i>
    </a>

    <!-- زر عرض خاص جديد مع إشعار أحمر -->
    <div class="circle-btn offer-btn" @click.stop="toggleOfferMessage" @mousedown="startDrag" @touchstart="startDrag">
      <i class="fas fa-gift"></i>
      <div class="notification-badge" v-if="hasNewOffer">3</div>
    </div>

    <!-- نافذة العرض الخاص -->
    <div v-if="showOfferMessage" class="bubble-chat-overlay" @click="closeOfferMessage">
      <div class="bubble-chat-window" @click.stop>
        <div class="bubble-chat-header">
          <div class="bubble-chat-title">
            <div class="bubble-avatar">🎁</div>
            <div>
              <div class="bubble-sender">Palm Treasure 🌴</div>
              <div class="bubble-time">عرض خاص</div>
            </div>
          </div>
          <button class="bubble-close-btn" @click.stop="closeOfferMessage">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M18 6L6 18M6 6L18 18" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
            </svg>
          </button>
        </div>
        
        <div class="bubble-chat-body">
          <div class="bubble-message bubble-received">
            <div class="bubble-tail"></div>
            <div class="bubble-content">
              <strong>🚀 انضم إلى فريقنا وابدأ ببناء نجاحك اليوم</strong>
              <br><br>
              نحن نقدم لك فرصة حقيقية لتكون جزءًا من فريق طموح ومتطور. يمكنك الآن دعوة أعضاء جدد إلى فريقك عبر رابط الإحالة الخاص بك والعمل معهم لتحقيق النجاح المشترك. كلما توسع فريقك وزاد عدد الأعضاء فيه، زادت فرصك للحصول على مكافآت مالية مجزية.
              <br><br>
              عند انضمامك إلينا، ستحصل على فرصة لتصبح مشرفًا على فريقك الخاص، حيث يمكنك إدارة فريقك وتنميته من خلال دعوة أعضاء جدد عبر رابط الإحالة الخاص بك ومساعدتهم على التقدم. ومع نمو الفريق وزيادة عدد الأعضاء النشطين، ستحصل على مكافآت شهرية وفقًا لعدد الأشخاص المنضمين إلى فريقك.
              <br><br>
              💼 نظام المكافآت لدينا واضح وعادل، ويعتمد على عدد الأعضاء في فريقك الذين ينضمون عبر رابط الإحالة الخاص بك، كما هو موضح في الجدول التالي:
              <br><br>
              <div style="background: #1A1F2A; border-radius: 10px; padding: 10px; margin: 10px 0;">
                <div style="display: flex; justify-content: space-between; border-bottom: 1px solid #D4AF37; padding: 5px 0;">
                  <span style="color: #D4AF37; font-weight: bold;">💰 قيمة الراتب</span>
                  <span style="color: #D4AF37; font-weight: bold;">👥 عدد الأشخاص</span>
                </div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">50$ | 10</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">100$ | 20</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">150$ | 30</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">200$ | 40</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">250$ | 50</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">300$ | 60</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">350$ | 70</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">400$ | 80</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">450$ | 90</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">500$ | 100</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">550$ | 120</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">600$ | 140</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">650$ | 160</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">700$ | 180</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">750$ | 200</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">800$ | 220</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">850$ | 240</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">900$ | 260</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">950$ | 280</div>
                <div style="display: flex; justify-content: space-between; padding: 5px 0;">1000$ | 300</div>
              </div>
              📈 كلما ازداد عدد الأعضاء في فريقك، زادت قيمة المكافأة التي تحصل عليها. هذه فرصة ممتازة لبناء شبكة قوية من الأشخاص الطموحين والعمل معًا لتحقيق دخل إضافي ونمو مستمر.
              <br><br>
              ✨ ابدأ اليوم بدعوة الأعضاء عبر رابط الإحالة الخاص بك، وكن قائد فريق ناجح يحقق نتائج مميزة ومكافآت متزايدة.
              <br><br>
              ⚠️ ملاحظة مهمة:<br>
              يجب على كل عضو ينضم إلى فريقك شراء مستوى VIP 2 حتى يتم احتسابه ضمن أعضاء فريقك في نظام المكافآت.
            </div>
            <div class="bubble-time">12:00</div>
          </div>
        </div>
        
        <div class="bubble-chat-footer">
          <button class="bubble-action-btn" @click.stop="closeOfferMessage">
            فهمت وشكرًا! 🎯
          </button>
        </div>
      </div>
    </div>

    <!-- قائمة اللغات -->
    <transition name="fade">
      <div v-if="showLangMenu" class="lang-menu" @click.stop>
        <div class="lang-menu-header">
          <i class="fas fa-language"></i>
          <span>اختر اللغة</span>
        </div>
        <div 
          class="lang-item" 
          v-for="l in languages" 
          :key="l.code"
          @click.stop="setLanguage(l)"
          :class="{ active: currentLang === l.code }"
        >
          <span class="lang-name">{{ l.name }}</span>
          <span class="lang-code-small">{{ l.code }}</span>
        </div>
      </div>
    </transition>

    <!-- الصفحات مع حاوية داخلية -->
    <div class="page-container">
      <router-view />
    </div>

    <!-- ⭐ شريط التنقل - تم تعديله ليظهر فوق كل شيء مع مؤشرات النقر ⭐ -->
    <div class="bottom-nav" v-if="authLoaded && showBottomNav">
      <div 
        class="nav-item" 
        @click.stop="navigateTo('/home')"
        :class="{ active: isActive('/home') }"
      >
        <i class="fas fa-home"></i>
        <span>الرئيسية</span>
      </div>

      <div 
        class="nav-item" 
        @click.stop="navigateTo('/vip')"
        :class="{ active: isActive('/vip') }"
      >
        <i class="fas fa-crown"></i>
        <span>VIP</span>
      </div>

      <div 
        class="nav-item" 
        @click.stop="navigateTo('/tasks')"
        :class="{ active: isActive('/tasks') }"
      >
        <i class="fas fa-tasks"></i>
        <span>المهام</span>
      </div>

      <div 
        class="nav-item" 
        @click.stop="navigateTo('/team')"
        :class="{ active: isActive('/team') }"
      >
        <i class="fas fa-users"></i>
        <span>الفريق</span>
      </div>

      <div 
        class="nav-item" 
        @click.stop="navigateTo('/profile')"
        :class="{ active: isActive('/profile') }"
      >
        <i class="fas fa-user"></i>
        <span>حسابي</span>
      </div>
    </div>

    <!-- إعلان Popup فاخر -->
    <transition name="fade">
      <div id="companyAd" class="ad-overlay" v-if="showAd" @click.self="closeAd">
        <div class="ad-box" @click.stop>
          <div class="ad-header">
            <h2>
              <i class="fas fa-crown"></i>
              Palm Treasure 🌴
              <i class="fas fa-crown"></i>
            </h2>
            <button class="ad-close-btn" @click.stop="closeAd">
              <i class="fas fa-times"></i>
            </button>
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

            <div class="ad-highlight">
              <i class="fas fa-handshake"></i>
              <span>شراكات عالمية مع Amazon, eBay, TikTok, Aliexpress, Alibaba, Shopee</span>
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

            <div class="commission-box">
              <h4>🤝 نظام العمولات</h4>
              <div class="commission-row">
                <span>المستوى 1: 6%</span>
                <span>المستوى 2: 2%</span>
                <span>المستوى 3: 1%</span>
              </div>
            </div>

            <div class="cta-message">
              <p>🚀 اختر المستوى المناسب لك وابدأ اليوم، فكل خطوة صغيرة قد تكون بداية لنجاح كبير مع Palm Treasure 🌴</p>
            </div>
          </div>

          <button class="ad-btn" @click.stop="closeAd">
            <i class="fas fa-check-circle"></i>
            أنا أعرف
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { getAuth, onAuthStateChanged } from "firebase/auth";

export default {
  name: "App",

  data() {
    return {
      authLoaded: false,
      user: null,

      showLangMenu: false,
      currentLang: "AR",

      showAd: false,
      showOfferMessage: false,
      hasNewOffer: true,

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

      languages: [
        { name: "Polski", code: "PL" },
        { name: "English", code: "EN" },
        { name: "Français", code: "FR" },
        { name: "Italiano", code: "IT" },
        { name: "日本語", code: "JP" },
        { name: "한국인", code: "KR" },
        { name: "Deutsch", code: "DE" },
        { name: "Русский", code: "RU" },
        { name: "Tiếng Việt", code: "VI" },
        { name: "Português", code: "PT" },
        { name: "Türkçe", code: "TR" },
        { name: "Español", code: "ES" },
        { name: "فارسی", code: "FA" },
        { name: "العربي", code: "AR" }
      ],

      dragging: false,
      currentButton: null,
      startX: 0,
      startY: 0,
      initialLeft: 0,
      initialBottom: 0,
    };
  },

  created() {
    const auth = getAuth();

    const saved = localStorage.getItem("app_language");
    if (saved) this.currentLang = saved;

    onAuthStateChanged(auth, (u) => {
      this.user = u;
      this.authLoaded = true;
      
      if (this.user) {
        setTimeout(() => {
          this.showAd = true;
        }, 1000);
      }
    });

    this.loadButtonPositions();
  },

  mounted() {
    document.addEventListener('mousemove', this.onDrag);
    document.addEventListener('mouseup', this.stopDrag);
    document.addEventListener('touchmove', this.onDrag, { passive: false });
    document.addEventListener('touchend', this.stopDrag);
    document.addEventListener('touchcancel', this.stopDrag);
  },

  beforeDestroy() {
    document.removeEventListener('mousemove', this.onDrag);
    document.removeEventListener('mouseup', this.stopDrag);
    document.removeEventListener('touchmove', this.onDrag);
    document.removeEventListener('touchend', this.stopDrag);
    document.removeEventListener('touchcancel', this.stopDrag);
  },

  computed: {
    showBottomNav() {
      if (!this.user) return false;

      const path = this.$route.path;
      const hidden = ["/login", "/register", "/admin", "/403"];

      return !hidden.some((r) => path.startsWith(r));
    }
  },

  methods: {
    toggleLanguageMenu() {
      this.showLangMenu = !this.showLangMenu;
    },

    setLanguage(lang) {
      this.currentLang = lang.code;
      localStorage.setItem("app_language", lang.code);
      this.showLangMenu = false;
      
      document.documentElement.dir = lang.code === 'AR' ? 'rtl' : 'ltr';
    },

    getVipClass(level) {
      if (level.includes('VIP 1')) return 'vip-bronze';
      if (level.includes('VIP 2') || level.includes('VIP 3')) return 'vip-silver';
      if (level.includes('VIP 4') || level.includes('VIP 5')) return 'vip-gold';
      if (level.includes('VIP 6') || level.includes('VIP 7') || level.includes('VIP 8') || level.includes('VIP 9')) return 'vip-platinum';
      if (level.includes('VIP 10') || level.includes('VIP 11') || level.includes('VIP 12') || level.includes('VIP 13') || level.includes('VIP 14')) return 'vip-elite';
      return '';
    },

    closeAd() {
      this.showAd = false;
    },

    toggleOfferMessage() {
      this.showOfferMessage = !this.showOfferMessage;
      if (this.showOfferMessage) {
        this.hasNewOffer = false;
      }
    },

    closeOfferMessage() {
      this.showOfferMessage = false;
    },

    navigateTo(path) {
      console.log("Navigating to:", path);
      if (this.$route.path !== path) {
        this.$router.push(path);
      }
    },

    isActive(path) {
      return this.$route.path === path;
    },

    startDrag(event) {
      // منع السلوك الافتراضي فقط للأزرار القابلة للسحب
      if (!event.target.closest('a')) {
        event.preventDefault();
      }
      
      const button = event.currentTarget;
      this.currentButton = button;
      this.dragging = true;
      
      if (event.type === 'mousedown') {
        this.startX = event.clientX;
        this.startY = event.clientY;
      } else if (event.type === 'touchstart') {
        this.startX = event.touches[0].clientX;
        this.startY = event.touches[0].clientY;
      }
      
      const computedStyle = window.getComputedStyle(button);
      this.initialLeft = parseInt(computedStyle.right) || 15;
      this.initialBottom = parseInt(computedStyle.bottom) || 100;
      
      button.classList.add('dragging');
    },

    onDrag(event) {
      if (!this.dragging || !this.currentButton) return;
      
      event.preventDefault();
      
      let currentX, currentY;
      if (event.type === 'mousemove') {
        currentX = event.clientX;
        currentY = event.clientY;
      } else if (event.type === 'touchmove') {
        currentX = event.touches[0].clientX;
        currentY = event.touches[0].clientY;
      } else {
        return;
      }
      
      const deltaX = currentX - this.startX;
      const deltaY = currentY - this.startY;
      
      const newRight = Math.max(5, Math.min(window.innerWidth - 50, this.initialLeft - deltaX));
      const newBottom = Math.max(10, Math.min(window.innerHeight - 150, this.initialBottom - deltaY));
      
      this.currentButton.style.right = newRight + 'px';
      this.currentButton.style.bottom = newBottom + 'px';
    },

    stopDrag() {
      if (this.dragging && this.currentButton) {
        this.saveButtonPosition(this.currentButton);
        this.currentButton.classList.remove('dragging');
      }
      
      this.dragging = false;
      this.currentButton = null;
    },

    saveButtonPosition(button) {
      const className = button.className.split(' ').find(cls => cls.includes('-btn'));
      if (!className) return;
      
      const right = button.style.right;
      const bottom = button.style.bottom;
      
      if (right && bottom) {
        const positions = JSON.parse(localStorage.getItem('buttonPositions') || '{}');
        positions[className] = { right, bottom };
        localStorage.setItem('buttonPositions', JSON.stringify(positions));
      }
    },

    loadButtonPositions() {
      this.$nextTick(() => {
        const positions = JSON.parse(localStorage.getItem('buttonPositions') || '{}');
        
        setTimeout(() => {
          document.querySelectorAll('.circle-btn').forEach(btn => {
            const className = btn.className.split(' ').find(cls => cls.includes('-btn'));
            if (className && positions[className]) {
              btn.style.right = positions[className].right;
              btn.style.bottom = positions[className].bottom;
            }
          });
        }, 100);
      });
    }
  }
};
</script>

<style>
/* استيراد خطوط وأيقونات */
@import url('https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;700;800&display=swap');
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Cairo', sans-serif;
  background: #0A0C10;
  color: #ffffff;
  overflow-x: hidden;
}

#app {
  min-height: 100vh;
  position: relative;
}

.page-container {
  width: 100%;
  max-width: 100%;
  margin: 0 auto;
  position: relative;
  z-index: 1;
  min-height: 100vh;
  padding-bottom: 80px;
}

#app.rtl {
  direction: rtl;
}

.circle-btn {
  position: fixed;
  width: 45px;
  height: 45px;
  background: #11151C;
  border: 2px solid #D4AF37;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #D4AF37;
  font-size: 20px;
  cursor: grab;
  z-index: 9999;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
  transition: all 0.3s ease;
  text-decoration: none;
  user-select: none;
  touch-action: none;
  pointer-events: auto;
}

.circle-btn.dragging {
  cursor: grabbing;
  opacity: 0.9;
  transform: scale(1.05);
  transition: none;
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.6);
}

.circle-btn:hover {
  transform: translateY(-5px) scale(1.1);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
  background: #D4AF37;
  color: #0A0C10;
}

.circle-btn:active {
  cursor: grabbing;
}

.circle-btn a {
  text-decoration: none;
  color: inherit;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.lang-code {
  font-size: 10px;
  font-weight: 700;
  margin-top: -5px;
  background: #D4AF37;
  color: #0A0C10;
  padding: 2px 4px;
  border-radius: 4px;
}

.lang-btn:hover .lang-code {
  background: #0A0C10;
  color: #D4AF37;
}

.notification-badge {
  position: absolute;
  top: -5px;
  right: -5px;
  background: #ff3b30;
  color: white;
  font-size: 11px;
  font-weight: bold;
  min-width: 18px;
  height: 18px;
  border-radius: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 4px;
  border: 2px solid #0A0C10;
  box-shadow: 0 2px 5px rgba(255, 59, 48, 0.5);
  animation: pulse-red 2s infinite;
  z-index: 10000;
}

@keyframes pulse-red {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.lang-btn {
  right: 15px;
  bottom: 100px;
  background: linear-gradient(135deg, #11151C, #1A1F2A);
}

.support-btn {
  right: 70px;
  bottom: 100px;
}

.instagram-btn {
  right: 125px;
  bottom: 100px;
}

.offer-btn {
  right: 180px;
  bottom: 100px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
}

.instagram-btn i {
  font-size: 22px;
  color: #D4AF37;
}

.instagram-btn:hover i {
  color: #0A0C10;
}

.offer-btn i {
  color: #0A0C10;
}

.offer-btn:hover i {
  color: #D4AF37;
}

.bubble-chat-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 12, 16, 0.8);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
}

.bubble-chat-window {
  width: 90%;
  max-width: 450px;
  background: #11151C;
  border-radius: 30px;
  overflow: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 0 2px #D4AF37;
  animation: bubbleSlide 0.3s ease;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

@keyframes bubbleSlide {
  from {
    transform: translateY(30px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.bubble-chat-header {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #0A0C10;
}

.bubble-chat-title {
  display: flex;
  align-items: center;
  gap: 15px;
}

.bubble-avatar {
  width: 50px;
  height: 50px;
  background: #0A0C10;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  color: #D4AF37;
  border: 2px solid #D4AF37;
}

.bubble-sender {
  font-weight: 800;
  font-size: 18px;
}

.bubble-time {
  font-size: 12px;
  opacity: 0.8;
}

.bubble-close-btn {
  background: rgba(10, 12, 16, 0.2);
  border: none;
  color: #0A0C10;
  cursor: pointer;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.bubble-close-btn:hover {
  background: rgba(10, 12, 16, 0.4);
  transform: rotate(90deg);
}

.bubble-chat-body {
  padding: 25px;
  max-height: 500px;
  overflow-y: auto;
  background: #1A1F2A;
}

.bubble-message {
  position: relative;
  margin-bottom: 20px;
}

.bubble-received {
  text-align: right;
}

.bubble-tail {
  position: absolute;
  bottom: -8px;
  right: 10px;
  width: 0;
  height: 0;
  border-left: 8px solid transparent;
  border-right: 8px solid transparent;
  border-top: 8px solid #2A2F3A;
}

.bubble-content {
  background: #2A2F3A;
  padding: 15px 20px;
  border-radius: 18px;
  border-top-right-radius: 4px;
  display: inline-block;
  max-width: 100%;
  text-align: right;
  line-height: 1.6;
  font-size: 14px;
  color: #ffffff;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.bubble-message .bubble-time {
  font-size: 11px;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 5px;
  text-align: right;
}

.bubble-chat-footer {
  padding: 20px;
  background: #11151C;
  border-top: 1px solid rgba(212, 175, 55, 0.2);
}

.bubble-action-btn {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  border-radius: 50px;
  font-weight: 800;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 5px 15px rgba(212, 175, 55, 0.3);
}

.bubble-action-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(212, 175, 55, 0.4);
}

.lang-menu {
  position: fixed;
  bottom: 160px;
  right: 15px;
  width: 220px;
  background: #11151C;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5), 0 0 0 2px #D4AF37;
  z-index: 9999;
  overflow: hidden;
  border: 1px solid rgba(212, 175, 55, 0.3);
}

.lang-menu-header {
  padding: 15px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A);
  color: #0A0C10;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 10px;
  border-bottom: 1px solid rgba(212, 175, 55, 0.3);
}

.lang-menu-header i {
  font-size: 18px;
}

.lang-item {
  padding: 12px 15px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #ffffff;
  transition: all 0.2s;
  border-bottom: 1px solid rgba(212, 175, 55, 0.1);
}

.lang-item:last-child {
  border-bottom: none;
}

.lang-item:hover {
  background: #1A1F2A;
  color: #D4AF37;
}

.lang-item.active {
  background: rgba(212, 175, 55, 0.2);
  color: #D4AF37;
}

.lang-name {
  font-size: 14px;
}

.lang-code-small {
  font-size: 12px;
  padding: 3px 8px;
  background: rgba(212, 175, 55, 0.2);
  border-radius: 20px;
  color: #D4AF37;
}

.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 75px;
  background: #11151C;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-top: 2px solid #D4AF37;
  box-shadow: 0 -5px 20px rgba(0, 0, 0, 0.5);
  z-index: 9998;
  direction: rtl;
  pointer-events: auto;
}

.nav-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: rgba(255, 255, 255, 0.6);
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 8px 12px;
  border-radius: 12px;
  flex: 1;
  height: 100%;
  pointer-events: auto;
}

.nav-item i {
  font-size: 22px;
  margin-bottom: 4px;
  transition: all 0.3s ease;
}

.nav-item span {
  font-size: 11px;
  font-weight: 500;
}

.nav-item:hover {
  color: #D4AF37;
  background: rgba(212, 175, 55, 0.1);
}

.nav-item.active {
  color: #D4AF37;
}

.nav-item.active i {
  transform: translateY(-2px);
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
}

.ad-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(10, 12, 16, 0.95);
  backdrop-filter: blur(10px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10001;
}

.ad-box {
  background: #11151C;
  width: 95%;
  max-width: 900px;
  max-height: 90vh;
  border-radius: 30px;
  overflow: hidden;
  border: 2px solid #D4AF37;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5), 0 0 30px rgba(212, 175, 55, 0.3);
  display: flex;
  flex-direction: column;
}

.ad-header {
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #0A0C10;
}

.ad-header h2 {
  margin: 0;
  font-size: 24px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.ad-header i {
  font-size: 24px;
}

.ad-close-btn {
  background: rgba(10, 12, 16, 0.2);
  border: none;
  color: #0A0C10;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 18px;
}

.ad-close-btn:hover {
  background: rgba(10, 12, 16, 0.4);
  transform: rotate(90deg);
}

.ad-content {
  flex: 1;
  overflow-y: auto;
  padding: 25px;
  color: #ffffff;
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
  background: #D4AF37;
  border-radius: 10px;
}

.company-message {
  background: #1A1F2A;
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 20px;
  border: 1px solid rgba(212, 175, 55, 0.2);
}

.company-message p {
  margin: 0;
  color: #ffffff;
  font-size: 14px;
  line-height: 1.8;
}

.ad-highlight {
  background: #1A1F2A;
  padding: 15px;
  border-radius: 16px;
  margin: 15px 0;
  display: flex;
  align-items: center;
  gap: 10px;
  border: 1px solid rgba(212, 175, 55, 0.3);
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

.commission-box {
  background: linear-gradient(135deg, #1A1F2A, #11151C);
  border-radius: 16px;
  padding: 15px;
  text-align: center;
  margin: 20px 0;
}

.commission-box h4 {
  color: #D4AF37;
  margin-bottom: 10px;
}

.commission-row {
  display: flex;
  justify-content: center;
  gap: 20px;
  color: #ffffff;
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
  width: calc(100% - 40px);
  margin: 0 20px 25px 20px;
  padding: 15px;
  background: linear-gradient(135deg, #D4AF37, #F6E27A, #C5A028);
  color: #0A0C10;
  border: none;
  border-radius: 50px;
  font-weight: 800;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 5px 20px rgba(212, 175, 55, 0.3);
}

.ad-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 30px rgba(212, 175, 55, 0.4);
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

@media (max-width: 768px) {
  .circle-btn {
    width: 40px;
    height: 40px;
    font-size: 18px;
  }
  
  .lang-btn { right: 10px; bottom: 90px; }
  .support-btn { right: 60px; bottom: 90px; }
  .instagram-btn { right: 110px; bottom: 90px; }
  .offer-btn { right: 160px; bottom: 90px; }
  
  .lang-menu {
    bottom: 150px;
    right: 10px;
    width: 200px;
  }
  
  .bubble-chat-window {
    width: 95%;
  }
  
  .ad-box {
    width: 95%;
  }
  
  .bottom-nav {
    height: 70px;
  }
  
  .nav-item i {
    font-size: 20px;
  }
  
  .nav-item span {
    font-size: 10px;
  }
  
  .vip-grid {
    grid-template-columns: 1fr;
  }
  
  .commission-row {
    flex-direction: column;
    gap: 10px;
  }
  
  .ad-content {
    padding: 15px;
  }
}

@media (max-width: 480px) {
  .lang-btn { right: 5px; bottom: 85px; }
  .support-btn { right: 50px; bottom: 85px; }
  .instagram-btn { right: 95px; bottom: 85px; }
  .offer-btn { right: 140px; bottom: 85px; }
  
  .circle-btn {
    width: 38px;
    height: 38px;
    font-size: 16px;
  }
  
  .bottom-nav {
    height: 65px;
  }
  
  .nav-item i {
    font-size: 18px;
  }
  
  .nav-item span {
    font-size: 9px;
  }
  
  .ad-header h2 {
    font-size: 20px;
  }
  
  .vip-section h3 {
    font-size: 18px;
  }
}
</style>
