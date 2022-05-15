<script setup>
import { computed } from '@vue/reactivity';
import { reactive, ref, watch } from 'vue'
const props = defineProps(['history', 'showIndex'])
const maxTime = ref(Math.max(...props.history.map(item => item.time)))
const maxLetters = ref(Math.max(...props.history.map(item => item.word.length)))
const maxTimePerLetter = ref(Math.max(...props.history.map(item => item.timePerLetter)))
const averageTimePerLetter = ref(Math.round(props.history.map(item => item.timePerLetter).reduce((a, b) => a + b, 0) / props.history.length))
const maxIndex = ref(props.history.length - 1)
watch(() => props.history, (newValue) => {
	maxTime.value = Math.max(...newValue.map(item => item.time))
	maxLetters.value = Math.max(...newValue.map(item => item.word.length))
	maxTimePerLetter.value = Math.max(...newValue.map(item => item.timePerLetter))
	maxIndex.value = newValue.length - 1
	averageTimePerLetter.value = Math.round(newValue.map(item => item.timePerLetter).reduce((a, b) => a + b, 0) / newValue.length)
})
const statIndex = ref(-1)
const averageLineStyle = computed(() => {
	return {
		"--bottom": averageTimePerLetter.value / maxTimePerLetter.value * 100 + 'vh'
	}
})
</script>

<template>
	<div class="plot">
		<div class="point" v-for="item, index in history"
			:style="{ left: index / maxIndex * 100 + '%', bottom: item.timePerLetter / maxTimePerLetter * 100 + '%', animationDelay: Math.random() * 1000 + 'ms', transitionDelay: index / maxIndex * 100 + 'ms' }"
			@mouseenter="statIndex = index" @click="$emit('wordClick', item.word)">
		</div>
		<div class="averageLine" v-if="averageTimePerLetter" :style="averageLineStyle">{{
				Math.round(averageTimePerLetter)
		}}ms/letter</div>
		<div class="stats" v-if="statIndex > -1 && props.history">
			<span style="font-size: 32px; font-weight: bold;">{{ props.history[statIndex].word }}</span>
			<div style="display: flex; flex-direction: column; gap: 0px;">
				<span v-if="showIndex">{{ statIndex }}</span>
				<span>{{ props.history[statIndex].time }}ms</span>
				<span>{{ props.history[statIndex].timePerLetter }}ms/letter</span>
			</div>
		</div>
	</div>
</template>

<style>
@keyframes fadeIn {
	from {
		opacity: 0;
	}
}

@keyframes slideFromBottom {
	from {
		transform: translateY(var(--bottom));
		opacity: 0;
	}
}

.plot {
	--point-size: 48px;
	width: calc(100% - var(--point-size));
	height: calc(100% - var(--point-size));
	bottom: 0;
	left: 0;
	position: absolute;
	display: flex;
	align-items: center;
	justify-content: center;
}

.point {
	width: var(--point-size);
	height: var(--point-size);
	box-sizing: border-box;
	border: 2px solid #0F04;
	border-radius: calc(var(--point-size) * 0.5);
	position: absolute;
	transition: 240ms cubic-bezier(0, 0.74, 0.25, 1) all, 240ms cubic-bezier(0, 0.74, 0.25, 1) left, 240ms cubic-bezier(0, 0.74, 0.25, 1) bottom;
	animation: fadeIn 500ms backwards;
	z-index: 15;
}

.point:hover {
	border-color: #0F0B;
	background-color: #0F02;
}

.point:active {
	border-width: calc(var(--point-size) * 0.4);
}

.stats {
	position: absolute;
	bottom: 0;
	left: auto;
	right: auto;
	padding: 16px;
	display: flex;
	align-items: center;
	gap: 12px;
	opacity: 0.8;
	z-index: 10;
}

.averageLine {
	width: 100vw;
	color: #0F0A;
	padding: 4px;
	border-top: 2px solid #0F02;
	position: absolute;
	left: 0;
	bottom: var(--bottom);
	transform: translateY(100%);
	transition: 500ms cubic-bezier(0.07, 0.56, 0.1, 0.99);
	animation: slideFromBottom 1s backwards cubic-bezier(0.07, 0.56, 0.1, 0.99);
}
</style>