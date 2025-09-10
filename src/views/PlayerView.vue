<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { Carousel, Slide, Navigation } from 'vue3-carousel';
import 'vue3-carousel/dist/carousel.css';

import router from '@/router';
import { useViewport } from '@/viewport';

import HeroCard from '@/components/HeroCard.vue';
import HeroPortrait from '@/components/HeroPortrait.vue';
import CardSelectPopup from '@/components/popups/CardSelectPopup.vue';
import Spellbook from '@/components/Spellbook.vue';
import { get } from '@/data/heroes';
import { sortCardsByTier, sortCardTiers } from '@/helper/cards';
import { useCompanionStore } from '@/stores/companion';

type CardFn = (c: Card) => boolean | void;

const store = useCompanionStore();

watch(store, () => {
    if (!store.id) {
        router.push({ name: 'heroes' });
    }
})

const hero = computed(() => get(store.id || 'xar'));
const hCards = computed(() => hero.value.cards.slice(0).sort(sortCardsByTier).reduce((acc, card) => {
    if (!acc.hasOwnProperty(card.color)) { acc[card.color] = []; }
    acc[card.color].push(card);
    return acc;
}, {} as { [color: string]: Card[] }));

function arrayUnique<T>(value: T, index: number, array: T[]): boolean {
    return array.indexOf(value) === index;
}

function getUpgrades(list: Card[], card: Card) {
    const options = list.map(c => c.tier).filter(t => sortCardTiers(t, card.tier) > 0).filter(arrayUnique).sort(sortCardTiers);
    return list.filter(c => options.length && c.tier === options[0]);
}

function getDowngrades(list: Card[], card: Card) {
    const options = list.map(c => c.tier).filter(t => sortCardTiers(t, card.tier) < 0).filter(arrayUnique).sort(sortCardTiers);
    return list.filter(c => options.length && c.tier === options[options.length - 1]);
}

function getCard(color: Card['color'], id?: string | null, modifyFn?: CardFn) {
    const card = id ? hCards.value[color].find(c => c.id === id) : hCards.value[color][0];
    const upgrades = getUpgrades(hCards.value[color], card);
    const downgrades = getDowngrades(hCards.value[color], card);

    return { data: card, upgrades, downgrades, modify: modifyFn || (() => null) };
}

const cards = computed(() => [
    getCard('y', store.gold, (card: Card) => { store.gold = card.id }),
    getCard('s', store.silver, (card: Card) => { store.silver = card.id }),
    getCard('r', store.red, (card: Card) => { store.red = card.id }),
    getCard('g', store.green, (card: Card) => { store.green = card.id }),
    getCard('b', store.blue, (card: Card) => { store.blue = card.id }),
    getCard('u'),
]);

function tierToLevel(t: string | null) {
    if (t === 'II') return 1;
    if (t === 'III') return 2;
    return 0;
}

const heroLevel = computed(() => cards.value.reduce((agg, card) => agg + tierToLevel(card.data.tier), 1))

const viewport = useViewport();

function getVisibleCount(width: number) {
    if (width >= (viewport.desktopBoundary)) return 3.8;
    if (width >= viewport.mobileBoundary) return 2.5;
    return 1;
}

const visibleCount = computed(() => getVisibleCount(viewport.width.value));
const navigableSlides = computed(() => cards.value.length - Math.floor(visibleCount.value));
const isAtStart = computed(() => store.focus <= 0);
const isAtEnd = computed(() => store.focus >= navigableSlides.value);
const choice = ref<null | { cards: Card[], select: CardFn }>(null);


function setNewCard(cards: Card[], select: CardFn) {
    if (cards.length === 0) return;
    if (cards.length === 1) return select(cards[0]);

    choice.value = { cards, select };
}

</script>

<template>
    <div>
        <HeroPortrait :hero="hero" :level="heroLevel" />

        <CardSelectPopup v-if="choice?.cards" :cards="choice.cards" :select="choice.select"
            :close="() => choice = null" />

        <div class="carousel-wrapper">
            <Carousel :items-to-show="visibleCount" :model-value="store.focus"
                @update:modelValue="val => store.focus = Math.min(val, navigableSlides)" :wrap-around="false"
                snap-align="center-even" :breakpoints="{
                    0: { itemsToShow: 1, snapAlign: 'center' },
                    [viewport.mobileBoundary]: { itemsToShow: getVisibleCount(viewport.mobileBoundary), snapAlign: 'center-even' },
                    [viewport.desktopBoundary]: { itemsToShow: getVisibleCount(viewport.desktopBoundary), snapAlign: 'center-even' },
                }" class="carousel">

                <template #addons>
                    <Navigation>
                        <template #prev>
                            <button class="nav-button prev" :class="{ disabled: isAtStart }"
                                :disabled="isAtStart">&laquo;</button>
                        </template>
                        <template #next>
                            <button class="nav-button next" :class="{ disabled: isAtEnd }"
                                :disabled="isAtEnd">&raquo;</button>
                        </template>
                    </Navigation>

                    <div class="custom-pagination">
                        <button v-for="(card, index) in cards" :key="index" class="pagination-bullet"
                            :class="[`color-${card.data.color}`, { active: index === store.focus }]"
                            @click="store.focus = index" aria-label="Go to slide" />
                    </div>

                    <div class="carousel-mask left" :class="{ 'hidden': isAtStart }" />
                    <div class="carousel-mask right" :class="{ 'hidden': isAtEnd }" />
                </template>

                <Slide v-for="(card, index) in cards" :key="index" class="slide" :class="{
                    'is-left': index === store.focus - 1,
                    'is-right': index === store.focus + Math.floor(visibleCount),
                    'is-muted': index < store.focus - 1 || index > store.focus + Math.floor(visibleCount)
                }">
                    <div class="slide-card-container">

                        <div class="card-modification upgrade" :class="{ 'disabled': card.upgrades.length === 0 }"
                            :title="$t('app.card.upgrade')" @click="() => setNewCard(card.upgrades, card.modify)">
                            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="12" viewBox="5 5 12 12"
                                fill="none">
                                <path d="M6 16L12 10L18 16" stroke="currentColor" stroke-width="2"
                                    stroke-linecap="round" stroke-linejoin="round" />
                                <path d="M6 12L12 6L18 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                                    stroke-linejoin="round" />
                            </svg>
                        </div>

                        <div class="card-modification downgrade" :class="{ 'disabled': card.downgrades.length === 0 }"
                            :title="$t('app.card.downgrade')" @click="() => setNewCard(card.downgrades, card.modify)">
                            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="12" viewBox="5 7 12 12"
                                fill="none">
                                <path d="M6 8L12 14L18 8" stroke="currentColor" stroke-width="2" stroke-linecap="round"
                                    stroke-linejoin="round" />
                                <path d="M6 12L12 18L18 12" stroke="currentColor" stroke-width="2"
                                    stroke-linecap="round" stroke-linejoin="round" />
                            </svg>
                        </div>

                        <HeroCard class="card" :card="card.data" />
                    </div>

                </Slide>
            </Carousel>
        </div>

        <Spellbook :cards="hero.spellbook" />
    </div>
</template>

<style scoped lang="scss">

</style>
