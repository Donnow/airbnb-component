<script setup>
import SearchIcon from '@/assets/search.svg';

const emit = defineEmits(['search']);

const handleClick = () => {
    emit('search');
};
</script>

<template>
    <div class="search-button" @click="handleClick">
        <SearchIcon />
    </div>
</template>

<style scoped>
.search-button {
    position: absolute;
    top: 50%;
    right: 8px;
    transform: translateY(-50%);
    display: flex;
    justify-content: center;
    align-items: center;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    color: #fff;
    background-color: #f00;
    cursor: pointer;
}

.search-button:hover {
    background-color: #e41d54;
}
</style>