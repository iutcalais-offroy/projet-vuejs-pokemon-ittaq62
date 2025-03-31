<template>
  <div class="deck-builder">
    <!-- Section du deck -->
    <div class="deck">
      <h3>Deck ({{ deck.length }} cartes)</h3>
      <div class="deck-controls">
        <n-input
          :value="deckName"
          placeholder="Nom du deck..."
          class="deck-name-input"
          @update:value="updateDeckName"
        />
        <n-button type="success" @click="saveDeck">Sauvegarder</n-button>
      </div>
      <div class="deck-container">
        <n-card
          v-for="card in deck"
          :key="card.id"
          class="pokemon-card"
          @click="removeFromDeck(card)"
        >
          <img :src="card.imageUrl" alt="Image Pokémon" class="card-image"/>
          <div class="pokemon-info">
            <h3 class="pokemon-name">{{ card.name }}</h3>
            <span class="pokemon-hp">PV {{ card.lifePoints }}</span>
          </div>
          <n-tag :class="'type-' + card.type.toLowerCase()">{{ card.type }}</n-tag>
          <p class="pokemon-details">Taille : {{ card.height }}m | Poids : {{ card.weight }}kg</p>
          <p class="pokemon-attack"><strong>{{ card.attack }}</strong> <span class="attack-damage">{{ card.attackDamage }} PV</span></p>
        </n-card>
      </div>
    </div>

    <h2>Liste des cartes Pokémon</h2>

    <!-- Barre de recherche -->
    <n-input
      v-model="searchQuery"
      placeholder="Rechercher une carte..."
      clearable
      class="search-bar"
      @update:value="updateSearchQuery"
    />

    <!-- Collection de cartes Pokémon -->
    <div class="card-container">
      <n-card
        v-for="card in filteredPokemons"
        :key="card.id"
        class="pokemon-card"
        @click="addToDeck(card)"
      >
        <img :src="card.imageUrl" alt="Image Pokémon" class="card-image"/>
        <div class="pokemon-info">
          <h3 class="pokemon-name">{{ card.name }}</h3>
          <span class="pokemon-hp">PV {{ card.lifePoints }}</span>
        </div>
        <n-tag :class="'type-' + card.type.toLowerCase()">{{ card.type }}</n-tag>
        <p class="pokemon-details">Taille : {{ card.height }}m | Poids : {{ card.weight }}kg</p>
        <p class="pokemon-attack"><strong>{{ card.attack }}</strong> <span class="attack-damage">{{ card.attackDamage }} PV</span></p>
      </n-card>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const pokemons = ref([]) // Liste complète des cartes Pokémon
const searchQuery = ref('') // Texte de recherche
const deck = ref([]) // Deck temporaire
const deckName = ref('') // Nom du deck
const router = useRouter()

// Mise à jour de la recherche
const updateSearchQuery = (newValue) => {
  searchQuery.value = newValue;
};

// Mise à jour du nom du deck
const updateDeckName = (newValue) => {
  deckName.value = newValue;
  console.log("Mise à jour du nom du deck:", deckName.value);
};

// Récupération des cartes Pokémon
const fetchPokemons = async () => {
  try {
    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/pokemon-cards')
    const data = await response.json()

    pokemons.value = data.map(card => ({
      id: card.id,
      name: card.name ? card.name.toLowerCase() : 'inconnu',
      lifePoints: card.lifePoints || 0,
      type: card.type ? card.type.name : 'Inconnu',
      attack: card.attack ? card.attack.name : 'Aucune',
      attackDamage: card.attack ? card.attack.damages : 0,
      height: card.height || 0,
      weight: card.weight || 0,
      imageUrl: card.imageUrl || 'https://via.placeholder.com/150'
    }));
  } catch (error) {
    console.error("Erreur lors du chargement des cartes Pokémon :", error);
  }
};

// Filtrage des Pokémon avec `computed`
const filteredPokemons = computed(() => {
  if (!searchQuery.value || searchQuery.value.trim() === '') {
    return pokemons.value;
  }
  return pokemons.value.filter(card => card.name.includes(searchQuery.value.toLowerCase()));
});

// Ajouter une carte au deck
const addToDeck = (card) => {
  if (!deck.value.find(p => p.id === card.id)) {
    deck.value.push(card);
  }
};

// Retirer une carte du deck
const removeFromDeck = (card) => {
  deck.value = deck.value.filter(p => p.id !== card.id);
};

// Sauvegarde du deck via API
const saveDeck = async () => {
  console.log("Nom du deck:", deckName.value);
  console.log("Cartes dans le deck:", deck.value);

  if (!deckName.value || deckName.value.trim() === '') {
    alert("Veuillez donner un nom à votre deck.");
    return;
  }
  if (deck.value.length === 0) {
    alert("Votre deck est vide.");
    return;
  }

  const token = localStorage.getItem('token');
  if (!token) {
    alert("Erreur : Vous devez être connecté pour sauvegarder un deck.");
    return;
  }

  const deckData = {
    name: deckName.value,
    cards: deck.value.map(card => card.id)
  };

  try {
    console.log("Envoi des données à l'API :", deckData);

    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/decks', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify(deckData)
    });

    const responseData = await response.json();
    console.log("Réponse API :", responseData);

    if (response.ok) {
      alert("Deck sauvegardé avec succès !");
      deck.value = [];
      deckName.value = '';
      router.push('/mes-decks'); // Redirection vers la page des decks
    } else {
      alert(responseData.message || "Erreur lors de la sauvegarde du deck.");
    }
  } catch (error) {
    console.error("Erreur réseau lors de la sauvegarde :", error);
    alert("Erreur réseau lors de la sauvegarde.");
  }
};

// Charger les cartes Pokémon au montage du composant
onMounted(fetchPokemons);
</script>

<style scoped>
.deck-builder {
  max-width: 1100px;
  margin: auto;
  text-align: center;
}

/* Deck en haut */
.deck {
  margin-bottom: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 10px;
}

/* Barre de recherche */
.search-bar {
  width: 300px;
  margin-bottom: 20px;
}

/* Conteneur des cartes */
.card-container, .deck-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}

/* Cartes du deck identiques aux cartes de la collection */
.pokemon-card {
  width: 250px;
  text-align: center;
  border-radius: 10px;
  padding: 15px;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
}

/* Image du Pokémon */
.card-image {
  width: 100%;
  height: auto;
  max-height: 150px;
}

/* Type de Pokémon (couleurs restaurées) */
.type-grass { background-color: #78c850; color: white; }
.type-fire { background-color: #f08030; color: white; }
.type-water { background-color: #6890f0; color: white; }
.type-electric { background-color: #f8d030; color: black; }
.type-normal { background-color: #a8a878; color: white; }
.type-bug { background-color: #a8b820; color: white; }
.type-poison { background-color: #a040a0; color: white; }
.type-ground { background-color: #e0c068; color: black; }
.type-flying { background-color: #a890f0; color: black; }
.type-psychic { background-color: #f85888; color: white; }
.type-rock { background-color: #b8a038; color: white; }
.type-ghost { background-color: #705898; color: white; }
.type-dark { background-color: #705848; color: white; }
.type-dragon { background-color: #7038f8; color: white; }
.type-steel { background-color: #b8b8d0; color: black; }
.type-fairy { background-color: #ee99ac; color: black; }

/* Nom du deck + bouton */
.deck-controls {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-bottom: 15px;
}

/* Input pour le nom du deck */
.deck-name-input {
  width: 200px;
}
</style>
