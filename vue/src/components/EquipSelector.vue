<script lang="ts" setup>
import { computed, h, ref, watch } from 'vue'
import { NSelect, NButton, useMessage, SelectOption } from 'naive-ui'

import { CdBuffData, EquipTemplate, EquipType, FleetShip, ShipType } from '../utils/types'
import store from '../utils/store';
import { contains, getShipCdStats } from '../utils/formulas';
import EquipInfo from './EquipInfo.vue'
import { RequipTypeName } from '../utils/namemap';

const props = defineProps<{
    ship: FleetShip,
    baseReload: number,
    techReload: number,
    extraBuffStats: CdBuffData,
}>();

const emit = defineEmits<{
    (event: 'select', equips: number[]): void
}>();

const message = useMessage();
// @ts-ignore
const shipInfo = store.state.ships[props.ship.id];
console.log('shipInfo', shipInfo);
const shipTemplate = store.state.shipTemplates[shipInfo.templateId];
console.log('shipTemplate', shipTemplate);

const slots = [0, 1, 2, 3, 4];
const weaponTypes = new Set([
    EquipType.artillery,
    EquipType.fighter,
    EquipType.torpedo_bomber,
    EquipType.dive_bomber,
    EquipType.reconnaissance,
]);

function isLowRarityWeapon(equip?: EquipTemplate) {
    return !!equip && weaponTypes.has(equip.type) && equip.rarity !== 5 && equip.rarity !== 6;
}

function getOptions(idx: number) {
    let result = [];
    let allow = shipTemplate.equipSlots[idx];
    for (let equipId in store.state.equips) {
        let equip = store.state.equips[equipId];
        if (isLowRarityWeapon(equip) || (store.state.config.ignoreCommonEquips && equip.rarity <= 3)) {
            continue
        }
        if (equip.ship_type_forbidden?.includes(shipTemplate.type)) {
            continue
        }
        if (
            contains(allow, equip.type) ||
            (
                idx === 3 &&
                equip.type === EquipType.auxiliaryCV &&
                (shipTemplate.type === ShipType.CV || shipTemplate.type === ShipType.CVL)
            ) ||
            (
                idx >= 3 &&
                equip.type === EquipType.auxiliaryBB &&
                (shipTemplate.type === ShipType.BB || shipTemplate.type === ShipType.BC || shipTemplate.type === ShipType.BBV)
            )
        ) {
            let name = equip.name + ' T' + equip.tech;
            if (allow.length > 1) {
                name = '(' + RequipTypeName[equip.type] + ')' + name;
            }
            result.push({
                value: equip.id,
                label: name,
                data: equip,
            });
        }
    }
    result.sort((a, b) => {
        if (a.data.type != b.data.type) {
            return a.data.type - b.data.type;
        }
        return (a.data.cd || 0) - (b.data.cd || 0);
    })
    if (result.length > 0 && idx >= 3) {
        result.push({
            value: 0,
            label: '无',
            data: {}
        })
    }
    return result;
}

const filteredOptions = computed(() => slots.map(getOptions));
const equipChoices = ref<number[][]>(slots.map(() => []));
const legacyEquips = computed(() => slots.map(idx => {
    const id = props.ship.equips[idx];
    return isLowRarityWeapon(store.state.equips[id]) ? id : 0;
}));

const resultInfo = ref<{ total?: number, desc?: string, updated?: boolean }>({});
const results = ref<{ [key: string]: any }[]>([]);
const dispResults = computed(() => results.value.slice(0, store.state.config.limit.display));

function getStatsForChoice(equipIds: number[]) {
    return getShipCdStats({ id: props.ship.id, equips: equipIds, extraBuff: {} }, props.extraBuffStats);
}

function product<T>(nestArr: T[][], nullVal: T) {
    function prod(current: T[][], next: T[]) {
        let ret: T[][] = [];
        if (!next || next.length === 0) {
            next = [nullVal]
        }
        for (let a of current) {
            for (let b of next) {
                let item = a.slice();
                item.push(b);
                ret.push(item);
            }
        }
        return ret;
    }
    let data: T[][] = [[]];
    for (let sub of nestArr) {
        data = prod(data, sub);
    }
    return data;
}

function loadAllResults() {
    const choices = equipChoices.value.map((ids, idx) => ids.length ? ids : [legacyEquips.value[idx]]);
    let allResults = product(choices, 0).map(getStatsForChoice);
    allResults.sort((a, b) => {
        return a.realCD - b.realCD;
    })
    return allResults;
}

function updateChoices(idx: number, data: number[]) {
    console.log('updateChoices', idx, data)
    equipChoices.value[idx] = data;

    let cnt = 1;
    let text = []
    for (const i of slots) {
        let cnti = (equipChoices.value[i].length || 1)
        if (filteredOptions.value[i].length) {
            text.push(cnti)
        }
        cnt = cnt * cnti;
    }
    console.log('总选项数量', text, cnt);
    resultInfo.value = { total: cnt, desc: text.join('*') };
    results.value = [];
    if (cnt < 50) {
        calculateChoices();
    }
    console.log('results', resultInfo.value)
}

function calculateChoices() {
    console.log('计算CD', resultInfo.value.total, store.state.config.limit.calculate)
    if (resultInfo.value.total && resultInfo.value.total > store.state.config.limit.calculate) {
        message.warning('选项数量过多!(当前限制为' + store.state.config.limit.calculate + ')')
        return;
    }
    results.value = loadAllResults();
    resultInfo.value.updated = true;
    console.log('CD计算结果', results.value);
}

watch(() => props.ship.equips, equips => {
    equipChoices.value = slots.map(idx => {
        const id = equips[idx];
        return id && !legacyEquips.value[idx] ? [id] : [];
    });
    updateChoices(0, equipChoices.value[0]);
}, { immediate: true });

watch(() => [props.techReload, props.baseReload], () => {
    // 更新舰娘装填信息后, 清除当前列表
    results.value = [];
})

function renderLabel(option: SelectOption) {
    return h(EquipInfo, { equip: Number(option.value), label: String(option.label) });
}

</script>


<template>
    <div>
        <n-select
            v-for="idx in slots"
            :key="idx"
            multiple
            cascade
            checkable
            filterable
            default-expand-all
            check-strategy="child"
            :value="equipChoices[idx]"
            @update:value="updateChoices(idx, $event)"
            :options="filteredOptions[idx]"
            max-tag-count="responsive"
            :placeholder="legacyEquips[idx] ? '保留已装装备' : '请选择'"
            :disabled="filteredOptions[idx].length === 0"
            :render-label="renderLabel"
        />
        总选项数量: {{ resultInfo.desc }}={{ resultInfo.total }}
        <n-button type="primary" @click="calculateChoices()">计算CD</n-button>
        <div class="equip-results">
            <table>
                <tbody>
                    <tr v-for="c in dispResults" @click="emit('select', c.ids)">
                        <td v-for="(equip, idx) in c.ids" :key="idx">
                            <equip-info :equip="equip"></equip-info>
                        </td>
                        <td>面板CD {{ c.dispCD }}</td>
                        <td>实际CD {{ c.realCD && c.realCD.toFixed(4) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.equip-results {
    overflow-x: auto;
}
</style>
