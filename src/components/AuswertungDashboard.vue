<script setup>
import { ref, computed } from 'vue'

// --- Daten laden ---
const ergebnisse = ref(JSON.parse(localStorage.getItem('shooting-x-diary-ergebnisse')) || [])
const disziplinen = ref(JSON.parse(localStorage.getItem('shooting-x-diary-disziplinen')) || [])

// --- Hilfsfunktion zum Berechnen des Gesamtscores (kopiert von ErgebnisErfassung) ---
function getGesamtScore(scoresObject) {
    if (!scoresObject) return 0;
    let total = 0;
    for (const stage in scoresObject) {
        for (const ring in scoresObject[stage]) {
            if (ring === 'Miss') continue;
            const wert = ring === 'X' ? 10 : parseInt(ring);
            const anzahl = scoresObject[stage][ring] || 0;
            total += anzahl * wert;
        }
    }
    return total;
}

// --- Allgemeine Statistiken ---
const gesamtAnzahlErgebnisse = computed(() => ergebnisse.value.length);

const letzteAktivitaet = computed(() => {
  if (ergebnisse.value.length === 0) return 'Keine';
  // Sortiert die Ergebnisse nach Datum, um das Neueste zu finden
  const neuestesDatum = ergebnisse.value.sort((a, b) => new Date(b.datum) - new Date(a.datum))[0].datum;
  return new Date(neuestesDatum).toLocaleDateString('de-DE');
});

// --- Hauptfunktion für die Auswertung pro Disziplin ---
const disziplinStatistiken = computed(() => {
  return disziplinen.value.map(disziplin => {
    const relevanteErgebnisse = ergebnisse.value
      .filter(e => e.disziplinId === disziplin.id)
      .map(e => ({ ...e, gesamtScore: getGesamtScore(e.scores) }))
      .sort((a, b) => new Date(b.datum) - new Date(a.datum)); // Neueste zuerst

    // Persönlichen Rekord finden
    const persoenlicherRekord = Math.max(0, ...relevanteErgebnisse.map(e => e.gesamtScore));

    // "Best of 3 from last 4" berechnen
    let leistungsSchnitt = 'N/A';
    if (relevanteErgebnisse.length >= 4) {
      const letzteVier = relevanteErgebnisse.slice(0, 4);
      const letzteVierScores = letzteVier.map(e => e.gesamtScore);
      letzteVierScores.sort((a, b) => b - a); // Sortiert die Scores: Bester zuerst
      const besteDrei = letzteVierScores.slice(0, 3);
      const summeBesteDrei = besteDrei.reduce((sum, score) => sum + score, 0);
      leistungsSchnitt = (summeBesteDrei / 3).toFixed(2);
    }

    return {
      id: disziplin.id,
      name: disziplin.name,
      rekord: persoenlicherRekord,
      schnitt: leistungsSchnitt
    };
  });
});
</script>

<template>
  <div class="auswertung-dashboard">
    
    <!-- Allgemeine Statistiken -->
    <div class="stats-grid">
      <div class="stat-card">
        <h4>Erfasste Ergebnisse</h4>
        <p class="stat-value">{{ gesamtAnzahlErgebnisse }}</p>
      </div>
      <div class="stat-card">
        <h4>Letzte Aktivität</h4>
        <p class="stat-value">{{ letzteAktivitaet }}</p>
      </div>
    </div>

    <!-- Statistiken pro Disziplin -->
    <div class="disziplin-stats-card">
        <h3>Leistungsübersicht</h3>
        <table>
            <thead>
                <tr>
                    <th>Disziplin</th>
                    <th>Persönlicher Rekord</th>
                    <th>Leistungsschnitt (Best of 3 from last 4)</th>
                </tr>
            </thead>
            <tbody>
                <tr v-if="disziplinStatistiken.length === 0">
                    <td colspan="3">Keine Disziplinen für eine Auswertung vorhanden.</td>
                </tr>
                <tr v-for="stat in disziplinStatistiken" :key="stat.id">
                    <td>{{ stat.name }}</td>
                    <td><strong>{{ stat.rekord }} Ringe</strong></td>
                    <td>{{ stat.schnitt }}</td>
                </tr>
            </tbody>
        </table>
    </div>

  </div>
</template>

<style scoped>
.auswertung-dashboard {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
}
.stat-card, .disziplin-stats-card {
  background-color: #fff;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
}
.stat-card h4 {
  margin: 0 0 0.5rem 0;
  color: #555;
  font-weight: 600;
}
.stat-value {
  margin: 0;
  font-size: 2.5em;
  font-weight: bold;
  color: var(--primary-color, #0d3e42);
}
.disziplin-stats-card h3 {
    margin-top: 0;
}
table {
  width: 100%;
  border-collapse: collapse;
}
th, td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid #eee;
}
th {
    font-weight: 600;
    color: #333;
}
tbody tr:last-child td {
    border-bottom: none;
}
</style>
