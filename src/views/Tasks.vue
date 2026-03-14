<template>
<div class="game">

  <div class="balance">
    رصيدك: <b>{{ balance.toFixed(2) }} USDT</b>
  </div>

  <div class="wheel-wrapper">

    <div class="pointer">▼</div>

    <div
      class="wheel"
      :style="{ transform: `rotate(${wheelRotation}deg)` }"
    >

      <div
        v-for="(segment,index) in wheelSegments"
        :key="index"
        class="segment"
        :class="segment.color"
        :style="{ transform:`rotate(${index*segmentAngle}deg)` }"
      >
        <span class="label">{{ segment.value }}x</span>
      </div>

    </div>

  </div>

  <div class="controls">

    <input
      type="number"
      v-model.number="bet"
      placeholder="مبلغ الرهان"
      :disabled="isSpinning"
    />

    <button
      @click="spin"
      :disabled="isSpinning || bet<=0 || bet>balance"
    >
      {{ isSpinning ? "جاري الدوران..." : "ابدأ الدوران" }}
    </button>

  </div>

  <div v-if="result" class="result" :class="{ 'win': result.win, 'lose': !result.win }">

    <div v-if="result.win">
      <span class="result-icon">🎉</span>
      ربحت {{ result.amount.toFixed(2) }} USDT (x{{ result.multiplier }})
    </div>

    <div v-else>
      <span class="result-icon">😢</span>
      خسرت الجولة (x{{ result.multiplier }})
    </div>

  </div>

</div>
</template>

