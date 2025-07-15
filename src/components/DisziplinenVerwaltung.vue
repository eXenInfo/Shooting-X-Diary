<script setup>
import { ref, watch } from 'vue'

const disziplinen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-disziplinen')) || [])
const vorlagen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-scheiben')) || [])

watch(disziplinen, (neueListe) => {
  localStorage.setItem('shooting-x-diary-disziplinen', JSON.stringify(neueListe))
}, { deep: true })

// --- Formular-State ---
const disziplinInBearbeitungId = ref(null)
const neueDisziplinName = ref('')
const neueDisziplinVorlageId = ref(null)
const neueDisziplinStages = ref([]) // Wird jetzt eine Liste von Objekten
const neueStageName = ref('')
const neueStageSchusszahl = ref(null) // NEU: für die Schusszahl

// --- Stage-Verwaltung ---
function addStage() {
  if (!neueStageName.value || !neueStageSchusszahl.value) {
      alert('Bitte Namen und Schusszahl für die Stage angeben.')
      return
  }
  // NEU: Wir fügen ein Objekt hinzu
  neueDisziplinStages.value.push({
      name: neueStageName.value,
      schusszahl: neueStageSchusszahl.value
  })
  neueStageName.value = ''
  neueStageSchusszahl.value = null
}
function removeStage(index) {
  neueDisziplinStages.value.splice(index, 1)
}

// --- Haupt-Speicherfunktion ---
function saveDisziplin() {
  if (!neueDisziplinName.value || !neueDisziplinVorlageId.value) {
    alert('Bitte Namen und eine Vorlage auswählen!')
    return
  }
  if (disziplinInBearbeitungId.value !== null) {
    const disziplin = disziplinen.value.find(d => d.id === disziplinInBearbeitungId.value)
    if (disziplin) {
      disziplin.name = neueDisziplinName.value
      disziplin.vorlageId = neueDisziplinVorlageId.value
      disziplin.stages = neueDisziplinStages.value
    }
  } else {
    const neueId = Math.max(0, ...disziplinen.value.map(d => d.id)) + 1
    disziplinen.value.push({
      id: neueId,
      name: neueDisziplinName.value,
      vorlageId: neueDisziplinVorlageId.value,
      stages: neueDisziplinStages.value
    })
  }
  resetForm();
}

// --- Bearbeiten, Löschen, Abbrechen ---
function deleteDisziplin(idToDelete) {
    if (confirm('Soll diese Disziplin wirklich gelöscht werden?')) {
        disziplinen.value = disziplinen.value.filter(d => d.id !== idToDelete)
    }
}
function startEditing(disziplin) {
  disziplinInBearbeitungId.value = disziplin.id
  neueDisziplinName.value = disziplin.name
  neueDisziplinVorlageId.value = disziplin.vorlageId
  neueDisziplinStages.value = [...disziplin.stages] 
}
function resetForm() {
  disziplinInBearbeitungId.value = null
  neueDisziplinName.value = ''
  neueDisziplinVorlageId.value = null
  neueDisziplinStages.value = []
  neueStageName.value = ''
  neueStageSchusszahl.value = null
}

// --- Hilfsfunktion ---
function getVorlageName(vorlageId) {
    const vorlage = vorlagen.value.find(v => v.id === vorlageId)
    return vorlage ? vorlage.name : 'Keine'
}
</script>

<template>
  <div class="disziplinen-verwaltung">
    <h2>{{ disziplinInBearbeitungId === null ? 'Neue Disziplin anlegen' : 'Disziplin bearbeiten' }}</h2>
    <form @submit.prevent="saveDisziplin" class="add-form">
      <div><label>Name der Disziplin</label><input type="text" v-model="neueDisziplinName" placeholder="Name"></div>
      <div><label>Zielscheiben-Vorlage</label><select v-model="neueDisziplinVorlageId"><option :value="null" disabled>Vorlage auswählen</option><option v-for="vorlage in vorlagen" :key="vorlage.id" :value="vorlage.id">{{ vorlage.name }}</option></select></div>
      
      <div>
        <label>Standard-Stages</label>
        <div class="stage-adder">
          <input type="text" v-model="neueStageName" placeholder="Stage-Name (z.B. Scheibe 1)">
          <input type="number" v-model.number="neueStageSchusszahl" placeholder="Anzahl Schuss" class="schusszahl-input">
          <button type="button" @click="addStage">Stage +</button>
        </div>
        <ul>
          <li v-for="(stage, index) in neueDisziplinStages" :key="index">
            {{ stage.name }} ({{ stage.schusszahl }} Schuss) <span @click="removeStage(index)" class="remove-stage">x</span>
          </li>
        </ul>
      </div>
      
      <div class="form-actions">
        <button type="submit">{{ disziplinInBearbeitungId === null ? 'Disziplin hinzufügen' : 'Änderungen speichern' }}</button>
        <button type="button" v-if="disziplinInBearbeitungId !== null" @click="resetForm">Abbrechen</button>
      </div>
    </form>
    <hr>
    <h3>Bestehende Disziplinen</h3>
    <table>
      <thead>
        <tr><th>Name</th><th>Scheibe</th><th>Stages</th><th>Aktionen</th></tr>
      </thead>
      <tbody>
        <tr v-for="disziplin in disziplinen" :key="disziplin.id">
          <td>{{ disziplin.name }}</td>
          <td>{{ getVorlageName(disziplin.vorlageId) }}</td>
          <td>
            <ul class="stage-list-in-table">
                <li v-for="stage in disziplin.stages" :key="stage.name">
                    {{ stage.name }} ({{ stage.schusszahl }} Schuss)
                </li>
            </ul>
          </td>
          <td>
            <button @click="startEditing(disziplin)">Bearbeiten</button>
            <button @click="deleteDisziplin(disziplin.id)">Löschen</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
/* Umfassendere Styles für das neue Formular */
.disziplinen-verwaltung { background-color: white; padding: 1rem; border-radius: 8px; margin-top: 1rem; }
.add-form { border: 1px solid #eee; padding: 1rem; margin-bottom: 1rem; border-radius: 5px; }
.add-form > div:not(:last-child) { margin-bottom: 1rem; }
label { display: block; margin-bottom: 0.5rem; font-weight: bold; }
input, select { width: 100%; padding: 8px; box-sizing: border-box; }
.stage-adder { display: flex; gap: 10px; }
.stage-adder input[type="text"] { flex-grow: 1; }
.schusszahl-input { width: 120px; }
ul { list-style: square; padding-left: 20px; margin-top: 0.5rem; }
.remove-stage { cursor: pointer; color: red; font-weight: bold; margin-left: 10px; }
.form-actions button { margin-right: 10px; }
table { width: 100%; border-collapse: collapse; }
th, td { border: 1px solid #ddd; padding: 8px; text-align: left; vertical-align: top; }
.stage-list-in-table { padding: 0; margin: 0; list-style: none; }
</style>