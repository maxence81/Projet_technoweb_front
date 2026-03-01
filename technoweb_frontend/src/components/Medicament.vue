<script setup>
import { reactive, ref, computed, onMounted } from 'vue';
import Ajout from './Gestion.vue';
import AjoutMedicament from './AjoutMedicament.vue';

const url = "https://projet-technoweb-backend.onrender.com/api/medicaments";
const fetchoption = { method: "GET" };

const state = reactive({
    medicaments: []
});

const recherche = ref('');

const medicamentsFiltres = computed(() => {
    if (!recherche.value.trim()) return state.medicaments;
    const q = recherche.value.toLowerCase();
    return state.medicaments.filter(m =>
        m.nom.toLowerCase().includes(q) ||
        (m.categorie?.libelle || m.categorie?.nom || '').toLowerCase().includes(q)
    );
});

function getMedicament() {
    fetch(url + "?size=1000", fetchoption)
        .then(response => response.json())
        .then(data => {
            state.medicaments = data._embedded.medicaments;
            console.log(state.medicaments);
            state.medicaments.forEach(medicament => getCategorie(medicament));
        })
        .catch(error => console.log(error));
}

function getCategorie(medicament){
    fetch(medicament._links.categorie.href,fetchoption)
    .then(response => response.json())
    .then(data => {
        medicament.categorie = data;
        console.log(medicament.categorie);
    })
    .catch(error => console.log(error));
}
function onMedicamentSupprime(medicament) {
    const index = state.medicaments.indexOf(medicament);
    if (index !== -1) {
        state.medicaments.splice(index, 1);
    }
}

function onMedicamentAjoute(medicament){
    getMedicament();
}

function onMedicamentModifie(medicament){
    getMedicament();
}

onMounted(() => {
    getMedicament();
});
</script>

<template>
    <div class="search-wrapper">
        <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <circle cx="11" cy="11" r="8"/>
            <line x1="21" y1="21" x2="16.65" y2="16.65"/>
        </svg>
        <input
            type="text"
            v-model="recherche"
            placeholder="Rechercher un médicament…"
            class="search-input"
        />
    </div>

    <ul class="liste">
        <li v-for="(medicament, index) in medicamentsFiltres" :key="index">
            <div class="card">
                <div class="stock-badge" :class="medicament.unitesEnStock > 10 ? 'badge-ok' : 'badge-low'">
                    <span class="stock-dot"></span>
                    {{ medicament.unitesEnStock > 10 ? 'En stock' : 'Stock faible' }}
                </div>

                <div class="card-image-wrapper">
                    <img
                        :src="medicament.imageURL"
                        :alt="'Image de ' + medicament.nom"
                        class="card-img"
                        @error="$event.target.src='https://placehold.co/80x80/2a2a2a/b0b0b0?text=💊'"
                    />
                </div>
                <div class="card-categorie">
                    {{ medicament.categorie?.libelle ?? medicament.categorie?.nom ?? '…' }}
                </div>
                <div class="card-body">
                    <h5 class="card-title">{{ medicament.nom }}</h5>
                    <div class="card-info">
                        <span class="info-label">Stock</span>
                        <span class="info-value">{{ medicament.unitesEnStock }} unités</span>
                    </div>
                    <div v-if="medicament.prix" class="card-info">
                        <span class="info-label">Prix</span>
                        <span class="info-value price">{{ medicament.prix }} €</span>
                    </div>
                </div>

                <div class="card-divider"></div>

                <div class="card-actions">
                    <Ajout :medicament="medicament" @medicamentSupprime="onMedicamentSupprime(medicament)" @medicamentAjoute="onMedicamentAjoute(medicament)" @medicamentModifie="onMedicamentModifie(medicament)"/>
                </div>
            </div>
        </li>
    </ul>
    <AjoutMedicament @medicamentAjoute="getMedicament()" />
</template>

