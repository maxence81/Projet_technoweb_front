<script setup>
import { reactive, onMounted } from 'vue';

const emit = defineEmits(['medicamentAjoute']);

const url = "https://projet-technoweb-backend.onrender.com/api/medicaments";
const categoriesUrl = "https://projet-technoweb-backend.onrender.com/api/categories";
const fetchoption = { method: "GET" };

const form = reactive({
    nom: '',
    quantiteParUnite: '',
    prixUnitaire: 0,
    unitesEnStock: 0,
    unitesCommandees: 0,
    niveauDeReappro: 0,
    indisponible: false,
    imageURL: '',
    categorie: ''
});

const state = reactive({
    categories: []
});

const message = reactive({ text: '', type: '' });

onMounted(() => {
    fetch(categoriesUrl, fetchoption)
        .then(response => response.json())
        .then(data => {
            state.categories = data._embedded.categories;
        })
        .catch(error => console.log(error));
});

function addMedicament() {
    fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            nom: form.nom,
            quantiteParUnite: form.quantiteParUnite,
            prixUnitaire: form.prixUnitaire,
            unitesEnStock: form.unitesEnStock,
            unitesCommandees: form.unitesCommandees,
            niveauDeReappro: form.niveauDeReappro,
            indisponible: form.indisponible,
            imageURL: form.imageURL,
            categorie: form.categorie
        })
    })
    .then(response => {
        if (!response.ok) throw new Error("Erreur " + response.status);
        return response.json();
    })
    .then(data => {
        console.log("Médicament ajouté :", data);
        message.text = "Médicament ajouté avec succès !";
        message.type = "success";
        emit('medicamentAjoute', data);
        form.nom = '';
        form.quantiteParUnite = '';
        form.prixUnitaire = 0;
        form.unitesEnStock = 0;
        form.unitesCommandees = 0;
        form.niveauDeReappro = 0;
        form.indisponible = false;
        form.imageURL = '';
        form.categorie = '';
    })
    .catch(error => {
        console.log(error);
        message.text = "Erreur lors de l'ajout du médicament.";
        message.type = "error";
    });
}
</script>

<template>
    <div class="ajout-card">
        <div class="ajout-header">
            <h3>Ajouter un médicament</h3>
        </div>

        <div v-if="message.text" class="message" :class="message.type">
            {{ message.text }}
        </div>

        <form @submit.prevent="addMedicament" class="ajout-form">
            <div class="form-group">
                <label for="nom">Nom</label>
                <input type="text" id="nom" v-model="form.nom" required placeholder="Ex: Paracétamol 500mg">
            </div>
            <div class="form-group">
                <label for="categorie">Catégorie</label>
                <select id="categorie" v-model="form.categorie" required>
                    <option value="" disabled>Choisir une catégorie</option>
                    <option
                        v-for="cat in state.categories"
                        :key="cat.code"
                        :value="cat._links.self.href"
                    >
                        {{ cat.libelle }}
                    </option>
                </select>
            </div>
            <div class="form-group">
                <label for="quantiteParUnite">Conditionnement</label>
                <input type="text" id="quantiteParUnite" v-model="form.quantiteParUnite" placeholder="Ex: Boîte de 8 comprimés">
            </div>
            <div class="form-row">
                <div class="form-group">
                    <label for="prixUnitaire">Prix (€)</label>
                    <input type="number" id="prixUnitaire" v-model.number="form.prixUnitaire" step="0.01" min="0">
                </div>
                <div class="form-group">
                    <label for="unitesEnStock">Stock</label>
                    <input type="number" id="unitesEnStock" v-model.number="form.unitesEnStock" min="0">
                </div>
            </div>
            <div class="form-row">
                <div class="form-group">
                    <label for="unitesCommandees">Commandées</label>
                    <input type="number" id="unitesCommandees" v-model.number="form.unitesCommandees" min="0">
                </div>
                <div class="form-group">
                    <label for="niveauDeReappro">Seuil réappro</label>
                    <input type="number" id="niveauDeReappro" v-model.number="form.niveauDeReappro" min="0">
                </div>
            </div>
            <div class="form-group">
                <label for="imageURL">URL de l'image</label>
                <input type="url" id="imageURL" v-model="form.imageURL" placeholder="https://...">
            </div>
            <div class="form-group checkbox-group">
                <input type="checkbox" id="indisponible" v-model="form.indisponible">
                <label for="indisponible">Indisponible</label>
            </div>
            <button type="submit" class="btn-submit">
                <span class="btn-label">Ajouter</span>
                <span class="gradient-container">
                    <span class="gradient"></span>
                </span>
            </button>
        </form>
    </div>
