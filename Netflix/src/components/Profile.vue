<template>
  <div class="profile-content">
    <div class="profile-card">
      <div class="avatar">
        {{ user?.email?.[0]?.toUpperCase() || '👤' }}
      </div>
      <h2 class="user-email">{{ user?.email || 'Пользователь' }}</h2>
      <p class="member-since">Зарегистрирован: {{ joinDate }}</p>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  user: Object
})

const joinDate = computed(() => {
  if (!props.user?.metadata?.creationTime) return '—'
  return new Date(props.user.metadata.creationTime).toLocaleDateString('ru-RU')
})
</script>

<style scoped>
.profile-content {
  max-width: 600px;
  margin: 0 auto;
  padding: 50px 20px;
}

.profile-card {
  background: #1c1c1c;
  border-radius: 16px;
  padding: 40px;
  text-align: center;
}

.avatar {
  width: 90px;
  height: 90px;
  background: #e50914;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 36px;
  font-weight: bold;
  color: white;
  margin: 0 auto 20px;
}

.user-email {
  font-size: 20px;
  font-weight: 600;
  color: white;
  margin: 0 0 10px;
  word-break: break-all;
}

.member-since {
  font-size: 14px;
  color: #aaa;
  margin: 0;
}

@media (max-width: 480px) {
  .profile-content {
    padding: 30px 15px;
  }

  .profile-card {
    padding: 25px 20px;
  }

  .avatar {
    width: 70px;
    height: 70px;
    font-size: 28px;
  }

  .user-email {
    font-size: 16px;
  }
}
</style>