<style scoped>
.search-wrapper {
    position: relative;
    max-width: 500px;
    margin: 0 auto 1.5rem;
    padding: 0 2rem;
}
.search-icon {
    position: absolute;
    left: calc(2rem + 14px);
    top: 50%;
    transform: translateY(-50%);
    width: 18px;
    height: 18px;
    color: #666;
    pointer-events: none;
}
.search-input {
    width: 100%;
    padding: 12px 16px 12px 44px;
    background: #1e1e1e;
    border: 1px solid #333;
    border-radius: 16px;
    color: #fff;
    font-size: 15px;
    outline: none;
    transition: border-color 0.2s, box-shadow 0.2s;
    box-shadow:
        inset 3px 3px 8px rgb(12, 12, 12),
        inset -3px -3px 8px rgb(45, 45, 45);
}
.search-input::placeholder {
    color: #666;
}
.search-input:focus {
    border-color: #818cf8;
    box-shadow:
        inset 3px 3px 8px rgb(12, 12, 12),
        inset -3px -3px 8px rgb(45, 45, 45),
        0 0 12px rgba(129, 140, 248, 0.15);
}

.liste {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 2rem;
    list-style: none;
    padding: 1.5rem 0;
    margin: 0;
    justify-items: center;
}

.card {
    width: 100%;
    max-width: 360px;
    border-radius: 30px;
    background: #212121;
    box-shadow:
        15px 15px 30px rgb(25, 25, 25),
        -15px -15px 30px rgb(60, 60, 60);
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 28px 24px 20px;
    position: relative;
    overflow: hidden;
    transition: transform 0.25s ease, box-shadow 0.25s ease;
    cursor: pointer;
}

.card:hover {
    transform: translateY(-6px);
    box-shadow:
        20px 20px 40px rgb(18, 18, 18),
        -20px -20px 40px rgb(65, 65, 65);
}

.stock-badge {
    position: absolute;
    top: 16px;
    right: 16px;
    display: flex;
    align-items: center;
    gap: 5px;
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0.03em;
    padding: 4px 10px;
    border-radius: 20px;
    text-transform: uppercase;
}

.badge-ok {
    background: rgba(34, 197, 94, 0.15);
    color: #4ade80;
}

.badge-low {
    background: rgba(239, 68, 68, 0.15);
    color: #f87171;
}

.stock-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: currentColor;
    animation: pulse 1.8s infinite;
}

@keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
}

.card-image-wrapper {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: #2a2a2a;
    box-shadow:
        6px 6px 12px rgb(20, 20, 20),
        -6px -6px 12px rgb(55, 55, 55);
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 8px;
    flex-shrink: 0;
    overflow: hidden;
}

.card-img {
    width: 80px;
    height: 80px;
    object-fit: cover;
    border-radius: 50%;
}

.card-categorie {
    margin-top: 12px;
    font-size: 12px;
    font-weight: 500;
    color: #7c7c7c;
    text-transform: uppercase;
    letter-spacing: 0.06em;
}

.card-body {
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    padding-top: 6px;
    text-align: center;
}

.card-title {
    font-size: 18px;
    font-weight: 700;
    color: #e0e0e0;
    margin: 0;
    line-height: 1.3;
    letter-spacing: 0.01em;
    max-width: 340px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.card-info {
    display: flex;
    justify-content: space-between;
    width: 100%;
    padding: 0 12px;
}

.info-label {
    font-size: 12px;
    color: #6b6b6b;
    font-weight: 500;
    text-transform: uppercase;
    letter-spacing: 0.05em;
}

.info-value {
    font-size: 13px;
    color: #a0a0a0;
    font-weight: 600;
}

.info-value.price {
    color: #818cf8;
}

.card-divider {
    width: 85%;
    height: 1px;
    background: linear-gradient(90deg, transparent, #3a3a3a, transparent);
    margin: 10px 0 8px;
    flex-shrink: 0;
}

.card-actions {
    display: flex;
    gap: 8px;
    justify-content: center;
    flex-wrap: wrap;
    flex-shrink: 0;
    width: 100%;
    padding: 0 8px;
}

.card-actions :deep(.button) {
    width: auto !important;
    min-width: 70px;
    height: 38px !important;
    font-size: 13px !important;
    border-radius: 10px;
}

.card-actions :deep(.label) {
    width: auto !important;
    min-width: 56px;
    height: 28px !important;
    line-height: 28px !important;
    padding: 0 10px;
    border-radius: 8px;
    font-size: 12px;
}

.card-actions :deep(.button::before) {
    width: 108%;
    height: 125%;
}

.card-actions :deep(.gradient-container) {
    width: 108%;
    height: 120%;
}
</style>