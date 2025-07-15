<script setup>
import { ref, watch } from 'vue'

// --- Bestehende Logik ---
const waffen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-waffen')) || [])

watch(waffen, (neueListe) => {
  localStorage.setItem('shooting-x-diary-waffen', JSON.stringify(neueListe))
}, { deep: true })

const neueWaffeName = ref('')
const neueWaffeKaliber = ref('')

function addWaffe() {
  if (!neueWaffeName.value) return
  const neueId = Math.max(0, ...waffen.value.map(w => w.id)) + 1
  waffen.value.push({ id: neueId, name: neueWaffeName.value, kaliber: neueWaffeKaliber.value })
  neueWaffeName.value = ''
  neueWaffeKaliber.value = ''
}

function deleteWaffe(idToDelete) {
  waffen.value = waffen.value.filter(w => w.id !== idToDelete)
}

// --- NEU: Logik für den Bearbeitungsmodus ---
const waffeInBearbeitungId = ref(null) // Speichert die ID der Waffe, die gerade bearbeitet wird
const editName = ref('') // Temporärer Speicher für den Namen im Bearbeitungsmodus
const editKaliber = ref('') // Temporärer Speicher für das Kaliber im Bearbeitungsmodus

function startEditing(waffe) {
  // Setzt den Bearbeitungsmodus für die angeklickte Waffe
  waffeInBearbeitungId.value = waffe.id
  // Kopiert die aktuellen Daten in die temporären Variablen
  editName.value = waffe.name
  editKaliber.value = waffe.kaliber
}

function saveEdit() {
  if (!waffeInBearbeitungId.value) return

  // Finde die Waffe in der Liste
  const waffe = waffen.value.find(w => w.id === waffeInBearbeitungId.value)
  if (waffe) {
    // Aktualisiere die Daten
    waffe.name = editName.value
    waffe.kaliber = editKaliber.value
  }
  // Beende den Bearbeitungsmodus
  cancelEdit()
}

function cancelEdit() {
  // Setzt den Bearbeitungsmodus zurück
  waffeInBearbeitungId.value = null
}
</script>

<template>
  <div class="waffen-verwaltung">
    <h2>Waffen-Verwaltung</h2>
    
    <form @submit.prevent="addWaffe">
      <input type="text" v-model="neueWaffeName" placeholder="Name der Waffe">
      <input type="text" v-model="neueWaffeKaliber" placeholder="Kaliber">
      <button type="submit">Hinzufügen</button>
    </form>

    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Kaliber</th>
          <th>Aktionen</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="waffen.length === 0">
          <td colspan="3">Noch keine Waffen erfasst.</td>
        </tr>
        <tr v-for="waffe in waffen" :key="waffe.id">
          
          <template v-if="waffeInBearbeitungId !== waffe.id">
            <td>{{ waffe.name }}</td>
            <td>{{ waffe.kaliber }}</td>
            <td>
              <button @click="startEditing(waffe)">Bearbeiten</button>
              <button @click="deleteWaffe(waffe.id)">Löschen</button>
            </td>
          </template>

          <template v-else>
            <td><input type="text" v-model="editName"></td>
            <td><input type="text" v-model="editKaliber"></td>
            <td>
              <button @click="saveEdit()">Speichern</button>
              <button @click="cancelEdit()">Abbrechen</button>
            </td>
          </template>

        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
/* Styles bleiben fast unverändert, nur ein kleiner Abstand für die Buttons */
button {
  cursor: pointer;
  margin-right: 5px; /* Fügt kleinen Abstand zwischen Buttons hinzu */
}
/* restliche Styles */
.waffen-verwaltung { background-color: white; padding: 1rem; border-radius: 8px; }
form { margin-bottom: 1rem; }
input { margin-right: 0.5rem; }
table { width: 100%; border-collapse: collapse; }
th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
</style>