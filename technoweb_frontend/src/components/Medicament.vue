<script setup>
import { reactive, onMounted } from 'vue';
import Ajout from './Ajout.vue';

const url = "https://springajax.herokuapp.com/api/medicaments";
const fetchoption= {method:"GET"}

const state = reactive({
    medicaments: []
});

function getMedicament(){
    fetch(url,fetchoption)
    .then(response => response.json())
    .then(data => {
        state.medicaments = data._embedded.medicaments;
        console.log(state.medicaments);
    })
    .catch(error => console.log(error))
}

onMounted(() => {
    getMedicament();
});
</script>

<template>
    <!-- medicaments card -->
    <ul class="liste">
        <li v-for="(medicament, index) in state.medicaments" :key="index">
            <div class="card">
                <div class="card-body">
                    <h5 class="card-title">{{ medicament.nom }}</h5>
                    <p class="card-text">{{ medicament.unitesEnStock }}</p>
                    <img :src="medicament.imageURL" alt="Image du médicament">
                    <Ajout :medicament="medicament"/>
                </div>
            </div>
        </li>
    </ul>
</template>

<style scoped>
.liste {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 1rem;
    list-style: none;
    padding: 0;
}
.card{
    width: 18rem;
    color: white;
}
</style>