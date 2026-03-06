<template>

<div class="games-page">

<!-- شريط الرصيد -->
<div class="top-bar">

<div class="balance">
💰 Balance : ${{ balance.toFixed(2) }}
</div>

</div>

<!-- قائمة الألعاب -->
<div v-if="!currentGame" class="games-grid">

<div class="game-card" @click="openGame('bigwheel')">
<img src="@/assets/bigwheel.jpg">
<p>Big Wheel</p>
</div>

<div class="game-card" @click="openGame('chicken')">
<img src="@/assets/chicken.jpg">
<p>Chicken Crash</p>
</div>

</div>

<!-- زر الرجوع -->
<div v-if="currentGame" class="back-bar">

<button class="back-btn" @click="closeGame">
⬅ Back
</button>

</div>

<!-- الألعاب -->

<BigWheelGame
v-if="currentGame === 'bigwheel'"
:balance="balance"
@updateBalance="updateBalance"
/>

<ChickenCrashGame
v-if="currentGame === 'chicken'"
:balance="balance"
@updateBalance="updateBalance"
/>

</div>

</template>

<script>

import BigWheelGame from "@/components/games/BigWheelGame.vue"
import ChickenCrashGame from "@/components/games/ChickenCrashGame.vue"

export default {

components:{
BigWheelGame,
ChickenCrashGame
},

data(){
return{

balance:100,
currentGame:null

}
},

methods:{

openGame(game){

this.currentGame = game

},

closeGame(){

this.currentGame = null

},

updateBalance(amount){

this.balance += amount

}

}

}

</script>

<style scoped>

.games-page{

background:#0e0e0e;
min-height:100vh;
color:white;
padding-bottom:40px;

}

.top-bar{

display:flex;
justify-content:flex-end;
padding:15px;
background:#1b1b1b;

}

.balance{

font-size:18px;
font-weight:bold;
color:#00ff88;

}

.games-grid{

display:grid;
grid-template-columns:repeat(2,1fr);
gap:15px;
padding:20px;

}

.game-card{

background:#1e1e1e;
border-radius:12px;
overflow:hidden;
text-align:center;
cursor:pointer;
transition:0.2s;

}

.game-card:hover{

transform:scale(1.05);

}

.game-card img{

width:100%;
height:140px;
object-fit:cover;

}

.game-card p{

padding:10px;
font-weight:bold;

}

.back-bar{

padding:15px;

}

.back-btn{

background:#ff4444;
border:none;
padding:10px 20px;
color:white;
font-size:16px;
border-radius:8px;
cursor:pointer;

}

</style>
