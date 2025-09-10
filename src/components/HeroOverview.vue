<script setup lang="ts">
import { computed } from 'vue';
import HeroPortrait from './HeroPortrait.vue';
import CardColorColumns from './CardColorColumns.vue';
import { useCompanionStore } from '@/stores/companion';
import router from '@/router';
import Spellbook from './Spellbook.vue';

const props = defineProps<{ hero: Hero; }>();
const store = useCompanionStore();
const isFavorite = computed(() => store.id === props.hero.id)

function setFavorite(h: Hero) {
    store.reset();
    store.id = h.id;
    router.push({ name: 'player' })
}

function unsetFavorite() {
    store.reset();
}

</script>

<template>
    <div>
        <CardColorColumns :cards="hero.cards" />
        <Spellbook :cards="hero.spellbook" />
    </div>
</template>

<style lang="scss">
.construction-site {
    background-color: #f6c033;
    border: 3px solid #000;
    color: black;
    font-weight: bold;
    white-space: pre;

    box-shadow:
        4px 4px 1px #f6c033,
        4px -4px 1px #f6c033,
        -4px -4px 1px #f6c033,
        -4px 4px 1px #f6c033;

    border-radius: 1em;
    margin: 2em auto;
    padding: .25em 5em;
    min-width: 250px;
    width: 45%;

    text-align: center;

    svg {
        margin: 0 auto;
    }
}

.btn-favorite {
    position: absolute;
    top: .05em;
    left: .15em;
    text-decoration: none;
    background: var(--color-background-highlight);
    color: #fff;
    font-size: 2rem;
    padding: .05em 0.2em;
    border-radius: 1em;
    overflow: hidden;
    font-weight: bold;
    line-height: 1;
    display: inline-flex;
    align-items: center;
    justify-content: center;

    border: 1px solid rgba(0, 0, 0, 0.75);
    box-shadow: 0 1px 3px 1px #000;
    transition: .5s ease-out;
    z-index: 10;
    cursor: pointer;
    text-shadow: 0 0 3px rgba(0, 0, 0, 0.6);

    &.active {
        background: var(--color-background-softer);
        color: rgb(247, 45, 45);
        text-shadow: 0 0 3px rgba(0, 0, 0, 1);
    }

    &:hover {
        color: rgb(255, 144, 144);
    }

    &:not(.active):hover {
        background: #50a0ce;
    }
}
</style>
