<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue';
import SearchItem from './SearchItem.vue';
import SearchButton from './SearchButton.vue';

// 管理激活状态
const activeItem = ref(null);

// 集中管理所有搜索数据
const searchData = reactive({
    location: '',
    dates: { startDate: '', endDate: '' },
    guests: { adults: 0, children: 0, infants: 0, pets: 0, description: '添加人数' }
});

// 使用计算属性生成搜索项，确保随着 searchData 更新而更新
const searchItems = computed(() => [
    {
        id: 'location',
        name: '地点',
        replace: searchData.location || '搜索目的地',
        value: searchData.location
    },
    {
        id: 'checkin',
        name: '入住',
        replace: searchData.dates.startDate
            ? `${searchData.dates.startDate}${searchData.dates.endDate ? ' 至 ' + searchData.dates.endDate : ''}`
            : '添加日期',
        value: searchData.dates
    },
    {
        id: 'guests',
        name: '人员',
        replace: searchData.guests.description,
        value: searchData.guests
    }
]);

const setActiveItem = (itemId) => {
    activeItem.value = activeItem.value === itemId ? null : itemId;
};

// 处理子组件的选择事件
const handleSelect = (itemId, data) => {
    console.log('收到选择事件:', itemId, data);

    if (itemId === 'location') {
        searchData.location = data.location || '';
    } else if (itemId === 'checkin') {
        searchData.dates = { ...data };

        // 如果已选择了开始和结束日期，可以自动切换到下一个选项
        if (data.startDate && data.endDate) {
            // 短暂延迟以确保用户看到选择结果
            setTimeout(() => {
                activeItem.value = 'guests';
            }, 500);
        }
    } else if (itemId === 'guests') {
        const updatedGuests = { ...data };
        updatedGuests.description = getGuestDescription(updatedGuests);
        searchData.guests = updatedGuests;

        // 如果用户点击了"应用"按钮，可以自动关闭下拉菜单
        if (document.activeElement && document.activeElement.classList.contains('apply-btn')) {
            setTimeout(() => {
                activeItem.value = null; // 关闭下拉菜单
            }, 100);
        }
    }
};

// 生成人员描述文本
const getGuestDescription = (guestData) => {
    const totalGuests = guestData.adults + guestData.children;
    const parts = [];

    if (totalGuests > 0) {
        parts.push(`${totalGuests} 位旅客`);
    }

    if (guestData.infants > 0) {
        parts.push(`${guestData.infants} 名婴幼儿`);
    }

    if (guestData.pets > 0) {
        parts.push(`${guestData.pets} 只宠物`);
    }

    return parts.join('，') || '添加人数';
};

// 处理搜索按钮点击
const handleSearch = () => {
    console.log('执行搜索:', searchData);
    // 这里可以添加实际的搜索逻辑
};

// 点击外部关闭下拉菜单
const handleOutsideClick = (event) => {
    const searchBar = document.querySelector('.search-bar');
    if (searchBar && !searchBar.contains(event.target)) {
        activeItem.value = null;
    }
};

onMounted(() => {
    document.addEventListener('click', handleOutsideClick);
});

onUnmounted(() => {
    document.removeEventListener('click', handleOutsideClick);
});
</script>

<template>
    <div class="search-bar">
        <SearchItem v-for="(item, index) in searchItems" :key="item.id" :item="item" :isActive="activeItem === item.id" :isLast="index === searchItems.length - 1" @activate="setActiveItem(item.id)" @select="(data) => handleSelect(item.id, data)" />
        <SearchButton @search="handleSearch" />
    </div>
</template>

<style scoped>
.search-bar {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translate(-50%);
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 860px;
    height: 66px;
    border: 1px solid #ccc;
    border-radius: 40px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.search-bar:hover {
    background-color: #f5f5f5;
}
</style>