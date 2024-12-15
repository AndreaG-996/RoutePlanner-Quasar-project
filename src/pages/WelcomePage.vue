<template>
  <div class="welcome-container" @click="goToMainPage">
    <h1 class="welcome-title">Plan Your Route</h1>
    <p class="welcome-subtitle">Tab anywhere to continue</p>
  </div>
</template>

<script>
export default {
  name: 'WelcomePage',
  methods: {
    goToMainPage() {
      console.log('Clicked to navigate')
      // from HomePage to MainPage
      this.$router.push('/main')
    },
  },

  mounted() {
    console.log('Welcome page loaded')
    setTimeout(() => {
      console.log('Redirecting to main page...')
      this.$router.push('/main')
    }, 5000) // 5秒后跳转
  },
}
</script>

<style scoped>
.welcome-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #2e4d34;
  color: #c4ddc3;
  cursor: pointer;
  animation: fadeIn 1s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.welcome-title {
  font-size: 3rem;
  font-family: 'Arial', sans-serif; /* Change the design of font */
  font-weight: bold;
  color: #c4ddc3;
}

.welcome-subtitle {
  font-size: 1.2rem;
  margin-top: 1rem;
  color: #87a08b;
}
</style>
