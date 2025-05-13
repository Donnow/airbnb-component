<script setup>
import LocationDropdown from './dropdowns/LocationDropdown.vue';
import DateDropdown from './dropdowns/DateDropdown.vue';
import GuestsDropdown from './dropdowns/GuestsDropdown.vue';

const props = defineProps({
    item: {
        type: Object,
        required: true
    },
    isActive: {
        type: Boolean,
        default: false
    },
    isLast: {
        type: Boolean,
        default: false
    }
});

const emit = defineEmits(['activate', 'select']);

const handleClick = () => {
    emit('activate');
};
</script>

<template>
    <div class="search-item" :class="{ 'active': isActive }" @click="handleClick">
        <label>{{ item.name }}</label>
        <div class="search-item-input">{{ item.replace }}</div>
        <!-- 分割线 -->
        <div v-if="!isLast" class="divider"></div>

        <!-- 动态加载对应的下拉组件 -->
        <div v-if="isActive" class="dropdown-container">
            <component :is="item.id === 'location'
                ? LocationDropdown
                : item.id === 'checkin'
                    ? DateDropdown
                    : GuestsDropdown" :location="item.id === 'location' ? item.value : undefined" :dates="item.id === 'checkin' ? item.value : undefined" :guests="item.id === 'guests' ? item.value : undefined" @select="(data) => $emit('select', data)" />
        </div>
    </div>
</template>

<style scoped>
.search-item {
    position: relative;
    width: 284px;
    height: 66px;
    border-radius: 40px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start;
    padding: 32px;
    cursor: pointer;
    transition: all 0.2s ease;
}

.search-item.active {
    background-color: #f7f7f7b1;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.06);
}

.search-item:hover:not(.active) {
    background-color: #c9c9c9;
}

.divider {
    position: absolute;
    top: 25%;
    right: 0;
    height: 50%;
    border-right: 1px solid #dddddd;
}

.search-item label {
    font-size: 12px;
    color: rgb(34, 34, 34);
    font-weight: 600;
}

.search-item-input {
    font-size: 14px;
    color: #333;
    background-color: transparent;
    font-weight: 400;
    border: none;
    outline: none;
    transition: color 0.2s;
}

.active .search-item-input {
    color: #222;
    font-weight: 500;
}

.dropdown-container {
    position: absolute;
    top: 100%;
    left: 0;
    z-index: 10;
    margin-top: 12px;
}
</style>