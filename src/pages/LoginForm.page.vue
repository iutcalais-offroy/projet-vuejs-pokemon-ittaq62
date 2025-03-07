<template>
  <div class="login-container">
    <n-card title="Connexion">
      <n-form @submit.prevent="login">
        <n-form-item label="Email">
          <n-input v-model="loginData.email" type="email" required />
        </n-form-item>
        <n-form-item label="Mot de passe">
          <n-input v-model="loginData.password" type="password" required />
        </n-form-item>
        <n-button type="primary" native-type="submit" @click="login">Se connecter</n-button>
      </n-form>
    </n-card>

    <n-card title="Inscription">
      <n-form @submit.prevent="register">
        <n-form-item label="Email">
          <n-input v-model="registerData.email" type="email" required />
        </n-form-item>
        <n-form-item label="Mot de passe">
          <n-input v-model="registerData.password" type="password" required />
        </n-form-item>
        <n-button type="success" native-type="submit" @click="register">S'inscrire</n-button>
      </n-form>
    </n-card>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// Données pour le formulaire de connexion
const loginData = ref({
  email: '',
  password: ''
})

// Données pour le formulaire d'inscription
const registerData = ref({
  email: '',
  password: ''
})

// Fonction de connexion
const login = async () => {
  console.log("Tentative de connexion avec :", loginData.value)
  try {
    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/users/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginData.value)
    })
    
    console.log("Réponse reçue :", response)
    const data = await response.json()
    
    if (response.ok) {
      console.log("Connexion réussie :", data)
      localStorage.setItem('token', data.token) // Stocke le token
      localStorage.setItem('userId', data.user.id) // Stocke l'ID utilisateur
      router.push('/deck-builder') // Redirection après connexion réussie
    } else {
      console.error("Erreur de connexion :", data.message)
      alert(data.message || 'Erreur lors de la connexion')
    }
  } catch (error) {
    console.error("Erreur réseau :", error)
    alert('Erreur réseau')
  }
}

// Fonction d'inscription
const register = async () => {
  console.log("Tentative d'inscription avec :", registerData.value)
  try {
    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/users', { // Endpoint correct pour créer un utilisateur
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(registerData.value)
    })

    console.log("Réponse reçue :", response)
    const data = await response.json()

    if (response.ok) {
      console.log("Inscription réussie :", data)
      alert('Inscription réussie, vous pouvez vous connecter.')
      registerData.value = { email: '', password: '' } // Réinitialisation du formulaire
    } else {
      console.error("Erreur d'inscription :", data.message)
      alert(data.message || 'Erreur lors de l’inscription')
    }
  } catch (error) {
    console.error("Erreur réseau :", error)
    alert('Erreur réseau')
  }
}
</script>

<style scoped>
.login-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  max-width: 400px;
  margin: auto;
}
</style>
