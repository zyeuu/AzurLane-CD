<template>
    <span class="equip" :class="{ 'equip-empty': !equipInfo }">
        <img
            v-if="iconUrl && failedIcon !== iconUrl"
            :src="iconUrl"
            width="28"
            height="28"
            alt=""
            loading="lazy"
            decoding="async"
            @error="failedIcon = iconUrl"
        />
        <span :class="equipInfo && 'rarity-' + equipInfo.rarity">{{ equipName }}</span>
    </span>
</template>

<script lang="ts" setup>
import { computed, ref } from 'vue'
import store from '../utils/store';

const props = defineProps<{
    equip: number,
    cnt?: number | null,
    label?: string,
}>();

const equipInfo = computed(() => store.state.equips[props.equip]);
const iconUrl = computed(() => {
    const id = equipInfo.value?.icon;
    return id ? `${import.meta.env.BASE_URL}img/equips/${id}.png` : '';
});
const failedIcon = ref('');

const equipName = computed(() => {
    if (props.label) {
        return props.label;
    }
    if (!equipInfo.value?.name) {
        return '－';
    }
    let name = equipInfo.value.name + ' T' + equipInfo.value.tech;
    if (props.cnt) {
        name = '(' + props.cnt + '*)' + name;
    }
    return name;
});

</script>
<style>
.equip {
    display: inline-flex;
    align-items: center;
    gap: 5px;
    vertical-align: middle;
    margin: 2px;
    padding: 2px;
    cursor: pointer;
}
.equip img {
    flex: none;
    object-fit: contain;
}
.rarity-3 {
    color: rgb(0, 180, 180);
}
.rarity-4 {
    color: rgb(128, 0, 128);
}
.rarity-5 {
    color: rgb(180, 160, 0);
}
.rarity-6 {
    background: linear-gradient(to right, blue, red);
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}
.equip-empty {
    min-width: 40px;
}
</style>
