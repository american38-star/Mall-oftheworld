<template>
  <div class="game-page">
    <!-- الرصيد و القائمة موجودة كما هي -->
    
    <!-- الألعاب القديمة -->
    <ChickenRoad v-if="selectedGame === 'chicken'" />
    <!-- ... باقي الألعاب القديمة ... -->
    
    <!-- اللعبة الجديدة -->
    <ChickenRoadGame 
      v-if="selectedGame === 'chicken-game'"
      :balance="balance"
      @update-balance="updateBalance"
      @show-result="showResult"
    />
  </div>
</template>

<script>
// استيراد الألعاب القديمة
import ChickenRoad from './components/games/ChickenRoad.vue'
// ... باقي الألعاب القديمة ...

// استيراد اللعبة الجديدة
import ChickenRoadGame from './components/games/ChickenRoadGame.vue'

export default {
  components: {
    ChickenRoad,
    ChickenRoadGame,
    // ... باقي الألعاب القديمة
  },
  // باقي الكود كما هو
}
</script>
