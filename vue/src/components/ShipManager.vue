<template>
    <div style="padding: 20px">
        <n-row v-for="ship, idx in ships" :key="ship.id">
            <n-col :span="21">
                <ship-basic-info :key="ship.id" :ship="ship" @update:model-value="ships[ship.id] = $event"></ship-basic-info>
            </n-col>
            <n-col :span="3">
                <n-button @click="removeShip(ship.id)" type="error">删除</n-button>
            </n-col>
            <hr />
        </n-row>
        <n-space justify="center">
            <n-button @click="startAddShip()">添加舰娘</n-button>
        </n-space>
    </div>

    <n-modal v-model:show="showModal" display-directive="show">
        <n-card style="width: 90%;" title="添加舰娘">
            <template #header-extra>
                <n-button @click="showModal = false">取消</n-button>
            </template>
            <n-form-item label-placement="left" :show-feedback="false">
                <n-input ref="searchRef" v-model:value="pattern" placeholder="搜索舰船 ID / 名称" clearable />
            </n-form-item>
            <br />
            <n-grid x-gap="8" y-gap="8" cols="2 400:3 600:4 800:5 1000:6 1200:12">
                <n-grid-item
                    v-for="option in options"
                    :key="option.id"
                    :bordered="false"
                    class="ship-option"
                    role="button"
                    tabindex="0"
                    @click="addShip(option.id)"
                    @keydown.enter="addShip(option.id)"
                    @keydown.space.prevent="addShip(option.id)"
                >
                    <ship-card :template="option.id"></ship-card>
                </n-grid-item>
            </n-grid>
            <n-empty v-if="options.length == 0" description="没有匹配的舰船"></n-empty>
        </n-card>
    </n-modal>
</template>

<style scoped>
.n-input-number {
    min-width: 50px;
}
.ship-option {
    padding: 8px;
    border: 1px solid #e0e0e6;
    border-radius: 4px;
    cursor: pointer;
}
.ship-option:hover,
.ship-option:focus-visible {
    background: #f3f5f7;
}
</style>

<script setup lang="ts">
import { ref, computed } from "vue";
import { NButton, NModal, NCard, NGrid, NSpace, NGridItem, NEmpty, NFormItem, NInput, NRow, NCol } from 'naive-ui'
import { InputInst } from "naive-ui/lib/input/src/interface";
import ShipBasicInfo from './ShipBasicInfo.vue'
import ShipCard from './ShipCard.vue'
import store from '../utils/store'

const showModal = ref(false);
const ships = computed(() => store.state.ships);
const pattern = ref<string>('');

const options = computed(() => {
    const query = pattern.value.trim().toLowerCase();
    return Object.values(store.state.shipTemplates)
        .filter(ship => !query || String(ship.id).includes(query) || ship.name.toLowerCase().includes(query))
        .sort((a, b) => a.id - b.id);
})

function addShip(templateId: number) {
    store.addShip(templateId);
    showModal.value = false;
}
function removeShip(idx: string) {
    store.removeShip(idx);
}

const searchRef = ref<InputInst | null>(null);
function startAddShip() {
    showModal.value = true;
    setTimeout(() => {
        searchRef.value && searchRef.value.focus();
    }, 2);
}
</script>
