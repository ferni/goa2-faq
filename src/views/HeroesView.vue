<script setup lang="ts">
import { computed } from 'vue';
import { get } from '@/data/heroes'
import router from '@/router';
import HeroOverview from '@/components/HeroOverview.vue';
import HeroSelection from '@/components/HeroSelection.vue';

const props = defineProps<{ hero?: string; }>();

const selectedHero = computed(() => {
    if (!props.hero) return null;
    try {
        return get(props.hero);
    } catch (e) {
        console.warn(e);
        return null
    }
});

</script>

<template>
    <div class="hero-overview">
        <Transition :name="'slide-' + (selectedHero ? 'left' : 'right')">
            <div class="overview-selection" v-if="!selectedHero">
                <HeroSelection :onClick="(h) => router.push({ name: 'heroes', params: { hero: h.id } })" />
            </div>

            <div class="overview-details" v-else>
                <HeroOverview :hero="selectedHero" />
            </div>
        </Transition>
    </div>
</template>

<style lang="scss">

</style>
