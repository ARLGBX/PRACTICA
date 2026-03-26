<template>
  <div class="container">
    <!-- Шапка -->
    <nav class="navbar">
      <div class="nav-left">
        <div class="logo">NETFLIX</div>
        <div class="nav-links">
          <a href="#" class="nav-link active">Главная</a>
          <a href="#" class="nav-link">Фильмы</a>
          <a href="#" class="nav-link">Сериалы</a>
        </div>
      </div>

      <div class="nav-right">
        <input
            class="search"
            type="text"
            placeholder="Поиск фильмов..."
            v-model="searchQuery"
        />
        <div class="user-icon">👤</div>
      </div>
    </nav>

    <!-- Каталог карточек -->
    <div class="catalog">
      <div
          class="card"
          v-for="movie in filteredMovies"
          :key="movie.id"
          @click="goToMovie(movie.id)"
      >
        <div class="card-poster" :style="{ backgroundImage: `url(${movie.poster})` }">
          <div class="card-gradient"></div>
          <div class="card-info">
            <h3>{{ movie.title }}</h3>
            <div class="card-stats">
              <span class="rating">⭐ {{ movie.rating }}</span>
              <span class="reviews">💬 {{ movie.reviewsCount }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="loading" class="loading">Загрузка фильмов...</div>

    <div class="load-more">
      <button @click="loadMore">Показать ещё</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { collection, getDocs } from 'firebase/firestore'
import { db } from '../firebase/config.js'

const router = useRouter()

const movies = ref([])
const searchQuery = ref('')
const loading = ref(true)

// Переход на страницу фильма
const goToMovie = (id) => {
  console.log(`Переход на фильм ID: ${id}`)
  router.push(`/movie/${id}`)
}

// Фильтрация по поиску
const filteredMovies = computed(() => {
  if (!searchQuery.value) return movies.value
  const q = searchQuery.value.toLowerCase()
  return movies.value.filter(movie =>
      movie.title.toLowerCase().includes(q)
  )
})

// Загрузка фильмов из Firebase
onMounted(async () => {
  try {
    const snap = await getDocs(collection(db, 'movies'))
    movies.value = snap.docs.map(doc => ({
      id: doc.id,
      ...doc.data()
    }))
  } catch (e) {
    console.error('Ошибка загрузки фильмов', e)
  } finally {
    loading.value = false
  }
})

// Пока кнопка "Показать ещё" не работает
const loadMore = () => alert('Подгрузка фильмов скоро будет подключена')
</script>

<style scoped>
/* Твои стили оставляем как были */
.container { max-width: 1400px; margin: 0 auto; padding: 30px; background: #141414; color: #fff; min-height: 100vh; }
.navbar { display: flex; align-items: center; justify-content: space-between; padding: 20px 50px; background: rgba(0,0,0,0.9); position: sticky; top: 0; z-index: 100; }
.nav-left { display: flex; align-items: center; gap: 40px; }
.logo { font-size: 34px; font-weight: 900; color: #e50914; }
.nav-links a { color: #ddd; text-decoration: none; font-size: 15.5px; margin-right: 25px; }
.nav-links .active { color: white; }
.nav-right { display: flex; align-items: center; gap: 15px; }
.search { padding: 10px 20px; width: 320px; background: rgba(255,255,255,0.12); border: none; border-radius: 4px; color: white; }
.user-icon { width: 40px; height: 40px; background: #e50914; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 22px; cursor: pointer; }
.catalog { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 28px; padding: 40px 50px; }
.card { height: 420px; border-radius: 16px; overflow: hidden; cursor: pointer; transition: transform 0.4s ease; }
.card:hover { transform: scale(1.07); }
.card-poster { width: 100%; height: 100%; background-size: cover; background-position: center; position: relative; }
.card-gradient { position: absolute; inset: 0; background: linear-gradient(to top, rgba(0,0,0,0.95), rgba(0,0,0,0.65), transparent); }
.card-info { position: absolute; bottom: 0; left: 0; right: 0; padding: 25px 22px; z-index: 2; color: white; }
.card-info h3 { margin: 0 0 12px 0; font-size: 19px; line-height: 1.35; }
.card-stats { display: flex; justify-content: space-between; font-size: 15px; }
.rating { color: #ffd700; }
.reviews { color: #ccc; }
.load-more { text-align: center; padding: 50px 0; }
.load-more button { padding: 14px 55px; background: #e50914; color: white; border: none; border-radius: 6px; font-size: 16px; cursor: pointer; }
.loading { text-align: center; color: #aaa; font-size: 20px; margin: 20px 0; }
</style>