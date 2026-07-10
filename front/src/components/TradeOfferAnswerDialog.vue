<template>
    <o-dialog v-model:active="showDialog" modal :title="t('waitingForPlayerAnswers')" :closable="false"
        :closeOnBackdrop="false" :closeOnEscape="false">
        <div class="flex flex-col">
            <div class="flex justify-center items-center gap-4 mb-8 flex-col">
                <div>{{ t(status) }}</div>
                <div v-for="answer in tradeAnswers" @click="answerClick(answer)"
                    class="flex justify-center items-center gap-2" :style="answerStyle(answer)">
                    <div>{{ answer.playerName }}</div>
                    <div>
                        <o-icon v-if="answer.state == TradeAnswerState.WAITING_FOR_ANSWER" spin
                            icon="timer-sand-empty"></o-icon>
                        <o-icon v-if="answer.state == TradeAnswerState.ACCEPTED" icon="check"
                            variant="success"></o-icon>
                        <o-icon v-if="answer.state == TradeAnswerState.REJECTED" icon="close" variant="danger"></o-icon>
                    </div>
                </div>
            </div>
        </div>
    </o-dialog>
</template>

<script setup lang="ts">

import { ODialog, OIcon } from '@oruga-ui/oruga-next';
import { computed, type PropType } from 'vue'
import { type CatanPlayerTradeOffer, type TradeAnswer, TradeAnswerState } from 'catan-back'
import { useI18n } from 'vue-i18n'
import type { Player } from 'boardgame-web-common'

let localization: any = {
    en: {
        waitingForPlayerAnswers: 'Waiting For Players Answers',
        choosePlayer: 'Choose player'
    },
    ru: {
        waitingForPlayerAnswers: 'Ожидание ответа игроков',
        choosePlayer: 'Выберите игрока'
    }
}
const { t } = useI18n({
    locale: 'en',
    messages: localization
})

interface TradeAnswerWithPlayerName extends TradeAnswer {
    playerName: string
}

const status = computed(() => {
    if (props.playerTradeOffer?.answers.some(ans => ans.state == TradeAnswerState.ACCEPTED)) {
        return 'choosePlayer'
    }
    return 'waitingForPlayerAnswers'
})

function answerStyle(answer: TradeAnswerWithPlayerName) {
    return {
        'cursor': answer.state == TradeAnswerState.ACCEPTED ? 'pointer' : 'default'
    }
}

const tradeAnswers = computed(() => {
    return props.playerTradeOffer?.answers.map(answer => {
        return {
            playerId: answer.playerId,
            playerName: props.players.find(pl => pl.userId == answer.playerId)?.name,
            state: answer.state
        } as TradeAnswerWithPlayerName
    })
})

function answerClick(answer: TradeAnswerWithPlayerName) {
    if (answer.state != TradeAnswerState.ACCEPTED) {
        return
    }
    emit('choseTradePlayer', answer.playerId)
}

const emit = defineEmits({
    choseTradePlayer(_playerId: string) {
        return true
    },
})

const showDialog = computed(() => {
    if (!props.playerTradeOffer) {
        return false
    }
    if (props.playerTradeOffer.playerId != props.localPlayerId) {
        return false
    }
    return true
})

const props = defineProps({
    playerTradeOffer: {
        type: Object as PropType<CatanPlayerTradeOffer | undefined>,
        required: true
    },
    localPlayerId: {
        type: String,
        required: true
    },
    players: {
        type: Object as PropType<Player[]>,
        required: true
    }
})


</script>

<style>
.resource-icon {
    width: 2rem;
    max-width: 2rem;
}
</style>