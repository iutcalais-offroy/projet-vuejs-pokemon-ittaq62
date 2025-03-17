<template>
  <div class="deck-builder">
    <h2>Liste des cartes Pokémon</h2>

    <!-- Barre de recherche corrigée -->
    <n-input
      :value="searchQuery"
      placeholder="Rechercher un Pokémon..."
      clearable
      class="search-bar"
      @update:value="updateSearchQuery"
    />

    <div class="card-container">
      <n-card
        v-for="card in filteredPokemons"
        :key="card.id"
        class="pokemon-card"
      >
        <!-- Image du Pokémon -->
        <img :src="card.imageUrl" alt="Image Pokémon" class="card-image"/>

        <!-- Nom et PV -->
        <div class="pokemon-info">
          <h3 class="pokemon-name">{{ card.name }}</h3>
          <span class="pokemon-hp">PV {{ card.lifePoints }}</span>
        </div>

        <!-- Type du Pokémon -->
        <n-tag :class="'type-' + card.type.toLowerCase()">{{ card.type }}</n-tag>

        <!-- Taille & Poids -->
        <p class="pokemon-details">
          Taille : {{ card.height }}m | Poids : {{ card.weight }}kg
        </p>

        <!-- Attaque -->
        <p class="pokemon-attack">
          <strong>{{ card.attack }}</strong> <span class="attack-damage">{{ card.attackDamage }} PV</span>
        </p>
      </n-card>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const pokemons = ref([]) // Liste complète des cartes Pokémon
const searchQuery = ref('') // Texte de recherche

// Correction : updateSearchQuery() avec la bonne syntaxe pour NaiveUI
const updateSearchQuery = (newValue) => {
  searchQuery.value = newValue;
  console.log("Mise à jour de searchQuery :", searchQuery.value);
};

// Récupération des cartes Pokémon depuis l’API
const fetchPokemons = async () => {
  try {
    const response = await fetch('https://pokemon-api-seyrinian-production.up.railway.app/pokemon-cards')
    const data = await response.json()
    
    console.log("Données reçues :", data)

    // Traitement des données pour éviter les erreurs
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

    console.log("Liste des Pokémon après traitement :", pokemons.value);
  } catch (error) {
    console.error("Erreur lors du chargement des cartes Pokémon :", error);
  }
};

// Filtrage avec `computed()`
const filteredPokemons = computed(() => {
  console.log("Recherche actuelle :", searchQuery.value);

  if (!searchQuery.value || searchQuery.value.trim() === '') {
    console.log("Aucune recherche active, affichage de tous les Pokémon");
    return pokemons.value; // Si la recherche est vide, afficher tous les Pokémon
  }

  console.log("Filtrage avec :", searchQuery.value.toLowerCase());

  return pokemons.value.filter(card => {
    console.log(`Comparaison : ${card.name} avec ${searchQuery.value.toLowerCase()}`);
    return card.name.includes(searchQuery.value.toLowerCase()); // Filtrage insensible à la casse
  });
});

// Charger les cartes au montage du composant
onMounted(fetchPokemons);
</script>

<style scoped>
/* Conteneur principal */
.deck-builder {
  max-width: 1100px;
  margin: auto;
  text-align: center;
}

/* Barre de recherche */
.search-bar {
  width: 300px;
  margin-bottom: 20px;
}

/* Conteneur des cartes */
.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}

/* Style de chaque carte */
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

/* Informations du Pokémon */
.pokemon-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 18px;
  font-weight: bold;
  margin-top: 10px;
}

/* Style des PV */
.pokemon-hp {
  color: red;
  font-weight: bold;
}

/* Type de Pokémon */
.type-grass { background-color: #78c850; color: white; }
.type-fire { background-color: #f08030; color: white; }
.type-water { background-color: #6890f0; color: white; }
.type-electric { background-color: #f8d030; color: black; }
.type-normal { background-color: #a8a878; color: white; }
.type-bug { background-color: #ab2; color: white; }
.type-poison { background-color: rgb(138, 44, 130); color: white; }
.type-ground { background-color: #db5; color: white; }


/* Détails du Pokémon */
.pokemon-details {
  font-size: 14px;
  color: gray;
}

/* Attaque */
.pokemon-attack {
  font-size: 16px;
  font-weight: bold;
  margin-top: 10px;
}

/* Dégâts de l'attaque */
.attack-damage {
  color: red;
  font-weight: bold;
}
</style>
