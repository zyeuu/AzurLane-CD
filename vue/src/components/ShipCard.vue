<script lang="ts" setup>
import { computed } from 'vue'
import ShipAvatar from './ShipAvatar.vue'
import store from '../utils/store';

const props = defineProps<{
    template: number,
    name?: string,
}>();

const templ = computed(() => {
    return store.state.shipTemplates[props.template];
})

const dispName = computed(() => {
    return props.name || templ.value?.name || '未选择';
})


</script>

<template>
    <div class="ship-card">
        <ship-avatar :src="templ?.img" :size="48" />
        <span class="ship-card-name">{{ dispName }}</span>
        <span v-if="template > 0" class="ship-card-id">ID {{ template }}</span>
    </div>
</template>

<style scoped>
.ship-card {
    display: flex;
    flex-direction: column;
    gap: 4px;
    min-width: 0;
}

.ship-card-name {
    overflow-wrap: anywhere;
}

.ship-card-id {
    color: #666;
    font-size: 12px;
    font-variant-numeric: tabular-nums;
}
</style>
