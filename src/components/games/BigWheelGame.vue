<template>
<div class="bigwheel-container">

<h2 class="title">Big Wheel Bonus</h2>

<div class="balance-box">
💰 Balance: {{ balance }} $
</div>

<div class="wheel-area">

<img
src="@/assets/bigwheel.jpg"
class="wheel"
:style="{ transform: 'rotate(' + rotation + 'deg)' }"
/>

<div class="pointer"></div>

</div>

<div class="controls">

<input
type="number"
v-model.number="bet"
class="bet-input"
min="1"
/>

<button
class="spin-btn"
@click="spinWheel"
:disabled="spinning"
>
Spin
</button>

</div>

<div class="result">
{{ resultMessage }}
</div>

</div>
</template>

<script>
export default {

name: "BigWheelGame",

data(){
return{

balance:1000,

bet:10,

rotation:0,

spinning:false,

resultMessage:""

}
},

methods:{

spinWheel(){

if(this.bet > this.balance){
this.resultMessage = "Not enough balance"
return
}

this.balance -= this.bet

this.spinning = true

const prizes = [0,2,3,5,10]

const randomIndex = Math.floor(Math.random()*prizes.length)

const winMultiplier = prizes[randomIndex]

const winAmount = this.bet * winMultiplier

const spinDeg = 3600 + (randomIndex * 72)

this.rotation += spinDeg

setTimeout(()=>{

this.spinning = false

this.balance += winAmount

if(winAmount > 0){

this.resultMessage = "You won " + winAmount + " $ 🎉"

}else{

this.resultMessage = "No win 😢"

}

},3000)

}

}

}
</script>

<style scoped>

.bigwheel-container{
text-align:center;
color:white;
background:#121212;
padding:20px;
min-height:100vh;
}

.title{
font-size:28px;
margin-bottom:20px;
}

.balance-box{
font-size:20px;
margin-bottom:20px;
background:#1f1f1f;
padding:10px;
border-radius:10px;
}

.wheel-area{
position:relative;
width:260px;
margin:auto;
}

.wheel{
width:260px;
transition:transform 3s ease-out;
}

.pointer{
position:absolute;
top:-10px;
left:50%;
transform:translateX(-50%);
width:0;
height:0;
border-left:15px solid transparent;
border-right:15px solid transparent;
border-bottom:30px solid red;
}

.controls{
margin-top:20px;
}

.bet-input{
padding:8px;
width:80px;
text-align:center;
border-radius:5px;
border:none;
}

.spin-btn{
padding:10px 20px;
margin-left:10px;
background:#00c853;
border:none;
border-radius:6px;
color:white;
font-size:16px;
cursor:pointer;
}

.spin-btn:disabled{
background:#777;
}

.result{
margin-top:20px;
font-size:18px;
}

</style>
