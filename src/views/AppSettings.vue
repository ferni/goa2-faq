<script setup lang="ts">
import ProposeFAQ from '@/components/Settings/ProposeFAQ.vue';
import QrCode from '@/components/Settings/QrCode.vue';

defineProps<{ setting?: string; }>();
</script>

<template>

    <div v-if="setting">
        <RouterLink class="settings-back no-print" to="/settings">
            <img src="@/assets/back-arrow.svg" width="20" />
        </RouterLink>
        <ProposeFAQ v-if="setting === 'faq'" />
        <QrCode v-else-if="setting === 'qr'" />
    </div>
    <div v-else>
        <div class="app-settings">
            <div>
                <RouterLink to="/settings/faq">{{ $t('app.settings.propose-FAQ') }}</RouterLink>
            </div>
            <div>
                <RouterLink to="/settings/qr">{{ $t('app.settings.generate-qr') }}</RouterLink>
            </div>
        </div>
    </div>
</template>

<style>

</style>
