<script setup>
    const props = defineProps({
        medicament: Object
    })
    const emit = defineEmits(['add', 'remove', 'delete', 'update'])

    function add(){
        emit('add', props.medicament)
    }
    function remove(){
        emit('remove', props.medicament)
    }
    function deleteMedicament(){
        emit('delete', props.medicament)
    }
    function update(){
        emit('update', props.medicament)
    }
</script>

<template>
    <button @click="add">+1</button>
    <button @click="remove">-1</button>
    <button @click="deleteMedicament">Supprimer</button>
    <button @click="update">Modifier</button>
    
</template>

<style scoped>

</style>
