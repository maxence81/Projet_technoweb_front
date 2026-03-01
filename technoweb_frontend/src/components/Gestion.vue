<script setup>
    import { ref } from 'vue';
    import ModifierMedicament from './ModifierMedicament.vue';

    const props = defineProps({
        medicament: Object
    })
    const supprime = ref(false);
    const erreur = ref('');
    const afficherModif = ref(false);
    const emit = defineEmits(['medicamentSupprime', 'medicamentAjoute', 'medicamentModifie']);

    function remove(medicament){
        erreur.value = '';
        const selfUrl = props.medicament._links.self.href;
        const lignesUrl = props.medicament._links.lignes.href;

        fetch(lignesUrl)
            .then(response => response.json())
            .then(data => {
                const lignes = data._embedded?.lignes || [];
                const suppressions = lignes.map(ligne =>
                    fetch(ligne._links.self.href, { method: "DELETE" })
                );
                return Promise.all(suppressions);
            })
            .then(() => {
                return fetch(selfUrl, { method: "DELETE" });
            })
            .then(response => {
                if (response.ok) {
                    console.log("Médicament supprimé");
                    supprime.value = true;
                    emit('medicamentSupprime', medicament);
                } else if (response.status === 409) {
                    erreur.value = "Impossible de supprimer : des lignes de commande existent, il faut supprimer les lignes avant.";
                } else {
                    erreur.value = "Erreur " + response.status;
                }
            })
            .catch(error => {
                console.log(error);
                erreur.value = "Erreur réseau.";
            });
    }

    function ajouter1(medicament){
        const selfUrl = props.medicament._links.self.href;
        const categorieUrl = medicament.categorie?._links?.self?.href || props.medicament._links.categorie.href;
        fetch(selfUrl, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                nom: medicament.nom,
                quantiteParUnite: medicament.quantiteParUnite,
                prixUnitaire: medicament.prixUnitaire,
                unitesEnStock: medicament.unitesEnStock + 1,
                unitesCommandees: medicament.unitesCommandees,
                niveauDeReappro: medicament.niveauDeReappro,
                indisponible: medicament.indisponible,
                imageURL: medicament.imageURL,
                categorie: categorieUrl
            })
        })
        .then(response => {
            if (response.ok) {
                medicament.unitesEnStock++;
                console.log("Stock +1");
            } else {
                erreur.value = "Erreur " + response.status;
            }
        })
        .catch(error => {
            console.log(error);
            erreur.value = "Erreur réseau.";
        });
    }

    function retirer1(medicament){
        if (medicament.unitesEnStock <= 0) {
            erreur.value = "Stock déjà à 0.";
            return;
        }
        const selfUrl = props.medicament._links.self.href;
        const categorieUrl = medicament.categorie?._links?.self?.href || props.medicament._links.categorie.href;
        fetch(selfUrl, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
                nom: medicament.nom,
                quantiteParUnite: medicament.quantiteParUnite,
                prixUnitaire: medicament.prixUnitaire,
                unitesEnStock: medicament.unitesEnStock - 1,
                unitesCommandees: medicament.unitesCommandees,
                niveauDeReappro: medicament.niveauDeReappro,
                indisponible: medicament.indisponible,
                imageURL: medicament.imageURL,
                categorie: categorieUrl
            })
        })
        .then(response => {
            if (response.ok) {
                medicament.unitesEnStock--;
                console.log("Stock -1");
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
    <button class="button" @click="ajouter1(medicament)">
        <span class="label">+1</span>
        <span class="gradient-container">
            <span class="gradient"></span>
        </span>
    </button>
    <button class="button" @click="retirer1(medicament)">
        <span class="label">-1</span>
        <span class="gradient-container">
            <span class="gradient"></span>
        </span>
    </button>
    <button class="button" @click="remove(medicament)">
        <span class="label">Supprimer</span>
        <span class="gradient-container">
            <span class="gradient"></span>
        </span>
    </button>
    <button class="button" @click="afficherModif = !afficherModif">
        <span class="label">Modifier</span>
        <span class="gradient-container">
            <span class="gradient"></span>
        </span>
    </button>
    <p v-if="erreur" class="erreur-msg">{{ erreur }}</p>
    <ModifierMedicament
        v-if="afficherModif"
        :medicament="medicament"
        @medicamentModifie="afficherModif = false; emit('medicamentModifie', $event)"
    />
    
</template>

<style scoped>
.button {
  border: none;
  outline: none;
  background-color: #3a3a3a;
  width: 180px;
  height: 60px;
  font-size: 18px;
  color: #fff;
  font-weight: 600;
  border-radius: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  position: relative;
  transition: all 0.3s;
}

.button::before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  width: 106%;
  height: 120%;
  z-index: -1;
  border-radius: inherit;
  transition: all 0.3s;
}

.gradient-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 106%;
  height: 115%;
  overflow: hidden;
  border-radius: inherit;
  z-index: -2;
  filter: blur(10px);
  transition: all 0.3s;
}

.gradient {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 110%;
  aspect-ratio: 1;
  border-radius: 100%;
  transition: all 0.3s;
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

.label {
  width: 156px;
  height: 45px;
  text-align: center;
  line-height: 45px;
  border-radius: 22px;
  background-color: rgba(43, 43, 43, 1);
  background-image: linear-gradient(
    180deg,
    rgb(43, 43, 43) 0%,
    rgb(68, 68, 68) 100%
  );
}

.button:hover .gradient-container {
  transform: translate(-50%, -50%) scale(0.98);
  filter: blur(5px);
}

.button:hover .gradient {
  filter: blur(5px);
}

@keyframes rotate {
  0% {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  100% {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}
</style>
