<script setup>
import { ref } from 'vue'
import { db } from '../firebase.js' 
import { collection, addDoc, serverTimestamp } from 'firebase/firestore'

const isSubmitting = ref(false)


const props = defineProps({
  isOpen: Boolean,
  site: Object
})
const emit = defineEmits(['close'])





const reservationData = ref({
  service: 'Visite guidée',
  heure: '09:00'
})

const services = ['Visite guidée & Interprétariat', 'Transport VIP', 'Pack Photo', 'Hébergement']

const confirmReservation = async () => {
  // 1. Sécurité : on vérifie si un site est bien sélectionné
  if (!props.site) return
  
  isSubmitting.value = true
  
  try {
    // 2. Préparation du "paquet" de données
    const reservationBrute = {
      Destination_nom: props.site.Nom,
      Prix: props.site.Prix_usd,
      Service: reservationData.value.service,
      HeureRendezVous: reservationData.value.heure,
      Statut: 'en_attente',
      Date_creation: serverTimestamp() // Heure du serveur Firebase
    }

    // 3. Envoi vers la collection "reservations"
    // Si la collection n'existe pas, Firebase la créera automatiquement !
    const docRef = await addDoc(collection(db, 'Reservations'), reservationBrute)
    
    console.log("Réservation réussie avec l'ID : ", docRef.id)
    
    // 4. Feedback utilisateur et fermeture
    alert(`Votre réservation pour ${props.site.Nom} a été enregistrée avec succès !`)
    emit('close')

  } catch (error) {
    console.error("Erreur Firestore :", error)
    alert("Désolé, une erreur est survenue lors de la réservation.")
  } finally {
    isSubmitting.value = false
  }
}
</script>

<template>
  <Transition name="slide">
    <div v-if="isOpen" class="fixed inset-0 z-100 flex justify-end">
      <div @click="emit('close')" class="absolute inset-0 bg-black/40 backdrop-blur-sm"></div>

      <div class="relative w-full max-w-md bg-white h-full shadow-2xl p-8 flex flex-col">
            <button 
        @click="$emit('close')" 
        class="absolute top-6 right-6 z-60 p-2 bg-slate-100 hover:bg-orange-100 text-slate-500 hover:text-orange-600 rounded-full transition-all duration-300 shadow-sm"
        aria-label="Fermer"
    >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
        </svg>
    </button>
        
        <div v-if="site" class="mt-6 grow overflow-y-auto no-scrollbar">
          <h2 class="text-2xl font-black text-slate-800 uppercase tracking-tighter">Réserver</h2>
          <p class="text-orange-500 font-bold mb-6">{{ site.Nom }}</p>
          
          <div class="aspect-video rounded-3xl overflow-hidden mb-8 shadow-inner bg-slate-100">
            <img :src="site.Image_url" class="w-full h-full object-cover">
          </div>

          <form class="space-y-8">
            <div>
              <label class="block text-[10px] font-black text-slate-600 uppercase tracking-widest mb-4">Choisir un Service</label>
              <div class="grid grid-cols-2 gap-3">
                <button v-for="s in services" :key="s" type="button"
                  @click="reservationData.service = s"
                  :class="['py-3 rounded-2xl text-xs font-bold border transition-all', 
                  reservationData.service === s ? 'border-orange-500 bg-orange-50 text-orange-600' : 'border-slate-100 text-slate-700']">
                  {{ s }}
                </button>
              </div>
            </div>

            <div>
              <label class="block text-[10px] font-black text-slate-600 uppercase tracking-widest mb-4">Heure d'arrivée</label>
              <input type="time" v-model="reservationData.heure" 
                class="w-full bg-slate-50  rounded-2xl py-4 px-6 text-slate-800 font-bold  focus:ring-2 focus:ring-orange-500/20 border-2 border-orange-500  transition-all">
            </div>
          </form>
        </div>

        <div class="pt-6 mt-auto border-t border-slate-50">
          <button class="w-full bg-orange-500 text-white py-5  font-black uppercase tracking-widest hover:bg-orange-700 transition-colors shadow-xl shadow-slate-200 cursor-pointer"
          :disabled="isSubmitting"
          @click="confirmReservation">
            Confirmer • {{ site?.Prix_usd?.toLocaleString() }}$ 
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.slide-enter-active, .slide-leave-active { transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1); }
.slide-enter-from, .slide-leave-to { transform: translateX(100%); opacity: 0; }
.no-scrollbar::-webkit-scrollbar { display: none; }
</style>