<script>
export default {

name:"WheelGame",

props: {
  balance: {
    type: Number,
    required: true
  }
},

emits: ['update-balance', 'show-result'],

data(){

return{

bet:0,

isSpinning:false,

wheelRotation:0,

result:null,

wheelSegments:[

{value:0, color:"red"},
{value:0.5, color:"orange"},
{value:1, color:"orange"},
{value:1.5, color:"green"},
{value:2, color:"green"},
{value:3, color:"green"},
{value:5, color:"green"},
{value:10, color:"gold"}

],

// أصوات اللعبة
spinSound: null,
winSound: null,
loseSound: null,
clickSound: null

}

},

computed:{

segmentAngle(){
return 360/this.wheelSegments.length
}

},

mounted() {
  // تهيئة الأصوات
  this.initSounds()
},

methods:{

initSounds() {
  // إنشاء أصوات باستخدام Web Audio API
  try {
    // صوت الدوران
    this.spinSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
    
    // صوت الفوز
    this.winSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
    
    // صوت الخسارة
    this.loseSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
    
    // صوت النقر
    this.clickSound = new Audio('data:audio/wav;base64,UklGRlwAAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YVQAAACAgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f3+AgICAf39/f39/f38=')
    
    // ضبط مستوى الصوت
    this.spinSound.volume = 0.3
    this.winSound.volume = 0.5
    this.loseSound.volume = 0.4
    this.clickSound.volume = 0.2
    
  } catch (e) {
    console.log('الصوت غير مدعوم في هذا المتصفح')
  }
},

playSound(sound) {
  if (sound) {
    sound.currentTime = 0
    sound.play().catch(e => console.log('تعذر تشغيل الصوت'))
  }
},

spin(){

if(this.bet>this.balance)return

// تشغيل صوت النقر
this.playSound(this.clickSound)

// التحقق من صحة الرهان
if (!this.bet || this.bet <= 0) {
  this.$emit('show-result', '❌ مبلغ الرهان غير صحيح', false)
  return
}

if (this.bet > this.balance) {
  this.$emit('show-result', '❌ الرصيد غير كافي', false)
  return
}

this.isSpinning=true
this.result=null

// تشغيل صوت الدوران
this.playSound(this.spinSound)

// خصم الرهان
const newBalance = this.balance - this.bet
this.$emit('update-balance', newBalance)

// اختيار الجزء الفائز بناءً على توزيع عشوائي لصالح الموقع
let random=Math.random()
let winIndex

// توزيع النتائج (لصالح الموقع)
if(random<0.3){ // 30% أحمر (0x)
  winIndex=0
}
else if(random<0.5){ // 20% برتقالي (0.5x, 1x)
  const orange=[1,2]
  winIndex=orange[Math.floor(Math.random()*orange.length)]
}
else if(random<0.7){ // 20% أخضر منخفض (1.5x, 2x)
  const greenLow=[3,4]
  winIndex=greenLow[Math.floor(Math.random()*greenLow.length)]
}
else if(random<0.85){ // 15% أخضر متوسط (3x, 5x)
  const greenMid=[5,6]
  winIndex=greenMid[Math.floor(Math.random()*greenMid.length)]
}
else{ // 15% ذهبي (10x)
  winIndex=7
}

const spins=10+Math.floor(Math.random()*8) // 10-18 دورة

// حساب الزاوية المستهدفة بدقة
// نريد أن يتوقف السهم (في الأعلى) على الجزء الفائز
// الجزء رقم winIndex يبدأ عند زاوية winIndex * segmentAngle
// السهم في الأعلى يشير إلى الزاوية 90 درجة
// لذلك يجب أن تكون الزاوية النهائية بحيث يكون الجزء الفائز في الأعلى
const targetRotation = 360 * spins + (90 - (winIndex * this.segmentAngle) - this.segmentAngle/2)

const start=this.wheelRotation
const duration=4000
const startTime=performance.now()

const animate=(time)=>{

const progress=Math.min((time-startTime)/duration,1)

// استخدام منحنى التباطؤ الطبيعي
const ease = 1 - Math.pow(1 - progress, 3)

this.wheelRotation = start + ((targetRotation - start) * ease)

if(progress<1){

requestAnimationFrame(animate)

}else{

// التأكد من الزاوية النهائية مضبوطة
this.wheelRotation = targetRotation
this.finish(winIndex)

}

}

requestAnimationFrame(animate)

},

finish(index){

this.isSpinning=false

const multiplier=this.wheelSegments[index].value

let win=false
let amount=0
let message=''

if(multiplier>1){

win=true
amount=this.bet*multiplier
const newBalance = this.balance + amount
this.$emit('update-balance', newBalance)

// تشغيل صوت الفوز
this.playSound(this.winSound)

message = `🎉 ربحت ${amount.toFixed(2)} USDT`
this.$emit('show-result', message, true)

}

else if(multiplier===1){

win=true
amount=this.bet
const newBalance = this.balance + this.bet
this.$emit('update-balance', newBalance)

// تشغيل صوت الفوز
this.playSound(this.winSound)

message = `🤝 استرداد الرهان`
this.$emit('show-result', message, true)

}

else if(multiplier===0.5){

win=false
amount=this.bet*0.5
const newBalance = this.balance + amount
this.$emit('update-balance', newBalance)

// تشغيل صوت الخسارة
this.playSound(this.loseSound)

message = `😢 خسرت نصف الرهان`
this.$emit('show-result', message, false)

}

else {

// تشغيل صوت الخسارة
this.playSound(this.loseSound)

message = `😢 خسرت كل الرهان`
this.$emit('show-result', message, false)

}

this.result={
win,
amount,
multiplier
}

// إعادة تعيين الرهان بعد ثانيتين
setTimeout(() => {
  this.bet=0
}, 2000)

}

}

}
</script>

<style scoped>

