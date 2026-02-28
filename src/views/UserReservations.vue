<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { db } from '../firebase.js' 
import { 
  collection, 
  query, 
  orderBy, 
  onSnapshot, 
  doc, 
  deleteDoc, 
  updateDoc 
} from 'firebase/firestore'

const reservations = ref([])
const loading = ref(true)

// Variable pour stocker la fonction de nettoyage
let unsubscribe = null

onMounted(() => {
  // 1. Référence à la collection (doit correspondre au nom dans ReservationSidebar)
  const resRef = collection(db, 'Reservations')
  
  // 2. Création d'une requête triée par date (la plus récente en haut)
  const q = query(resRef, orderBy('Date_creation', 'desc'))

  // 3. Écoute en temps réel avec onSnapshot
  unsubscribe = onSnapshot(q, (snapshot) => {
    reservations.value = snapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data()
    }))
    loading.value = false
  }, (error) => {
    console.error("Erreur de lecture :", error)
    loading.value = false
  })
})

// 4. Nettoyage du listener pour éviter les fuites de mémoire
onUnmounted(() => {
  if (unsubscribe) unsubscribe()
})

const formatDate = (timestamp) => {
  if (!timestamp) return '---'
  
  // Conversion de l'objet Firebase en objet Date JavaScript
  const date = timestamp.toDate() 
  
  // Formatage : 28 fév. 2026
  return date.toLocaleDateString('fr-FR', {
    day: 'numeric',
    month: 'short',
    year: 'numeric'
  })
}

const deleteReservation = async (id) => {
  if (confirm("Êtes-vous sûr de vouloir supprimer ce voyage ?")) {
    try {
      await deleteDoc(doc(db, "Reservations", id))
      // Pas besoin de rafraîchir manuellement, onSnapshot s'en occupe !
    } catch (error) {
      console.error("Erreur de suppression:", error)
    }
  }
}

const toggleStatus = async (reservation) => {
  const newStatut = reservation.Statut === 'en_attente' ? 'confirme' : 'en_attente'
  
  try {
    const docRef = doc(db, "Reservations", reservation.id)
    await updateDoc(docRef, {
      Statut: newStatut
    })
  } catch (error) {
    console.error("Erreur de mise à jour:", error)
  }
}
</script>

<template>
  <section class="p-8 max-w-6xl mx-auto">
    <div class="flex justify-between items-center mb-8">
      <h2 class="text-3xl font-black text-slate-800 uppercase tracking-tighter">
        Mes <span class="text-orange-500">Voyages</span>
      </h2>
      <span class="bg-slate-200 text-slate-600 px-4 py-1 rounded-full text-xs font-bold">
        {{ reservations.length }} Réservation(s)
      </span>
    </div>

    <div class="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden">
      <table class="w-full text-left border-collapse">
        <thead>
          <tr class="bg-slate-50 border-b border-slate-100">
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Destination</th>
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Service</th>
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Date</th>
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Heure</th>
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Prix</th>
            <th class="px-6 py-4 text-[10px] font-black text-slate-400 uppercase tracking-widest">Statut</th>
          </tr>
        </thead>
        
        <tbody class="divide-y divide-slate-50">
          <tr v-for="res in reservations" :key="res.id" class="hover:bg-orange-50/30 transition-colors group">
            <td class="px-6 py-4">
              <span class="font-bold text-slate-800 group-hover:text-orange-600 transition-colors">
                {{ res.Destination_nom }}
              </span>
            </td>
            <td class="px-6 py-4 text-sm text-slate-500 font-medium">
              {{ res.Service }}
            </td>
            <td class="px-6 py-4 text-sm text-slate-500">
              {{ formatDate(res.Date_creation) }}
            </td>
            <td class="px-6 py-4 text-sm text-slate-500">
              {{ res.HeureRendezVous }}
            </td>
            <td class="px-6 py-4">
              <span class="font-black text-slate-800">{{ res.Prix }}$</span>
            </td>
            <td class="px-6 py-4">
              <span :class="[
                'px-3 py-1 rounded-full text-[10px] font-black uppercase tracking-tighter',
                res.Statut === 'en_attente' ? 'bg-orange-100 text-orange-600' : 'bg-green-100 text-green-600'
              ]">
                {{ res.Statut?.replace('_', ' ') }}
              </span>
            </td>
            <td class="px-6 py-4 text-right flex justify-end gap-3">
    <button 
      @click="toggleStatus(res)"
      class="p-2 hover:bg-orange-100 text-slate-400 hover:text-orange-600 rounded-xl transition-all"
      title="Changer le statut"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
      </svg>
    </button>

    <button 
      @click="deleteReservation(res.id)"
      class="p-2 hover:bg-red-50 text-slate-400 hover:text-red-500 rounded-xl transition-all"
      title="Supprimer"
    >
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
      </svg>
    </button>
  </td>
          </tr>
        </tbody>
      </table>

      <div v-if="reservations.length === 0 && !loading" class="py-12 text-center text-slate-400 italic">
        Aucun voyage trouvé dans votre historique.
      </div>
    </div>
  </section>
</template>