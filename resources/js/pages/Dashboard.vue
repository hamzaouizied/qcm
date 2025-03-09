<script setup lang="ts">
import AppLayout from '@/layouts/AppLayout.vue';
import { type BreadcrumbItem } from '@/types';
import { Head } from '@inertiajs/vue3';
import { ref, computed, onMounted } from 'vue';
import Timer from "./Timer.vue";

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Dashboard',
        href: '/dashboard',
    },
];

const questions = ref([
    {
        text: "Quel est le langage principal de Vue.js ?",
        choices: ["JavaScript", "Python", "Ruby", "PHP"],
        answer: "JavaScript"
    },
    {
        text: "Quel hook est utilisé pour les effets secondaires dans Vue 3 ?",
        choices: ["onMounted", "onUpdated", "onUnmounted", "watchEffect"],
        answer: "watchEffect"
    }
]);

const userAnswers = ref<{ [key: number]: string }>({});
const submitted = ref(false);
const score = ref(0);
const currentQuestionIndex = ref(0);
const timeLeft = ref(3600); // 1 heure en secondes
const testFinished = ref(false);

const currentQuestion = computed(() => questions.value[currentQuestionIndex.value]);

const nextQuestion = () => {
    if (currentQuestionIndex.value < questions.value.length - 1) {
        currentQuestionIndex.value++;
    } else {
        submitQuiz();
    }
};

const submitQuiz = () => {
    score.value = questions.value.reduce((total, question, index) => {
        return total + (userAnswers.value[index] === question.answer ? 1 : 0);
    }, 0);
    submitted.value = true;
    testFinished.value = true;
};

const resetQuiz = () => {
    userAnswers.value = {};
    submitted.value = false;
    score.value = 0;
    currentQuestionIndex.value = 0;
    timeLeft.value = 3600;
    testFinished.value = false;
    startTimer();
};

const startTimer = () => {
    const timer = setInterval(() => {
        if (timeLeft.value > 0) {
            timeLeft.value--;
        } else {
            clearInterval(timer);
            submitQuiz();
        }
    }, 1000);
};

const previousQuestion = () => {
    if (currentQuestionIndex.value > 0) {
        currentQuestionIndex.value--;
    }
};
onMounted(startTimer);
</script>

<template>

    <Head title="Dashboard" />

    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="min-h-screen bg-gray-100 flex p-4">
            <div class="bg-white rounded-lg shadow-lg w-full p-6">
                <div class="flex flex-row">
                    <div class="basis-full">
                        <h1 class="text-2xl font-bold mb-6">Exam - 1</h1>
                    </div>
                    <div class="basis-full">
                        <Timer />
                    </div>
                </div>

                <!-- Question Section -->
                <div v-if="!testFinished">
                    <div v-if="!submitted">
                        <div
                            class="question bg-gray-50 p-6 rounded-lg shadow-sm hover:shadow-md transition-shadow duration-300">
                            <!-- Compteur de questions -->
                            <p class="text-lg font-semibold mb-4">
                                Question {{ currentQuestionIndex + 1 }} sur {{ questions.length }}
                            </p>
                            <p class="text-lg font-semibold mb-4">{{ currentQuestion.text }}</p>

                            <!-- Choices -->
                            <div v-for="(choice, i) in currentQuestion.choices" :key="i" class="choice mb-3">
                                <label
                                    class="flex items-center space-x-3 p-3 rounded-lg border border-gray-200 hover:bg-blue-50 cursor-pointer transition-colors duration-200">
                                    <input type="radio" :name="'question-' + currentQuestionIndex" :value="choice"
                                        v-model="userAnswers[currentQuestionIndex]"
                                        class="form-radio h-5 w-5 text-blue-600" />
                                    <span class="text-gray-700">{{ choice }}</span>
                                </label>
                            </div>

                            <!-- Navigation Buttons -->
                            <div class="flex justify-between mt-6">
                                <!-- Bouton "Précédent" masqué pour la première question -->
                                <button v-if="currentQuestionIndex > 0" @click="previousQuestion"
                                    class="bg-gray-600 text-white py-2 px-4 rounded-lg hover:bg-gray-700 transition-colors duration-200">
                                    Précédent
                                </button>
                                <!-- Espace vide pour aligner le bouton "Suivant" à droite -->
                                <div v-else></div>

                                <button @click="nextQuestion" :disabled="!userAnswers[currentQuestionIndex]"
                                    class="bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors duration-200 disabled:bg-gray-300 disabled:cursor-not-allowed">
                                    Suivant
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Result Section -->
                <div v-else class="text-center">
                    <h2 class="text-xl font-bold mb-4">Résultat : {{ score }} / {{ questions.length }}</h2>
                    <button @click="resetQuiz"
                        class="bg-green-600 text-white py-2 px-6 rounded-lg hover:bg-green-700 transition-colors duration-200">
                        Recommencer
                    </button>
                </div>
            </div>
        </div>
    </AppLayout>
</template>

<style scoped></style>
