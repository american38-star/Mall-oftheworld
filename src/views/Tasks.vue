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

{value:0.1,color:"red"},
{value:0.2,color:"orange"},
{value:0.3,color:"orange"},
{value:0.5,color:"yellow"},
{value:0.8,color:"yellow"},
{value:1.0,color:"green"},
{value:1.2,color:"green"},
{value:1.5,color:"green"}

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

let random=Math.random()
let winIndex

// 70% خسارة
if(random<0.7){

const lose=[0,1,2,3,4]
winIndex=lose[Math.floor(Math.random()*lose.length)]

}

// 20% تعادل
else if(random<0.9){

winIndex=5

}

// 10% ربح
else{

const win=[6,7]
winIndex=win[Math.floor(Math.random()*win.length)]

}

const spins=8+Math.floor(Math.random()*5)

const targetRotation=
360*spins+
(360-(winIndex*this.segmentAngle)-this.segmentAngle/2)

const start=this.wheelRotation
const duration=3500
const startTime=performance.now()

const animate=(time)=>{

const progress=Math.min((time-startTime)/duration,1)

const ease=1-Math.pow(1-progress,3)

this.wheelRotation=start+(targetRotation*ease)

if(progress<1){

requestAnimationFrame(animate)

}else{

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

if(multiplier>1){

win=true
amount=this.bet*multiplier
const newBalance = this.balance + amount
this.$emit('update-balance', newBalance)

// تشغيل صوت الفوز
this.playSound(this.winSound)

this.$emit('show-result', `🎉 ربحت ${amount.toFixed(2)} USDT`, true)

}

else if(multiplier===1){

win=true
amount=this.bet
const newBalance = this.balance + this.bet
this.$emit('update-balance', newBalance)

// تشغيل صوت الفوز
this.playSound(this.winSound)

this.$emit('show-result', `🤝 استرداد الرهان`, true)

} else {

// تشغيل صوت الخسارة
this.playSound(this.loseSound)

this.$emit('show-result', `😢 خسرت الرهان`, false)

}

this.result={
win,
amount,
multiplier
}

this.bet=0

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
width:300px;
height:300px;
margin:auto;
}

.pointer{
position:absolute;
top:-15px;
left:50%;
transform:translateX(-50%);
font-size:50px;
color:gold;
z-index:10;
text-shadow: 0 0 15px gold;
animation: pointerGlow 1.5s infinite;
}

@keyframes pointerGlow {
  0%, 100% { text-shadow: 0 0 10px gold; }
  50% { text-shadow: 0 0 25px gold; }
}

.wheel{

width:100%;
height:100%;

border-radius:50%;

border:5px solid gold;

position:relative;

overflow:hidden;

box-shadow: 0 0 30px rgba(255,215,0,0.3);

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

transition: filter 0.2s;

border: 1px solid rgba(255,255,255,0.1);

}

.label{

transform:rotate(-45deg);

font-weight:bold;

font-size: 18px;

text-shadow: 0 2px 5px black;

}

.red{
  background: linear-gradient(135deg, #f44336, #b71c1c);
}

.orange{
  background: linear-gradient(135deg, #ff9800, #f57c00);
}

.yellow{
  background: linear-gradient(135deg, #ffeb3b, #fbc02d);
  color:black;
}

.green{
  background: linear-gradient(135deg, #4caf50, #1b5e20);
}

.controls{

margin-top:40px;

display:flex;

gap:10px;

justify-content:center;

flex-wrap: wrap;

}

input{

padding:15px 20px;

border-radius:50px;

border:none;

background: rgba(30,35,51,0.8);

color:white;

font-size:16px;

width: 200px;

border: 2px solid rgba(255,215,0,0.3);

text-align: center;

transition: all 0.3s;

}

input:focus{
  outline: none;
  border-color: gold;
  box-shadow: 0 0 20px rgba(255,215,0,0.3);
}

input:disabled{
  opacity: 0.5;
  cursor: not-allowed;
}

button{

padding:15px 30px;

background: linear-gradient(135deg, gold, #ffd700);

border:none;

border-radius:50px;

cursor:pointer;

font-weight:bold;

font-size:16px;

color: #0a0f1e;

transition: all 0.3s;

box-shadow: 0 10px 20px rgba(255,215,0,0.3);

}

button:hover:not(:disabled){
  transform: translateY(-3px);
  box-shadow: 0 15px 30px rgba(255,215,0,0.5);
}

button:disabled{
  opacity: 0.5;
  cursor: not-allowed;
}

.result{

margin-top:30px;

font-size:20px;

padding: 20px;

border-radius: 50px;

animation: slideUp 0.5s;

background: rgba(0,0,0,0.3);

border: 1px solid;

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
  font-size: 30px;
  margin-left: 10px;
  vertical-align: middle;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
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
  
  .wheel-wrapper {
    width: 250px;
    height: 250px;
  }
  
  .label {
    font-size: 14px;
  }
  
  .controls {
    flex-direction: column;
    align-items: center;
  }
  
  input, button {
    width: 100%;
    max-width: 280px;
  }
}

</style>
