<template>
  <n-card title="Connexion" style="max-width: 400px; margin: auto;">
    <n-form @submit.prevent="handleLogin">
      <n-form-item label="Email">
        <n-input v-model="loginData.email" type="email" placeholder="Entrez votre email" />
      </n-form-item>
      <n-form-item label="Mot de passe">
        <n-input v-model="loginData.password" type="password" placeholder="Entrez votre mot de passe" />
      </n-form-item>
      <n-button type="primary" block @click="handleLogin">Se connecter</n-button>
    </n-form>
  </n-card>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useMessage } from 'naive-ui';

const router = useRouter();
const message = useMessage();

const loginData = ref({ email: '', password: '' });

async function handleLogin() {
  try {
    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/users/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(loginData.value)
    });
    const result = await response.json();
    if (response.ok) {
      localStorage.setItem('token', result.token);
      localStorage.setItem('userId', result.user.id);
      message.success('Connexion réussie !');
      router.push('/deck-builder');
    } else {
      message.error(result.message || 'Erreur de connexion');
    }
  } catch (error) {
    message.error('Problème de connexion au serveur');
  }
}
</script>
