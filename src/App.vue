<script setup lang="ts">
import { RouterLink, RouterView, useRoute, useRouter } from 'vue-router'
import FaqPopup from './components/popups/FaqPopup.vue';
import LanguageSwitcher from './components/LanguageSwitcher.vue';
import { useCompanionStore } from './stores/companion';
import { computed, onMounted } from 'vue';
import { get } from './data/heroes';
import TimeIndicator from './components/TimeIndicator.vue';
import { useAppStore } from './stores/app';
import { expansions } from './types/Expansion';
import UpdateNotification from './components/UpdateNotification.vue';

const store = useCompanionStore();
const selectedHeroName = computed(() => store.id ? get(store.id)?.name : null);
const build_date = __APP_BUILD_DATE__;
const version_number = __APP_VERSION__;

const router = useRouter();
const route = useRoute();
onMounted(async () => {
    await router.isReady();
    const query = route.query;

    if (Object.keys(query).length <= 0) return;

    let qExpansions: string[];
    if (typeof query.expansions === 'string') {
        qExpansions = query.expansions.split(',')
    } else if (Array.isArray(query.expansions)) {
        qExpansions = query.expansions;
    }

    if (!qExpansions || !qExpansions.length) {
        return;
    }

    qExpansions = qExpansions
        .map(e => e.trim())
        .map(e => e.charAt(0).toUpperCase() + e.slice(1).toLocaleLowerCase())
        .filter(e => expansions.includes(e as any));

    if (!qExpansions.length) return;

    const store = useAppStore();
    store.filteredExpansions = qExpansions;

    if (Object.keys(query).length > 0) {
        router.replace({ query: {} });
    }
});

</script>

<template>
    <header class="no-print">
        <nav>
            <RouterLink to="/hero">{{ $t('app.header.overview') }}</RouterLink>
            <RouterLink to="/settings"><img style="margin-top: .4rem;" src="@/assets/gear.svg" width="24" />
            </RouterLink>


            <div style="position: absolute; top: 2px; right: 8px;">
                <LanguageSwitcher />
            </div>

        </nav>
    </header>

    <div class="content">
        <RouterView />
        <FaqPopup />

        <div class="footer no-print">
            v{{ version_number }} &raquo;
            {{ $t('app.last-update') }}
            <TimeIndicator :date="build_date" />
        </div>
    </div>

    <UpdateNotification />
</template>

<style scoped lang="scss">

</style>
