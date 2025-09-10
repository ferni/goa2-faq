<script setup lang="ts">
import { computed } from 'vue';
import { cardColumn } from '@/helper/cards';
import HeroCard from './HeroCard.vue';

const props = defineProps<{
    cards: Card[];
}>();

const columns = computed(() => props.cards.slice(0).reduce<Card[][]>(
    (acc, current) => {
        acc[cardColumn(current)].push(current);
        return acc;
    },
    [[], [], [], []] as Card[][]
));

</script>

<template>
    <div class="card-column-container">
        <div v-for="card in cards">
                <HeroCard :card="card" />
        </div>
    </div>
</template>

<style lang="scss">
.card-column-container {
    display: flex;
    flex-wrap: wrap;
}
</style>