</template>

<style scoped>
.ajout-card {
    width: 100%;
    max-width: 450px;
    border-radius: 30px;
    background: #212121;
    box-shadow:
        15px 15px 30px rgb(25, 25, 25),
        -15px -15px 30px rgb(60, 60, 60);
    padding: 28px 24px;
    margin: 2rem auto;
}

.ajout-header h3 {
    color: #e0e0e0;
    font-size: 20px;
    font-weight: 700;
    text-align: center;
    margin: 0 0 20px;
}

.message {
    text-align: center;
    padding: 8px 12px;
    border-radius: 10px;
    font-size: 13px;
    font-weight: 600;
    margin-bottom: 16px;
}
.message.success {
    background: rgba(34, 197, 94, 0.15);
    color: #4ade80;
}
.message.error {
    background: rgba(239, 68, 68, 0.15);
    color: #f87171;
}

.ajout-form {
    display: flex;
    flex-direction: column;
    gap: 14px;
}

.form-row {
    display: flex;
    gap: 12px;
}
.form-row .form-group {
    flex: 1;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 4px;
}

.form-group label {
    font-size: 11px;
    color: #7c7c7c;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    font-weight: 500;
}

.form-group input[type="text"],
.form-group input[type="number"],
.form-group input[type="url"],
.form-group select {
    background: #2a2a2a;
    border: 1px solid #3a3a3a;
    border-radius: 10px;
    padding: 10px 12px;
    color: #e0e0e0;
    font-size: 14px;
    outline: none;
    transition: border-color 0.2s;
    box-shadow:
        inset 3px 3px 6px rgb(20, 20, 20),
        inset -3px -3px 6px rgb(50, 50, 50);
}
.form-group input:focus,
.form-group select:focus {
    border-color: #818cf8;
}

.form-group select option {
    background: #2a2a2a;
    color: #e0e0e0;
}

.checkbox-group {
    flex-direction: row;
    align-items: center;
    gap: 8px;
}
.checkbox-group input[type="checkbox"] {
    accent-color: #818cf8;
    width: 16px;
    height: 16px;
}

.btn-submit {
    border: none;
    outline: none;
    background-color: #3a3a3a;
    width: 100%;
    height: 48px;
    font-size: 16px;
    color: #fff;
    font-weight: 600;
    border-radius: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    position: relative;
    transition: all 0.3s;
    margin-top: 6px;
}

.btn-submit::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(255, 255, 255, 0.2);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
    width: 102%;
    height: 120%;
    z-index: -1;
    border-radius: inherit;
    transition: all 0.3s;
}

.btn-submit .gradient-container {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 102%;
    height: 115%;
    overflow: hidden;
    border-radius: inherit;
    z-index: -2;
    filter: blur(10px);
    transition: all 0.3s;
}

.btn-submit .gradient {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 110%;
    aspect-ratio: 1;
    border-radius: 100%;
    background-image: linear-gradient(
        90deg,
        hsl(226, 81%, 64%),
        hsl(271, 81%, 64%),
        hsl(316, 81%, 64%),
        hsl(1, 81%, 64%),
        hsl(46, 81%, 64%),
        hsl(91, 81%, 64%),
        hsl(136, 81%, 64%),
        hsl(181, 81%, 64%)
    );
    animation: rotate 2s linear infinite;
    filter: blur(10px);
}

.btn-label {
    width: calc(100% - 16px);
    height: 36px;
    text-align: center;
    line-height: 36px;
    border-radius: 8px;
    background-image: linear-gradient(180deg, rgb(43, 43, 43) 0%, rgb(68, 68, 68) 100%);
}

.btn-submit:hover .gradient-container {
    transform: translate(-50%, -50%) scale(0.98);
    filter: blur(5px);
}
.btn-submit:hover .gradient {
    filter: blur(5px);
}

@keyframes rotate {
    0% { transform: translate(-50%, -50%) rotate(0deg); }
    100% { transform: translate(-50%, -50%) rotate(360deg); }
}
</style>
