<template>
  <div class="watchlist">
    <nav class="navbar">...</nav>

    <div class="watchlist-container">
      <h1>Смотреть позже</h1>

      <div v-if="watchlist.length === 0" class="empty">
        <p>У вас нет фильмов в списке "Смотреть позже"</p>
        <router-link to="/" class="btn-primary">На главную</router-link>
      </div>

      <div v-else class="watchlist-grid">
        <div v-for="item in watchlist" :key="item.movieId" class="watchlist-card">
          <img :src="item.poster" class="card-poster">
          <div class="card-info">
            <h3>{{ item.title }}</h3>
            <div class="card-meta">{{ item.year }} · {{ item.duration }} мин</div>
            <div class="card-actions">
              <button @click="watchNow(item.movieId)" class="btn-watch">Смотреть</button>
              <button @click="removeFromWatchlist(item.movieId)" class="btn-remove">Удалить</button>
            </div>
          </div>
        </div>
      </div>

      <div class="watchlist-actions">
        <button @click="clearAll" class="btn-clear">Очистить все</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { collection, getDocs, deleteDoc, doc, query, where } from 'firebase/firestore'
import { db, auth } from '../firebase/config'
import { onAuthStateChanged } from 'firebase/auth'
import { useRouter } from 'vue-router'

const router = useRouter()
const user = ref(null)
const watchlist = ref([])

onAuthStateChanged(auth, (currentUser) => {
  user.value = currentUser
  if (currentUser) loadWatchlist()
})

const loadWatchlist = async () => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid))
  const snapshot = await getDocs(q)
  watchlist.value = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))
}

const removeFromWatchlist = async (movieId) => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid), where('movieId', '==', movieId))
  const snapshot = await getDocs(q)
  snapshot.forEach(async (doc) => await deleteDoc(doc.ref))
  await loadWatchlist()
}

const watchNow = (movieId) => router.push(`/movie/${movieId}`)

const clearAll = async () => {
  const q = query(collection(db, 'watchlist'), where('userId', '==', user.value.uid))
  const snapshot = await getDocs(q)
  snapshot.forEach(async (doc) => await deleteDoc(doc.ref))
  await loadWatchlist()
}
</script>

<style scoped>
.watchlist {
  min-height: 100vh;
  background: #141414;
  color: #fff;
  font-family: Arial, sans-serif;
}

.navbar {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px 20px;
  background: #111;
  position: sticky;
  top: 0;
  z-index: 10;
}

.watchlist-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 50px;
}

.watchlist-container h1 {
  font-size: 32px;
  font-weight: 700;
  margin-bottom: 30px;
}

.empty {
  text-align: center;
  padding: 80px 20px;
  color: #aaa;
}

.empty p {
  font-size: 18px;
  margin-bottom: 20px;
}

.btn-primary {
  display: inline-block;
  padding: 12px 25px;
  background: #e50914;
  color: white;
  border: none;
  border-radius: 6px;
  text-decoration: none;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-primary:hover {
  background: #b20710;
}

.watchlist-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.watchlist-card {
  display: flex;
  background: #1c1c1c;
  border-radius: 10px;
  overflow: hidden;
  transition: transform 0.3s ease;
}

.watchlist-card:hover {
  transform: scale(1.02);
}

.card-poster {
  width: 100px;
  height: 140px;
  object-fit: cover;
  flex-shrink: 0;
}

.card-info {
  padding: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  flex: 1;
}

.card-info h3 {
  font-size: 16px;
  margin: 0 0 8px 0;
  font-weight: 600;
}

.card-meta {
  font-size: 13px;
  color: #aaa;
  margin-bottom: 12px;
}

.card-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.btn-watch {
  padding: 7px 14px;
  background: #e50914;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 13px;
  font-weight: bold;
  transition: background 0.3s;
}

.btn-watch:hover {
  background: #b20710;
}

.btn-remove {
  padding: 7px 14px;
  background: #333;
  color: #aaa;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.3s;
}

.btn-remove:hover {
  background: #555;
  color: white;
}

.watchlist-actions {
  text-align: right;
  margin-top: 10px;
}

.btn-clear {
  padding: 10px 22px;
  background: transparent;
  color: #aaa;
  border: 1px solid #555;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.btn-clear:hover {
  background: #333;
  color: white;
  border-color: #e50914;
}

@media (max-width: 768px) {
  .watchlist-container {
    padding: 25px 20px;
  }

  .watchlist-container h1 {
    font-size: 24px;
  }

  .watchlist-grid {
    grid-template-columns: 1fr;
    gap: 15px;
  }

  .watchlist-actions {
    text-align: center;
  }
}

@media (max-width: 480px) {
  .watchlist-container {
    padding: 15px;
  }

  .watchlist-container h1 {
    font-size: 20px;
    margin-bottom: 20px;
  }

  .card-poster {
    width: 80px;
    height: 115px;
  }

  .card-info {
    padding: 10px;
  }

  .card-info h3 {
    font-size: 14px;
  }

  .card-actions {
    gap: 6px;
  }

  .btn-watch,
  .btn-remove {
    font-size: 12px;
    padding: 6px 10px;
  }
}
</style>