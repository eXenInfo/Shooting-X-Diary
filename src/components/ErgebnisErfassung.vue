<script setup>
import { ref, watch, computed } from 'vue'

// --- Stammdaten & Ergebnisse laden ---
const waffen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-waffen')) || [])
const disziplinen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-disziplinen')) || [])
const vorlagen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-scheiben')) || [])
const ergebnisse = ref(JSON.parse(localStorage.getItem('shooting-x-diary-ergebnisse')) || [])

watch(ergebnisse, (neueListe) => {
  localStorage.setItem('shooting-x-diary-ergebnisse', JSON.stringify(neueListe))
}, { deep: true })

// --- Formular-State ---
const ergebnisInBearbeitungId = ref(null)
const gewaehlteWaffeId = ref(null)
const gewaehlteDisziplinId = ref(null)
const datum = ref(new Date().toISOString().split('T')[0])
const notizen = ref('')
const scores = ref({})

// --- Computed Properties für dynamische Logik ---
const gewaehlteDisziplin = computed(() => disziplinen.value.find(d => d.id === gewaehlteDisziplinId.value))
const zielscheibe = computed(() => {
  if (!gewaehlteDisziplin.value) return null
  return vorlagen.value.find(v => v.id === gewaehlteDisziplin.value.vorlageId)
})

// --- Beobachter, der das Formular bei Disziplin-Wechsel vorbereitet ---
watch(gewaehlteDisziplinId, (neuerWert, alterWert) => {
    // Nur zurücksetzen, wenn es eine echte Änderung ist und wir nicht im Edit-Modus sind
    if (neuerWert !== alterWert && ergebnisInBearbeitungId.value === null) {
        resetScores();
    }
})

function resetScores() {
    scores.value = {}
    if (gewaehlteDisziplin.value && gewaehlteDisziplin.value.stages) {
        gewaehlteDisziplin.value.stages.forEach(stage => {
            scores.value[stage.name] = {} // Wichtig: stage.name als Schlüssel verwenden
        })
    }
}

// --- Haupt-Speicherfunktion ---
function saveErgebnis() {
  if (!gewaehlteWaffeId.value || !gewaehlteDisziplinId.value) {
    alert('Bitte Waffe und Disziplin auswählen!')
    return
  }

  // BEARBEITEN
  if (ergebnisInBearbeitungId.value !== null) {
    const ergebnis = ergebnisse.value.find(e => e.id === ergebnisInBearbeitungId.value)
    if (ergebnis) {
      ergebnis.datum = datum.value
      ergebnis.waffenId = gewaehlteWaffeId.value
      ergebnis.disziplinId = gewaehlteDisziplinId.value
      ergebnis.notizen = notizen.value
      ergebnis.scores = scores.value
    }
  } 
  // HINZUFÜGEN
  else {
    const neueId = Math.max(0, ...ergebnisse.value.map(e => e.id)) + 1
    ergebnisse.value.unshift({
      id: neueId,
      datum: datum.value,
      waffenId: gewaehlteWaffeId.value,
      disziplinId: gewaehlteDisziplinId.value,
      notizen: notizen.value,
      scores: scores.value
    })
  }
  resetForm();
}

// --- Bearbeiten, Löschen, Abbrechen für Ergebnisse ---
function startEditing(ergebnis) {
  ergebnisInBearbeitungId.value = ergebnis.id
  datum.value = ergebnis.datum
  gewaehlteWaffeId.value = ergebnis.waffenId
  gewaehlteDisziplinId.value = ergebnis.disziplinId
  notizen.value = ergebnis.notizen
  // Wichtig: Tiefe Kopie der Scores, damit Änderungen nicht sofort live sind
  scores.value = JSON.parse(JSON.stringify(ergebnis.scores))
}

function deleteErgebnis(idToDelete) {
    if (confirm('Soll dieses Ergebnis wirklich gelöscht werden?')) {
        ergebnisse.value = ergebnisse.value.filter(e => e.id !== idToDelete)
    }
}

function resetForm() {
  ergebnisInBearbeitungId.value = null
  gewaehlteWaffeId.value = null
  gewaehlteDisziplinId.value = null
  datum.value = new Date().toISOString().split('T')[0]
  notizen.value = ''
  scores.value = {}
}

// --- Hilfsfunktionen ---
function getNameById(liste, id) {
  const item = liste.find(i => i.id === id);
  return item ? item.name : 'Unbekannt';
}

function getGesamtScore(scoresObject) {
    let total = 0;
    for (const stage in scoresObject) {
        for (const ring in scoresObject[stage]) {
            if (ring === 'Miss') continue; // Fehlschüsse zählen 0 Punkte
            const wert = ring === 'X' ? 10 : parseInt(ring);
            const anzahl = scoresObject[stage][ring] || 0;
            total += anzahl * wert;
        }
    }
    return total;
}

function getVerbleibendeSchuesse(stage) {
    const maxSchuss = stage.schusszahl;
    const erfassteSchuesse = Object.values(scores.value[stage.name] || {}).reduce((summe, anzahl) => summe + (anzahl || 0), 0);
    return maxSchuss - erfassteSchuesse;
}
</script>

