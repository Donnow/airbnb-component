<script setup>
import { ref, computed, watch } from 'vue';

const props = defineProps({
    guests: {
        type: Object,
        default: () => ({
            adults: 0,
            children: 0,
            infants: 0,
            pets: 0,
            description: '添加人数'
        })
    }
});

const emit = defineEmits(['select']);

const adults = ref(0);
const children = ref(0);
const infants = ref(0);
const pets = ref(0);

// 当 props.guests 变化时更新本地状态
watch(() => props.guests, (newGuests) => {
    if (newGuests) {
        adults.value = newGuests.adults || 0;
        children.value = newGuests.children || 0;
        infants.value = newGuests.infants || 0;
        pets.value = newGuests.pets || 0;
    }
}, { immediate: true, deep: true });

const totalGuests = computed(() => adults.value + children.value);
const guestDescription = computed(() => {
    const parts = [];

    if (totalGuests.value > 0) {
        parts.push(`${totalGuests.value} 位旅客`);
    }

    if (infants.value > 0) {
        parts.push(`${infants.value} 名婴幼儿`);
    }

    if (pets.value > 0) {
        parts.push(`${pets.value} 只宠物`);
    }

    return parts.join('，') || '添加旅客';
});

const increaseCount = (event, type) => {
    // 阻止事件冒泡，防止菜单关闭
    event.stopPropagation();

    if (type === 'adults' && adults.value < 16) {
        adults.value++;
    } else if (type === 'children' && children.value < 5) {
        children.value++;
    } else if (type === 'infants' && infants.value < 5) {
        infants.value++;
    } else if (type === 'pets' && pets.value < 5) {
        pets.value++;
    }

    updateSelection();
};

const decreaseCount = (event, type) => {
    // 阻止事件冒泡，防止菜单关闭
    event.stopPropagation();

    if (type === 'adults' && adults.value > 0) {
        adults.value--;
    } else if (type === 'children' && children.value > 0) {
        children.value--;
    } else if (type === 'infants' && infants.value > 0) {
        infants.value--;
    } else if (type === 'pets' && pets.value > 0) {
        pets.value--;
    }

    updateSelection();
};

const updateSelection = () => {
    emit('select', {
        adults: adults.value,
        children: children.value,
        infants: infants.value,
        pets: pets.value,
        description: guestDescription.value
    });
};

const resetGuests = (event) => {
    // 阻止事件冒泡
    event.stopPropagation();

    // 重置所有人数为0
    adults.value = 0;
    children.value = 0;
    infants.value = 0;
    pets.value = 0;

    // 通知父组件重置
    updateSelection();
};

const applySelection = (event) => {
    // 阻止事件冒泡
    event.stopPropagation();

    // 发送最终选择给父组件
    updateSelection();

    // 可以添加额外逻辑，例如关闭下拉菜单
    // 这里依赖父组件处理关闭逻辑
};
</script>

<template>
    <div class="guests-dropdown">
        <h3>旅客</h3>

        <div class="guest-option">
            <div class="guest-info">
                <div class="guest-type">成人</div>
                <div class="guest-age">13岁以上</div>
            </div>
            <div class="counter">
                <button :disabled="adults === 0" @click="decreaseCount($event, 'adults')" :class="{ disabled: adults === 0 }">
                    -
                </button>
                <span>{{ adults }}</span>
                <button :disabled="adults === 16" @click="increaseCount($event, 'adults')" :class="{ disabled: adults === 16 }">
                    +
                </button>
            </div>
        </div>

        <div class="guest-option">
            <div class="guest-info">
                <div class="guest-type">儿童</div>
                <div class="guest-age">2-12岁</div>
            </div>
            <div class="counter">
                <button :disabled="children === 0" @click="decreaseCount($event, 'children')" :class="{ disabled: children === 0 }">
                    -
                </button>
                <span>{{ children }}</span>
                <button :disabled="children === 5" @click="increaseCount($event, 'children')" :class="{ disabled: children === 5 }">
                    +
                </button>
            </div>
        </div>

        <div class="guest-option">
            <div class="guest-info">
                <div class="guest-type">婴幼儿</div>
                <div class="guest-age">2岁以下</div>
            </div>
            <div class="counter">
                <button :disabled="infants === 0" @click="decreaseCount($event, 'infants')" :class="{ disabled: infants === 0 }">
                    -
                </button>
                <span>{{ infants }}</span>
                <button :disabled="infants === 5" @click="increaseCount($event, 'infants')" :class="{ disabled: infants === 5 }">
                    +
                </button>
            </div>
        </div>

        <div class="guest-option">
            <div class="guest-info">
                <div class="guest-type">宠物</div>
                <div class="guest-age">服务性动物不计入内</div>
            </div>
            <div class="counter">
                <button :disabled="pets === 0" @click="decreaseCount($event, 'pets')" :class="{ disabled: pets === 0 }">
                    -
                </button>
                <span>{{ pets }}</span>
                <button :disabled="pets === 5" @click="increaseCount($event, 'pets')" :class="{ disabled: pets === 5 }">
                    +
                </button>
            </div>
        </div>
        <div class="selection-summary">
            已选: {{ guestDescription }}
        </div>

        <div class="action-buttons">
            <button class="reset-btn" @click="resetGuests">重置</button>
            <button class="apply-btn" @click="applySelection">应用</button>
        </div>
    </div>
</template>

<style scoped>
.guests-dropdown {
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

.guest-option {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16px 0;
    border-bottom: 1px solid #ebebeb;
}

.guest-info {
    display: flex;
    flex-direction: column;
}

.guest-type {
    font-weight: 600;
    font-size: 14px;
}

.guest-age {
    color: #717171;
    font-size: 12px;
    margin-top: 4px;
}

.counter {
    display: flex;
    align-items: center;
    gap: 16px;
}

.counter button {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    border: 1px solid #b0b0b0;
    background: transparent;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 16px;
}

.counter button:hover:not(.disabled) {
    border-color: #000;
}

.counter button.disabled {
    opacity: 0.3;
    cursor: not-allowed;
}

.counter span {
    width: 20px;
    text-align: center;
}

.selection-summary {
    margin-top: 16px;
    padding: 12px;
    background-color: #f7f7f7;
    border-radius: 8px;
    font-size: 14px;
}

.action-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 16px;
}

.action-buttons button {
    padding: 10px 20px;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
}

.reset-btn {
    background-color: transparent;
    border: 1px solid #dddddd;
    color: #484848;
}

.reset-btn:hover {
    background-color: #f7f7f7;
}

.apply-btn {
    background-color: #ff385c;
    border: none;
    color: white;
}

.apply-btn:hover {
    background-color: #e31c5f;
}

.action-buttons {
    display: flex;
    justify-content: space-between;
    margin-top: 16px;
}

.reset-btn,
.apply-btn {
    flex: 1;
    padding: 12px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    font-size: 14px;
}

.reset-btn {
    background-color: #f0f0f0;
    color: #333;
    margin-right: 8px;
}

.apply-btn {
    background-color: #007bff;
    color: white;
}

.reset-btn:hover {
    background-color: #e0e0e0;
}

.apply-btn:hover {
    background-color: #0056b3;
}
</style>