.game{
text-align:center;
padding:30px;
background: linear-gradient(135deg, #0a0f1e 0%, #1a1f2f 100%);
min-height:100vh;
color:white;
font-family: 'Cairo', sans-serif;
}

.balance{
font-size:20px;
margin-bottom:30px;
padding: 15px 25px;
background: rgba(0,0,0,0.3);
border-radius: 50px;
display: inline-block;
border: 1px solid gold;
}

.balance b{
color: gold;
font-size: 22px;
margin-right: 5px;
}

.wheel-wrapper{
position:relative;
width: 350px;
height: 350px;
margin: 20px auto;
}

@media (max-width: 480px) {
  .wheel-wrapper {
    width: 280px;
    height: 280px;
  }
}

.pointer{
position:absolute;
top:-15px;
left:50%;
transform:translateX(-50%);
font-size:60px;
color:gold;
z-index:10;
text-shadow: 0 0 20px gold, 0 0 40px rgba(255,215,0,0.5);
animation: pointerGlow 1.5s infinite;
}

@keyframes pointerGlow {
  0%, 100% { text-shadow: 0 0 15px gold; }
  50% { text-shadow: 0 0 30px gold; }
}

.wheel{
width:100%;
height:100%;
border-radius:50%;
border:6px solid gold;
position:relative;
overflow:hidden;
box-shadow: 0 0 40px rgba(255,215,0,0.4);
background: #222;
transition: transform 4s cubic-bezier(0.25, 0.1, 0.15, 1);
}

.segment{
position:absolute;
width:50%;
height:50%;
left:50%;
top:50%;
transform-origin:0% 0%;
clip-path:polygon(0 0,100% 0,0 100%);
display:flex;
align-items:center;
justify-content:center;
border: 1px solid rgba(255,255,255,0.15);
box-sizing: border-box;
}

.label{
transform:rotate(-45deg);
font-weight:bold;
font-size: 24px;
text-shadow: 0 2px 5px black;
color: white;
}

@media (max-width: 480px) {
  .label {
    font-size: 18px;
  }
}

/* ألوان الأجزاء */
.segment.red{
  background: linear-gradient(135deg, #d32f2f, #b71c1c);
}

.segment.orange{
  background: linear-gradient(135deg, #fb8c00, #f57c00);
  color: white;
}

.segment.green{
  background: linear-gradient(135deg, #388e3c, #2e7d32);
}

.segment.gold{
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  position: relative;
  overflow: hidden;
}

.segment.gold::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at 30% 30%, rgba(255,255,255,0.3), transparent);
  pointer-events: none;
}

.controls{
margin-top:40px;
display:flex;
gap:15px;
justify-content:center;
flex-wrap: wrap;
}

input{
padding:15px 25px;
border-radius:50px;
border:none;
background: rgba(30,35,51,0.9);
color:white;
font-size:16px;
width: 220px;
border: 2px solid rgba(255,215,0,0.3);
text-align: center;
transition: all 0.3s;
}

input:focus{
outline: none;
border-color: gold;
box-shadow: 0 0 25px rgba(255,215,0,0.4);
}

input:disabled{
opacity: 0.5;
cursor: not-allowed;
}

button{
padding:15px 35px;
background: linear-gradient(135deg, gold, #ffd700);
border:none;
border-radius:50px;
cursor:pointer;
font-weight:bold;
font-size:18px;
color: #0a0f1e;
transition: all 0.3s;
box-shadow: 0 10px 25px rgba(255,215,0,0.4);
min-width: 200px;
}

button:hover:not(:disabled){
transform: translateY(-4px);
box-shadow: 0 18px 35px rgba(255,215,0,0.6);
}

button:disabled{
opacity: 0.5;
cursor: not-allowed;
}

.result{
margin-top:35px;
font-size:22px;
padding: 20px 30px;
border-radius: 60px;
animation: slideUp 0.5s;
background: rgba(0,0,0,0.4);
border: 2px solid;
display: inline-block;
backdrop-filter: blur(5px);
}

.result.win{
border-color: #4caf50;
color: #4caf50;
}

.result.lose{
border-color: #f44336;
color: #f44336;
}

.result-icon{
font-size: 35px;
margin-left: 12px;
vertical-align: middle;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(25px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 480px) {
  .game {
    padding: 15px;
  }
  
  .balance {
    font-size: 16px;
    padding: 12px 20px;
  }
  
  .balance b {
    font-size: 18px;
  }
  
  .pointer {
    font-size: 45px;
    top: -10px;
  }
  
  .controls {
    flex-direction: column;
    align-items: center;
    gap: 12px;
  }
  
  input, button {
    width: 100%;
    max-width: 280px;
    padding: 12px 20px;
    font-size: 16px;
  }
  
  .result {
    font-size: 18px;
    padding: 15px 20px;
  }
  
  .result-icon {
    font-size: 25px;
  }
}

</style>
