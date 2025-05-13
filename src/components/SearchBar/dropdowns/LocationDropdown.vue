<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
    location: {
        type: String,
        default: ''
    }
});

const emit = defineEmits(['select']);

// 保存之前选择的位置
const selectedLocation = ref('');

// 当父组件传入的位置变化时更新本地状态
watch(() => props.location, (newLocation) => {
    if (newLocation) {
        selectedLocation.value = newLocation;
    }
}, { immediate: true });

const locations = [
    '北京',
    '上海',
    '广州',
    '深圳',
    '杭州'
];

const selectLocation = (location) => {
    selectedLocation.value = location;
    emit('select', { location });
};
</script>

<template>
    <div class="location-dropdown">
        <h3>热门目的地</h3>
        <div class="location-list">
            <div v-for="location in locations" :key="location" class="location-item" :class="{ 'active': selectedLocation === location }" @click="selectLocation(location)">
                {{ location }}
            </div>
        </div>
    </div>
</template>

<style scoped>
.location-dropdown {
    width: 350px;
    background-color: white;
    border-radius: 16px;
    box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
    padding: 24px;
}

h3 {
    font-size: 14px;
    margin-bottom: 16px;
}

.location-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.location-item {
    padding: 10px 16px;
    border-radius: 8px;
}

.location-item:hover,
.location-item.active {
    background-color: #f5f5f5;
}
</style>