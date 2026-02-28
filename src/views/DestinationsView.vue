<script setup>
import { ref, computed, onMounted } from 'vue'
import { db } from '../firebase.js' 
import { collection, onSnapshot } from 'firebase/firestore'
import ReservationSidebar from '../components/ReservationSidebar.vue'


const props = defineProps(['searchText'])
const sites = ref([])
const loading = ref(true)
const selectedCategory = ref('Tout')

const categories = [
  { id: 'all', name: 'Tout' },
  { id: 'lacs', name: 'Plage' },
  { id: 'nature', name: 'Nature' },
  { id: 'culture', name: 'Culture' }
]


const isSidebarOpen = ref(false)
const selectedSite = ref(null)

const handleOpen = (site) => {
  selectedSite.value = site
  isSidebarOpen.value = true
}


onMounted(() => {
  const colRef = collection(db, 'Destinations')
  
  onSnapshot(colRef, (snapshot) => {
    // On transforme les documents Firestore en objets JavaScript
    sites.value = snapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data()
    }))
    loading.value = false
  }, (err) => {
    console.error("Erreur Firebase:", err)
    loading.value = false
  })
})




const filteredSites = computed(() => {
  return sites.value.filter(site => {
    const matchCategory = selectedCategory.value === 'Tout' || site.Categorie === selectedCategory.value
    const searchLower = props.searchText.toLowerCase()
    const matchSearch = (site.Nom?.toLowerCase().includes(searchLower)) || 
                        (site.Province?.toLowerCase().includes(searchLower))
    return matchCategory && matchSearch
  })
})


</script>

<template>
  
  <div class="bg-slate-50 min-h-screen py-8 px-6">
    <div class="flex items-center gap-4 overflow-x-auto no-scrollbar pb-10">
        <button 
          v-for="cat in categories" 
          :key="cat.id"
          @click="selectedCategory = cat.name"
          :class="[
            'px-6 py-2.5 rounded-full font-bold text-sm transition-all duration-300 whitespace-nowrap',
            selectedCategory === cat.name 
              ? 'bg-orange-500 text-white shadow-md' 
              : 'bg-white text-slate-500 border border-slate-100 hover:border-orange-500'
          ]"
        >
          {{ cat.name }}
        </button>
      </div>

    <div class="max-w-7xl mx-auto">
      
      

      <div v-if="loading" class="grid grid-cols-1 md:grid-cols-3 gap-8">
        <div v-for="n in 3" :key="n" class="h-96 bg-slate-200 animate-pulse"></div>
      </div>

      <div v-else class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <div v-for="site in filteredSites" :key="site.id" 
             class="bg-olive-100  overflow-hidden shadow-xl hover:shadow-md transition-all duration-300 flex flex-col group">
          
          <div class="relative h-64 overflow-hidden">
            <img :src="site.Image_url" class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500">
            <div class="absolute top-4 left-4 bg-white/90 backdrop-blur-sm px-3 py-1 rounded-full text-[10px] font-black text-orange-600 uppercase">
              {{ site.Categorie }}
            </div>
          </div>

          <div class="p-6 flex flex-col grow">
            <div class="flex justify-between items-start mb-2">
              <h3 class="text-xl font-bold text-olive-800">{{ site.Nom }}</h3>
              <span class="text-orange-500 font-black">{{ site.Prix_usd?.toLocaleString() }}$ </span>
            </div>
            
            <p class="text-olive-700 text-sm mb-4 flex items-center gap-1">
               {{ site.Province }} • {{ site.Distance_km }}km • {{ site.Duree_estimee }}
            </p>

            <div class="mt-auto pt-4 border-t border-slate-50">
              <button 
                    @click="handleOpen(site)"
                    class="flex items-center gap-2 text-orange-600 font-bold text-sm group/btn cursor-pointer"
                  >
                    Réserver
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 transform group-hover/btn:translate-x-1 transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
                    </svg>
                    </button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="!loading && filteredSites.length === 0" class="text-center py-20">
        <p class="text-slate-400">Aucune destination trouvée dans cette catégorie.</p>
      </div>

    </div>
    <ReservationSidebar 
      :isOpen="isSidebarOpen" 
      :site="selectedSite" 
      
      @close="isSidebarOpen = false" 
    />
  </div>
</template>

<style scoped>
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
</style>
<style scoped>
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
</style>