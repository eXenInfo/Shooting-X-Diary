<script setup>
import { ref, watch } from 'vue'

const vorlagen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-scheiben')) || [])
watch(vorlagen, (neueListe) => {
  localStorage.setItem('shooting-x-diary-scheiben', JSON.stringify(neueListe))
}, { deep: true })

const neueVorlageName = ref('')
const neueVorlageRinge = ref('')

function addVorlage() {
  if (!neueVorlageName.value || !neueVorlageRinge.value) return

  const neueId = Math.max(0, ...vorlagen.value.map(v => v.id)) + 1
  vorlagen.value.push({
    id: neueId,
    name: neueVorlageName.value,
    // Wir speichern die Ringe als Array von Strings
    ringe: neueVorlageRinge.value.split(',').map(r => r.trim())
  })
  neueVorlageName.value = ''
  neueVorlageRinge.value = ''
}

function deleteVorlage(idToDelete) {
    vorlagen.value = vorlagen.value.filter(v => v.id !== idToDelete)
}
</script>

<template>
  <div class="scheiben-verwaltung">
    <h2>Zielscheiben-Vorlagen</h2>
    <form @submit.prevent="addVorlage">
      <input type="text" v-model="neueVorlageName" placeholder="Name der Vorlage (z.B. DSB Präzision)">
      <input type="text" v-model="neueVorlageRinge" placeholder="Ringwerte, mit Komma getrennt (z.B. X,10,9,8)">
      <button type="submit">Vorlage hinzufügen</button>
    </form>
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Ringwerte</th>
          <th>Aktionen</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="vorlagen.length === 0">
          <td colspan="3">Noch keine Vorlagen erfasst.</td>
        </tr>
        <tr v-for="vorlage in vorlagen" :key="vorlage.id">
          <td>{{ vorlage.name }}</td>
          <td>{{ vorlage.ringe.join(', ') }}</td>
          <td>
              <button @click="deleteVorlage(vorlage.id)">Löschen</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
.scheiben-verwaltung {
  background-color: white;
  padding: 1rem;
  border-radius: 8px;
  margin-top: 1rem;
}
form { margin-bottom: 1rem; }
input { margin-right: 0.5rem; width: 300px; }
table { width: 100%; border-collapse: collapse; }
th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
button { cursor: pointer; margin-right: 5px; }
</style>