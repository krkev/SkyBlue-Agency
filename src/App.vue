<script setup>
import { ref } from 'vue'
import Header from './components/Header.vue'
import DestinationsView from './views/DestinationsView.vue'
import UserReservations from './views/UserReservations.vue' // Import de la nouvelle vue

const searchQuery = ref('')
const currentView = ref('destinations') // L'état qui contrôle la vue active

</script>

<template>
  <div class="min-h-screen ">
    <Header v-model:search="searchQuery" v-model:activePage="currentView" />
    
    <main class="py-8">
      <DestinationsView 
        v-if="currentView === 'destinations'" 
        :search-text="searchQuery" 
      />

      <UserReservations 
        v-if="currentView === 'voyages'" 
      />

      <div v-if="currentView === 'services'" class="text-center py-20">
        <h2 class="text-2xl font-bold text-slate-600">Page Services en cours de développement...</h2>
      </div>
    </main>
  </div>
</template>