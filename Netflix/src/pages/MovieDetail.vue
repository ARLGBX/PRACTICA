<template>
  <div class="container">
    <button class="back-btn" @click="goBack">← Назад к каталогу</button>

    <div class="movie">
      <div class="poster">
        <img :src="movie.poster || 'https://via.placeholder.com/300x450'" alt="poster" />
      </div>

      <div class="info">
        <h1>{{ movie.title }}</h1>
        <div class="rating">⭐ {{ movie.rating }}</div>
        <p class="description">{{ movie.description || 'Описание скоро появится' }}</p>

        <button class="action" @click="toggleFavorite">
          {{ isFavorite ? "❤️ Убрать из избранного" : "♡ Добавить в избранное" }}
        </button>
      </div>
    </div>

    <div class="reviews">
      <h2>Отзывы</h2>
      <div class="review" v-for="r in reviews" :key="r.id">
        <div class="review-header">
          <span class="user">{{ r.user }}</span>
          <span>👍 {{ r.likes }}</span>
        </div>
        <p>{{ r.text }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { doc, getDoc } from 'firebase/firestore'
import { db } from '../firebase/config.js'

const route = useRoute()
const router = useRouter()

const movieId = route.params.id
const movie = ref({ title: "Загрузка...", description: "", rating: "—", poster: "" })
const reviews = ref([
  { id: 1, user: "User1", text: "Очень крутой фильм! Рекомендую.", likes: 45 },
  { id: 2, user: "KinoFan", text: "Отличная режиссура и актёрская игра.", likes: 22 }
])
const isFavorite = ref(false)

const toggleFavorite = () => {
  isFavorite.value = !isFavorite.value
}

const goBack = () => router.push('/')

onMounted(async () => {
  if (!movieId) return

  const docRef = doc(db, 'movies', movieId)
  const docSnap = await getDoc(docRef)

  if (docSnap.exists()) {
    movie.value = { id: docSnap.id, ...docSnap.data() }
  } else {
    movie.value = {
      title: "Фильм не найден",
      description: "Скоро появится",
      rating: "—",
      poster: "https://via.placeholder.com/300x450"
    }
  }
})
</script>

<style scoped>
.container { max-width: 1400px; margin: 0 auto; padding: 30px; background: #141414; color: #fff; min-height: 100vh; }
.back-btn { background: transparent; color: #aaa; border: 1px solid #555; padding: 12px 24px; border-radius: 8px; cursor: pointer; margin-bottom: 30px; font-size: 16px; }
.back-btn:hover { background: #333; color: white; }
.movie { display: flex; gap: 50px; margin-bottom: 60px; }
.poster { width: 340px; height: 500px; border-radius: 12px; overflow: hidden; flex-shrink: 0; }
.poster img { width: 100%; height: 100%; object-fit: cover; }
.info h1 { font-size: 42px; margin-bottom: 15px; }
.rating { font-size: 24px; color: #ffd700; margin-bottom: 25px; }
.description { font-size: 18px; line-height: 1.7; color: #ccc; max-width: 800px; }
.action { margin-top: 30px; padding: 16px 36px; background: #e50914; color: white; border: none; border-radius: 8px; font-size: 17px; cursor: pointer; }
.reviews { margin-top: 50px; }
.review { background: #1c1c1c; padding: 20px; border-radius: 12px; margin-bottom: 20px; }
.review-header { display: flex; justify-content: space-between; margin-bottom: 12px; }
.user { color: #e50914; font-weight: bold; }
</style>