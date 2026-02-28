<script setup>
import { ref } from 'vue'

const showLoginMenu = ref(false)

const menuLinks = [
  { id: 'destinations', name: 'Destinations' },
  { id: 'voyages', name: 'Mes Voyages' },
  { id: 'services', name: 'Services' }
]

const props = defineProps(['search', 'activePage']) // On ajoute activePage
const emit = defineEmits(['update:search', 'update:activePage'])


</script>

<template>
  <header class="h-20 bg-orange-200  px-8 flex items-center justify-between sticky top-0 z-100">
    
    
    <div class="flex items-center gap-12">
      <h1 class="text-2xl font-black text-orange-500 tracking-tighter uppercase">
        SkyBlue
      </h1>

      <nav class="hidden lg:flex items-center gap-8">
            <button 
            v-for="link in menuLinks" 
            :key="link.id"
            @click="$emit('update:activePage', link.id)" 
            :class="[
              'text-sm font-bold transition-all duration-300 pb-1 border-b-2',
              activePage === link.id ? 'text-orange-600 border-orange-500' : 'text-slate-600 border-transparent hover:text-orange-400'
            ]"
          >
      {{ link.name }}
    </button>
      </nav>
    </div>

    <div class="flex items-center gap-6">
      <div class="relative hidden sm:block">
      <input 
        type="text" 
        :value="search"
        @input="$emit('update:search', $event.target.value)"
        placeholder="Rechercher un site.." 
        class="bg-slate-100 border-none rounded-full py-2 px-5 text-sm focus:ring-2 focus:ring-orange-500/20 outline-none w-48 focus:w-64 transition-all"
      >
    </div>

      <div class="relative">
        <button 
          @click="showLoginMenu = !showLoginMenu"
          class="bg-orange-500 hover:bg-orange-600 text-white px-6 py-2.5  text-sm font-bold shadow-md  transition-all flex items-center gap-2 cursor-pointer"
        >
          Se connecter
          <span :class="{'rotate-180': showLoginMenu}" class="transition-transform duration-300 text-[10px]">▼</span>
        </button>

        <div v-if="showLoginMenu" 
             class="absolute right-0 mt-3 w-48 bg-white border border-slate-100 rounded-2xl shadow-xl overflow-hidden py-2 z-50">
          <button class="w-full text-left px-4 py-3 text-sm text-slate-700 hover:bg-orange-50 hover:text-orange-600 transition font-medium">
             Espace Voyageur
          </button>
          <div class="h-px bg-slate-50 my-1"></div>
          <button class="w-full text-left px-4 py-3 text-sm text-slate-700 hover:bg-orange-50 hover:text-orange-600 transition font-bold">
             Administration
          </button>
        </div>
      </div>
    </div>
  </header>
</template>