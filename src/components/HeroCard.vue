<script setup lang="ts">
import { computed } from 'vue';
import { useI18n } from 'vue-i18n'
import { count as countFaq } from '@/data/faq'
import { useAppStore } from '@/stores/app'
import Markdown from './Markdown.vue';

const props = defineProps<{ card: Card; }>();
const hasFaq = computed(() => countFaq(props.card.id) > 0)

function showFAQs() {
    if (hasFaq.value) {
        useAppStore().$showFaq(props.card.id)
    }
}

const { t: $translate } = useI18n();

function processCardType(type: Card['type']) {
    let tt = $translate('app.card-type.' + (type.basic ? 'basic.' : '') + type.type[0]);

    for (let i = 1; i < type.type.length; i++) {
        tt += ' / ' + $translate('app.card-type.' + type.type[i]);
    }

    if (type.ranged) {
        tt += ' \u2014 ' + $translate('app.card-type.ranged')
    }

    return tt;
}

</script>

<template>
    <div class="hero-card-wrapper" :class="{ [`hero-card-color-${card.color}`]: true, 'has-faq': hasFaq }"
        @click="showFAQs()">
        <div class="hero-card-name">{{ card.name }}</div>
        <div v-if="card.tier" class="hero-card-tier">{{ card.tier }}</div>
        <div class="hero-card-type">
            {{ processCardType(card.type) }}
        </div>
        <div class="hero-card-text">
            <Markdown :text="$t(card.text)" />
        </div>
    </div>
</template>


<style lang="scss">
.hero-card-wrapper {
    font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;

    font-weight: 300;
    -webkit-font-smoothing: antialiased;

    page-break-inside: avoid !important;


    p {
        margin-top: 0;
        margin-bottom: 0;
    }
    .hero-card-name {
        font-weight: 400;
        font-size: 8pt;
    }

    .hero-card-tier {
        display: none;
    }

    .hero-card-type {
        display: none;
    }

    .hero-card-text {
        padding-top: 1px;
        padding-bottom: 1px;
        text-align: center;
        border: 1px solid rgb(165, 165, 165);
        width: 51mm;
        background-color: #BDBDBD;
        .markdown-container {
            line-height: 9pt;
        }
        
        font-size: 6.5pt;
        letter-spacing: -0.3pt;
        em {
          font-size: 5.5pt;  
          letter-spacing: -0.4pt;
        }
    }
}
</style>
