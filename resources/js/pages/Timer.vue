<template>
    <div class="base-timer">
        <svg class="base-timer__svg" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
            <g class="base-timer__circle">
                <circle class="base-timer__path-elapsed" cx="50" cy="50" r="45"></circle>
                <path :stroke-dasharray="circleDasharray" class="base-timer__path-remaining" :class="remainingPathColor"
                    d="
              M 50, 50
              m -45, 0
              a 45,45 0 1,0 90,0
              a 45,45 0 1,0 -90,0
            "></path>
            </g>
        </svg>
        <span class="base-timer__label">{{ formattedTimeLeft }}</span>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';

const FULL_DASH_ARRAY = 283;
const WARNING_THRESHOLD = 10;
const ALERT_THRESHOLD = 5;

const COLOR_CODES = {
    info: {
        color: "green"
    },
    warning: {
        color: "orange",
        threshold: WARNING_THRESHOLD
    },
    alert: {
        color: "red",
        threshold: ALERT_THRESHOLD
    }
};

const TIME_LIMIT = 3600;

const timePassed = ref(0);
const timerInterval = ref<number | null>(null);

const circleDasharray = computed(() => {
    return `${(timeFraction.value * FULL_DASH_ARRAY).toFixed(0)} 283`;
});

const formattedTimeLeft = computed(() => {
    const timeLeft = TIME_LIMIT - timePassed.value;
    const minutes = Math.floor(timeLeft / 60);
    let seconds = timeLeft % 60;

    if (seconds < 10) {
        seconds = `0${seconds}`;
    }

    return `${minutes}:${seconds}`;
});

const timeFraction = computed(() => {
    const rawTimeFraction = (TIME_LIMIT - timePassed.value) / TIME_LIMIT;
    return rawTimeFraction - (1 / TIME_LIMIT) * (1 - rawTimeFraction);
});

const remainingPathColor = computed(() => {
    const { alert, warning, info } = COLOR_CODES;
    const timeLeft = TIME_LIMIT - timePassed.value;

    if (timeLeft <= alert.threshold) {
        return alert.color;
    } else if (timeLeft <= warning.threshold) {
        return warning.color;
    } else {
        return info.color;
    }
});

const onTimesUp = () => {
    if (timerInterval.value) {
        clearInterval(timerInterval.value);
    }
};

const startTimer = () => {
    timerInterval.value = setInterval(() => {
        timePassed.value += 1;
        if (timePassed.value >= TIME_LIMIT) {
            onTimesUp();
        }
    }, 1000);
};

onMounted(() => {
    startTimer();
});

onUnmounted(() => {
    if (timerInterval.value) {
        clearInterval(timerInterval.value);
    }
});
</script>

<style scoped>
.base-timer {
    position: relative;
    width: 120px;
    height: 120px;
}

.base-timer__svg {
    transform: scaleX(-1);
}

.base-timer__circle {
    fill: none;
    stroke: none;
}

.base-timer__path-elapsed {
    stroke-width: 7px;
    stroke: grey;
}

.base-timer__path-remaining {
    stroke-width: 7px;
    stroke-linecap: round;
    transform: rotate(90deg);
    transform-origin: center;
    transition: 1s linear all;
    fill-rule: nonzero;
    stroke: currentColor;
}

.base-timer__path-remaining.green {
    color: rgb(65, 184, 131);
}

.base-timer__path-remaining.orange {
    color: orange;
}

.base-timer__path-remaining.red {
    color: red;
}

.base-timer__label {
    position: absolute;
    width: 121px;
    height: 119px;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 25px;
}
</style>