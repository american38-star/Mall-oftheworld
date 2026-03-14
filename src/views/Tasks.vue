<!-- src/views/Tasks.vue -->
<template>
  <div class="games-page">

    <!-- الرصيد -->
    <div class="balance-container">
      <div class="balance-card">
        <i class="fas fa-coins"></i>
        <span>Balance: ${{ balance.toFixed(2) }}</span>
      </div>
    </div>

    <!-- رسالة النتيجة -->
    <transition name="fade">
      <div
        v-if="resultMessage.show"
        class="result-message"
        :class="resultMessage.type"
      >
        <i :class="resultMessage.icon"></i>
        <span>{{ resultMessage.text }}</span>
      </div>
    </transition>

    <!-- التحميل -->
    <div v-if="loading" class="loading">
      <i class="fas fa-spinner fa-spin"></i>
      <span>جاري تحميل الألعاب...</span>
    </div>

    <!-- الألعاب -->
    <div v-else class="games-container">

      <!-- الألعاب المميزة -->
      <div class="featured-row">

        <!-- Chicken -->
        <div
          v-if="chickenGame"
          class="featured-card chicken-card"
          @click="openGame(chickenGame)"
        >
          <div class="game-image-container featured">
            <img
              v-if="chickenGame.image"
              :src="chickenGame.image"
              class="game-image"
              @error="handleImageError"
            >
            <div v-else class="game-icon featured">
              {{ chickenGame.icon }}
            </div>
          </div>

          <div class="game-info">
            <h3 class="game-name featured">
              {{ chickenGame.name }}
            </h3>
          </div>
        </div>

        <!-- Wheel -->
        <div
          v-if="bigWheelGame"
          class="featured-card wheel-card"
          @click="openGame(bigWheelGame)"
        >
          <div class="game-image-container featured">
            <img
              v-if="bigWheelGame.image"
              :src="bigWheelGame.image"
              class="game-image"
              @error="handleImageError"
            >
            <div v-else class="game-icon featured">
              {{ bigWheelGame.icon }}
            </div>
          </div>

          <div class="game-info">
            <h3 class="game-name featured">
              {{ bigWheelGame.name }}
            </h3>
          </div>
        </div>

      </div>

      <!-- باقي الألعاب -->
      <div v-if="otherGames.length" class="games-grid">

        <div
          v-for="game in otherGames"
          :key="game.id"
          class="game-card"
          @click="openGame(game)"
        >

          <div class="game-image-container">

            <img
              v-if="game.image"
              :src="game.image"
              class="game-image"
              @error="handleImageError"
            >

            <div v-else class="game-icon">
              {{ game.icon }}
            </div>

          </div>

          <h3 class="game-name">{{ game.name }}</h3>
          <p class="game-description">{{ game.description }}</p>

        </div>

      </div>
    </div>

    <!-- نافذة اللعبة -->
    <transition name="slide-up">
      <div v-if="selectedGame" class="game-modal">

        <div class="game-modal-header">
          <button class="close-button" @click="closeGame">
            ✕
          </button>
          <h2>{{ selectedGame.name }}</h2>
        </div>

        <div class="game-modal-content">
          <component
            :is="selectedGame.component"
            :balance="balance"
            @update-balance="updateBalance"
            @show-result="showResult"
          />
        </div>

      </div>
    </transition>

  </div>
</template>

<script>
import { auth, getUserBalance, updateUserBalance, addTransaction } from "../firebase"
import { shallowRef } from "vue"

export default {

  name: "Tasks",

  setup() {

    const gameModules =
      import.meta.glob("../components/games/*.vue", { eager: true })

    const imageModules =
      import.meta.glob("../assets/images/*", { eager: true })

    const games = []

    for (const path in gameModules) {

      const component = gameModules[path].default

      const fileName =
        path.split("/").pop().replace(".vue", "")

      const lower =
        fileName.toLowerCase()

      let image = null

      for (const img in imageModules) {

        if (img.toLowerCase().includes(lower)) {

          image = imageModules[img].default
          break
        }
      }

      let name =
        fileName.replace(/([A-Z])/g, " $1")

      let icon = "🎮"

      let description = "اضغط للعب"

      if (component.meta) {

        name =
          component.meta.name || name

        icon =
          component.meta.icon || icon

        description =
          component.meta.description || description
      }

      games.push({
        id: fileName,
        name,
        icon,
        image,
        description,
        component: shallowRef(component)
      })
    }

    return {
      gamesComponents: games
    }
  },

  data() {

    return {

      balance: 0,

      loading: true,

      selectedGame: null,

      resultMessage: {
        show: false,
        type: "",
        icon: "",
        text: ""
      }
    }
  },

  computed: {

    chickenGame() {

      return this.gamesComponents.find(g =>
        g.id.toLowerCase().includes("chicken"))
    },

    bigWheelGame() {

      return this.gamesComponents.find(g =>
        g.id.toLowerCase().includes("wheel"))
    },

    otherGames() {

      const ids = [
        this.chickenGame?.id,
        this.bigWheelGame?.id
      ]

      return this.gamesComponents
        .filter(g => !ids.includes(g.id))
    }
  },

  async created() {

    const user = auth.currentUser

    if (!user) {

      this.loading = false
      return
    }

    this.balance =
      await getUserBalance(user.uid)

    this.loading = false
  },

  methods: {

    openGame(game) {

      this.selectedGame = game
    },

    closeGame() {

      this.selectedGame = null
    },

    async updateBalance(newBalance) {

      const user = auth.currentUser

      if (!user) return

      this.balance = newBalance

      await updateUserBalance(
        user.uid,
        newBalance
      )
    },

    showResult(message, win) {

      this.resultMessage = {

        show: true,

        text: message,

        type: win ? "win" : "lose",

        icon: win
          ? "fas fa-trophy"
          : "fas fa-skull"
      }

      setTimeout(() => {

        this.resultMessage.show = false

      }, 2000)
    },

    handleImageError(e) {

      e.target.style.display = "none"
    }
  }
}
</script>

<style scoped>

.games-page{
min-height:100vh;
background:#1a1f2e;
padding:20px;
color:white
}

.balance-container{
display:flex;
justify-content:center;
margin-bottom:30px
}

.balance-card{
background:#2a2f42;
padding:15px 40px;
border-radius:40px;
font-size:22px
}

.featured-row{
display:grid;
grid-template-columns:1fr 1fr;
gap:20px
}

.featured-card{
background:#2a2f42;
padding:20px;
border-radius:12px;
cursor:pointer
}

.games-grid{
display:grid;
grid-template-columns:repeat(auto-fill,minmax(250px,1fr));
gap:20px;
margin-top:40px
}

.game-card{
background:#2a2f42;
padding:15px;
border-radius:10px;
cursor:pointer
}

.game-image-container{
height:150px;
background:#1e2335;
margin-bottom:10px
}

.game-image{
width:100%;
height:100%;
object-fit:cover
}

.game-icon{
display:flex;
align-items:center;
justify-content:center;
height:100%;
font-size:50px
}

.game-modal{
position:fixed;
top:0;
left:0;
right:0;
bottom:0;
background:black
}

.game-modal-header{
background:#1e2335;
padding:15px;
display:flex;
gap:10px
}

.close-button{
background:none;
border:none;
color:#ffd700;
font-size:22px;
cursor:pointer
}

</style>
