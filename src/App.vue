<script setup>
import { ref } from 'vue'

// Import all the components
import WaffenVerwaltung from './components/WaffenVerwaltung.vue'
import DisziplinenVerwaltung from './components/DisziplinenVerwaltung.vue'
import ErgebnisErfassung from './components/ErgebnisErfassung.vue'
import ZielscheibenVerwaltung from './components/ZielscheibenVerwaltung.vue'
import AuswertungDashboard from './components/AuswertungDashboard.vue'

// A reactive variable to keep track of the active tab
const aktiverTab = ref('Erfassung') // Start with the 'Erfassung' tab
</script>

<template>
  <div id="app-container">
    <header>
      <img src="/x-diary.png" alt="App Logo" class="logo">
      <h1>Shooting-X-Diary</h1>
    </header>

    <nav>
      <button @click="aktiverTab = 'Erfassung'" :class="{ active: aktiverTab === 'Erfassung' }">
        Erfassung
      </button>
      <button @click="aktiverTab = 'Auswertung'" :class="{ active: aktiverTab === 'Auswertung' }">
        Auswertung
      </button>
      <button @click="aktiverTab = 'Stammdaten'" :class="{ active: aktiverTab === 'Stammdaten' }">
        Stammdaten
      </button>
    </nav>

    <main>
      <!-- Conditional rendering based on the active tab -->
      <div v-if="aktiverTab === 'Erfassung'">
        <ErgebnisErfassung />
      </div>

      <div v-if="aktiverTab === 'Auswertung'">
         <AuswertungDashboard />
      </div>

      <div v-if="aktiverTab === 'Stammdaten'">
        <WaffenVerwaltung />
        <DisziplinenVerwaltung />
        <ZielscheibenVerwaltung />
      </div>
    </main>
  </div>
</template>

<style>
/* Global Styles */
:root {
  --primary-color: #0d3e42; /* Your primary color */
  --background-color: #f4f4f9;
  --card-background: #ffffff;
  --text-color: #333;
  --border-color: #e0e0e0;
  --accent-color: #2c5b9c;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  background-color: var(--background-color);
  color: var(--text-color);
  margin: 0;
}

#app-container {
  max-width: 1000px;
  margin: 0 auto;
}

main {
  padding: 1rem;
}

/* Header Styles */
header {
  background-color: var(--card-background);
  padding: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
  border-bottom: 1px solid var(--border-color);
}

.logo {
  height: 50px;
  width: auto;
}

h1 {
  margin: 0;
  font-size: 2.2em;
  color: var(--primary-color);
  font-weight: 600;
}

/* Navigation Styles */
nav {
  display: flex;
  background-color: var(--card-background);
  border-bottom: 1px solid var(--border-color);
}

nav button {
  flex-grow: 1;
  padding: 1rem;
  font-size: 1em;
  background-color: transparent;
  border: none;
  border-bottom: 3px solid transparent; /* Underline effect for active tab */
  color: #555; /* Darker text for light background */
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  font-weight: 500;
}

nav button:hover {
  background-color: #f0f0f0;
}

nav button.active {
  color: var(--primary-color);
  border-bottom-color: var(--primary-color); /* Highlight active tab with an underline */
  font-weight: 700;
}

/* Placeholder card style for empty tabs */
.placeholder-card {
    background-color: var(--card-background);
    padding: 2rem;
    border-radius: 8px;
    text-align: center;
}
</style>
