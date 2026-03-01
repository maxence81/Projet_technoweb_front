<script setup>
    import { ref } from 'vue';

    const props = defineProps({
        medicament: Object
    })
    const emit = defineEmits(['medicamentModifie']);

    function modifier(medicament){
        const selfUrl = props.medicament._links.self.href;
        const categorieUrl = medicament.categorie?._links?.self?.href || props.medicament._links.categorie.href;
        fetch(selfUrl, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                nom: medicament.nom,
                quantiteParUnite: medicament.quantiteParUnite,
                prixUnitaire: medicament.prixUnitaire,
                unitesEnStock: medicament.unitesEnStock,
                unitesCommandees: medicament.unitesCommandees,
                niveauDeReappro: medicament.niveauDeReappro,
                indisponible: medicament.indisponible,
                imageURL: medicament.imageURL,
                categorie: categorieUrl
            })
        })
        .then(response => {
            if (response.ok) {
                console.log("Médicament modifié");
                emit('medicamentModifie', medicament);
            } else {
                erreur.value = "Erreur " + response.status;
            }
        })
        .catch(error => {
            console.log(error);
            erreur.value = "Erreur réseau.";
        });
    }
</script>

<template>
    <div class="modif-card">
        <h4 class="modif-title">Modifier le médicament</h4>
        <form @submit.prevent="modifier(medicament)" class="modif-form">
            <div class="form-group">
                <label for="nom">Nom</label>
                <input type="text" id="nom" v-model="medicament.nom">
            </div>
            <div class="form-group">
                <label for="quantiteParUnite">Conditionnement</label>
                <input type="text" id="quantiteParUnite" v-model="medicament.quantiteParUnite">
            </div>
            <div class="form-row">
                <div class="form-group">
                    <label for="prixUnitaire">Prix (€)</label>
                    <input type="number" id="prixUnitaire" v-model.number="medicament.prixUnitaire" step="0.01">
                </div>
                <div class="form-group">
                    <label for="unitesEnStock">Stock</label>
                    <input type="number" id="unitesEnStock" v-model.number="medicament.unitesEnStock">
                </div>
            </div>
            <div class="form-row">
                <div class="form-group">
                    <label for="unitesCommandees">Commandées</label>
                    <input type="number" id="unitesCommandees" v-model.number="medicament.unitesCommandees">
                </div>
                <div class="form-group">
                    <label for="niveauDeReappro">Seuil réappro</label>
                    <input type="number" id="niveauDeReappro" v-model.number="medicament.niveauDeReappro">
                </div>
            </div>
            <div class="form-group">
                <label for="imageURL">URL de l'image</label>
                <input type="text" id="imageURL" v-model="medicament.imageURL">
            </div>
            <div class="form-group checkbox-group">
                <input type="checkbox" id="indisponible" v-model="medicament.indisponible">
                <label for="indisponible">Indisponible</label>
            </div>
            <button type="submit" class="btn-submit">
                <span class="btn-label">Modifier</span>
                <span class="gradient-container">
                    <span class="gradient"></span>
                </span>
            </button>
        </form>
    </div>
</template>

<style scoped>
.modif-card {
    width: 100%;
    padding: 16px;
    margin-top: 12px;
    border-radius: 20px;
    background: #1a1a1a;
}

.modif-title {
    color: #e0e0e0;
    font-size: 16px;
    font-weight: 700;
    text-align: center;
    margin: 0 0 14px;
}

.modif-form {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.form-row {
    display: flex;
    gap: 10px;
}
.form-row .form-group {
    flex: 1;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 3px;
}

.form-group label {
    font-size: 10px;
    color: #7c7c7c;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    font-weight: 500;
}

.form-group input[type="text"],
.form-group input[type="number"],
.form-group input[type="url"] {
    background: #1e1e1e;
    border: 1px solid #333;
    border-radius: 8px;
    padding: 8px 10px;
    color: #ffffff;
    font-size: 13px;
    outline: none;
    transition: border-color 0.2s;
    box-shadow:
        inset 3px 3px 6px rgb(15, 15, 15),
        inset -3px -3px 6px rgb(40, 40, 40);
}
.form-group input:focus {
    border-color: #818cf8;
}

.checkbox-group {
    flex-direction: row;
    align-items: center;
    gap: 8px;
}
.checkbox-group input[type="checkbox"] {
    accent-color: #818cf8;
    width: 14px;
    height: 14px;
}

.btn-submit {
    border: none;
    outline: none;
    background-color: #3a3a3a;
    width: 100%;
    height: 42px;
    font-size: 14px;
    color: #fff;
    font-weight: 600;
    border-radius: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    position: relative;
    transition: all 0.3s;
    margin-top: 4px;
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
    width: calc(100% - 12px);
    height: 32px;
    text-align: center;
    line-height: 32px;
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