<template>
  <div class="ergebnis-erfassung">
    <h2>{{ ergebnisInBearbeitungId === null ? 'Neues Ergebnis erfassen' : 'Ergebnis bearbeiten' }}</h2>
    <form @submit.prevent="saveErgebnis">
      <div><label for="datum">Datum:</label><input type="date" id="datum" v-model="datum"></div>
      <div><label for="waffe">Waffe:</label><select id="waffe" v-model="gewaehlteWaffeId"><option :value="null" disabled>Bitte Waffe auswählen</option><option v-for="waffe in waffen" :key="waffe.id" :value="waffe.id">{{ waffe.name }}</option></select></div>
      <div><label for="disziplin">Disziplin:</label><select id="disziplin" v-model="gewaehlteDisziplinId"><option :value="null" disabled>Bitte Disziplin auswählen</option><option v-for="disziplin in disziplinen" :key="disziplin.id" :value="disziplin.id">{{ disziplin.name }}</option></select></div>

      <div v-if="gewaehlteDisziplin && zielscheibe" class="stages-container">
        <div v-for="stage in gewaehlteDisziplin.stages" :key="stage.name" class="stage-box">
          <div class="stage-header">
            <h4>{{ stage.name }}</h4>
            <span :class="{ 'verbleibend-ok': getVerbleibendeSchuesse(stage) >= 0, 'verbleibend-fehler': getVerbleibendeSchuesse(stage) < 0 }">
              Verbleibend: {{ getVerbleibendeSchuesse(stage) }} / {{ stage.schusszahl }}
            </span>
          </div>
          <div class="score-inputs">
            <div v-for="ring in zielscheibe.ringe" :key="ring" class="score-input">
              <label :for="`${stage.name}-${ring}`">{{ ring }}</label>
              <input type="number" :id="`${stage.name}-${ring}`" v-model.number="scores[stage.name][ring]" min="0">
            </div>
            <div class="score-input miss-input">
              <label :for="`${stage.name}-Miss`">Miss</label>
              <input type="number" :id="`${stage.name}-Miss`" v-model.number="scores[stage.name]['Miss']" min="0">
            </div>
          </div>
        </div>
      </div>
      
      <div><label for="notizen">Notizen:</label><textarea id="notizen" v-model="notizen"></textarea></div>
      <div class="form-actions">
          <button type="submit">{{ ergebnisInBearbeitungId === null ? 'Ergebnis speichern' : 'Änderungen speichern' }}</button>
          <button type="button" v-if="ergebnisInBearbeitungId !== null" @click="resetForm">Abbrechen</button>
      </div>
    </form>
  </div>

  <div class="ergebnis-liste">
    <h3>Erfasste Ergebnisse</h3>
    <ul>
      <li v-if="ergebnisse.length === 0">Noch keine Ergebnisse erfasst.</li>
      <li v-for="ergebnis in ergebnisse" :key="ergebnis.id">
        <div class="ergebnis-header">
            <span>
                <strong>{{ ergebnis.datum }}:</strong> 
                {{ getNameById(disziplinen, ergebnis.disziplinId) }} 
                - <strong>Gesamt: {{ getGesamtScore(ergebnis.scores) }} Ringe</strong>
            </span>
            <div class="ergebnis-aktionen">
                <button @click="startEditing(ergebnis)">Bearbeiten</button>
                <button @click="deleteErgebnis(ergebnis.id)">Löschen</button>
            </div>
        </div>
        <p v-if="ergebnis.notizen">Notiz: {{ ergebnis.notizen }}</p>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.ergebnis-erfassung, .ergebnis-liste { background-color: #fff; padding: 1rem; border-radius: 8px; margin-top: 1rem; }
form div:not(.form-actions) { margin-bottom: 1rem; }
label { display: block; margin-bottom: 0.25rem; font-weight: 500; }
input, select, textarea { width: 100%; padding: 8px; box-sizing: border-box; border: 1px solid #ccc; border-radius: 4px; }
.stages-container { border: 1px solid #eee; padding: 1rem; border-radius: 5px; }
.stage-box { margin-bottom: 1.5rem; }
.stage-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.75rem; }
.stage-header h4 { margin: 0; }
.verbleibend-ok { font-size: 0.9em; color: #333; }
.verbleibend-fehler { font-size: 0.9em; color: red; font-weight: bold; }
.score-inputs { display: flex; flex-wrap: wrap; gap: 15px; }
.score-input { display: flex; flex-direction: column; align-items: center; }
.score-input input { width: 50px; text-align: center; }
.miss-input { border-left: 2px solid #f0ad4e; padding-left: 15px; }
.form-actions button { margin-right: 10px; }
.ergebnis-liste ul { list-style: none; padding: 0; }
.ergebnis-liste li { border-bottom: 1px solid #eee; padding: 1rem 0; }
.ergebnis-header { display: flex; justify-content: space-between; align-items: center; }
.ergebnis-aktionen button { font-size: 0.8em; padding: 4px 8px; }
.ergebnis-liste p { font-size: 0.9em; color: #666; margin: 0.5rem 0 0; }
</style>