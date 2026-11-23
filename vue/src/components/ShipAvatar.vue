<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
    src?: string,
    size: number,
}>();

const url = computed(() => props.src && import.meta.env.BASE_URL + props.src.replace(/^\/+/, ''));
</script>

<template>
    <img
        v-if="url"
        class="ship-avatar"
        :src="url"
        :width="size"
        :height="size"
        alt=""
        loading="lazy"
        decoding="async"
    />
</template>

<style scoped>
.ship-avatar {
    display: block;
    object-fit: cover;
    border-radius: 8px;
}
</style>
