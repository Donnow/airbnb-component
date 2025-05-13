<script setup>
import { ref, watch, computed } from 'vue';

const props = defineProps({
    dates: {
        type: Object,
        default: () => ({ startDate: '', endDate: '' })
    }
});

const emit = defineEmits(['select']);

// 初始化日期状态，确保安全访问
const startDate = ref('');
const endDate = ref('');

// 当 props.dates 变化时更新本地状态
watch(() => props.dates, (newDates) => {
    if (newDates) {
        startDate.value = newDates.startDate || '';
        endDate.value = newDates.endDate || '';
    }
}, { immediate: true, deep: true });

// 简化的日期数据 - 实际应用中可能需要日历组件
const currentMonth = new Date().getMonth();
const currentYear = new Date().getFullYear();

// 检查日期是否在选择范围内
const isInRange = (day) => {
    if (!startDate.value || !endDate.value) return false;

    const dateStr = `${currentYear}-${(currentMonth + 1).toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`;
    const date = new Date(dateStr);
    const start = new Date(startDate.value);
    const end = new Date(endDate.value);

    return date > start && date < end;
};

// 生成当月的日期
const generateDays = (year, month) => {
    const daysInMonth = new Date(year, month + 1, 0).getDate();
    const days = [];

    for (let i = 1; i <= daysInMonth; i++) {
        days.push(i);
    }

    return days;
};

const days = generateDays(currentYear, currentMonth);
const monthNames = ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月'];

const selectDate = (event, day) => {
    const selectedDate = `${currentYear}-${(currentMonth + 1).toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`;

    if (!startDate.value || (startDate.value && endDate.value)) {
        // 如果没有开始日期或者已经选择了结束日期，重新开始选择
        event.stopPropagation(); // 阻止事件冒泡，防止菜单关闭
        startDate.value = selectedDate;
        endDate.value = '';
        // 通知父组件入住日期已选择
        emit('select', { startDate: startDate.value, endDate: endDate.value });
    } else {
        // 如果已有开始日期，选择结束日期

        if (new Date(selectedDate) >= new Date(startDate.value)) {
            endDate.value = selectedDate;
            // 通知父组件日期已选择完成
            emit('select', { startDate: startDate.value, endDate: endDate.value });
        } else {
            // 如果选择的日期早于开始日期，更新开始日期
            startDate.value = selectedDate;
            endDate.value = '';
            // 通知父组件入住日期已更新
            emit('select', { startDate: startDate.value, endDate: endDate.value });
        }
    }
};

// 格式化日期为更友好的显示形式
const formatDate = (dateString) => {
    if (!dateString) return '';

    const date = new Date(dateString);
    const month = date.getMonth() + 1;
    const day = date.getDate();

    return `${month}月${day}日`;
};
</script>

<template>
    <div class="date-dropdown">
        <h3>选择日期</h3>
        <div class="date-status">
            <div v-if="startDate && !endDate" class="date-status-text">
                <strong>入住日期:</strong> {{ formatDate(startDate) }} | 请选择退房日期
            </div>
            <div v-else-if="startDate && endDate" class="date-status-text">
                <strong>入住:</strong> {{ formatDate(startDate) }} | <strong>退房:</strong> {{ formatDate(endDate) }}
            </div>
            <div v-else class="date-status-text">
                请选择入住日期
            </div>
        </div>

        <div class="calendar">
            <div class="month-header">
                {{ monthNames[currentMonth] }} {{ currentYear }}
            </div>

            <div class="weekdays">
                <div class="weekday" v-for="day in ['一', '二', '三', '四', '五', '六', '日']" :key="day">
                    {{ day }}
                </div>
            </div>

            <div class="days">
                <!-- 空白填充，根据当月第一天是周几 -->
                <div class="day empty" v-for="n in new Date(currentYear, currentMonth, 1).getDay()" :key="'empty-' + n"></div>
                <div v-for="day in days" :key="day" class="day" :class="{
                    'selected-start': startDate === `${currentYear}-${(currentMonth + 1).toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`,
                    'selected-end': endDate === `${currentYear}-${(currentMonth + 1).toString().padStart(2, '0')}-${day.toString().padStart(2, '0')}`,
                    'in-range': isInRange(day)
                }" @click="selectDate($event, day)">
                    {{ day }}
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.date-dropdown {
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

.date-status {
    margin-bottom: 16px;
    padding: 12px;
    background-color: #f8f8f8;
    border-radius: 8px;
    font-size: 13px;
}

.date-status-text {
    line-height: 1.4;
}

.date-status strong {
    color: #222;
}

.month-header {
    text-align: center;
    font-weight: bold;
    margin-bottom: 12px;
}

.weekdays {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    text-align: center;
    margin-bottom: 8px;
}

.weekday {
    font-size: 12px;
    color: #777;
}

.days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 4px;
}

.day {
    height: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    cursor: pointer;
}

.day:hover {
    border: 2px solid #222;
}

.day.empty {
    cursor: default;
}

.selected-start,
.selected-end {
    background-color: #222;
    color: white;
}

.in-range {
    background-color: #ffd9df;
    color: #484848;
}

.day.empty:hover {
    background-color: transparent;
}
</style>