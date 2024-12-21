<script setup>
import { list } from "../questions.js";
import { ref, computed } from "vue";
import { Select } from "primevue";
import Button from "primevue/button";
const selectedAnswers = ref({});
const feedback = ref({});
const numberOfTests = ref(30);
const numberOfGroups = ref({ value: 1 });
const groupCount = ref(0);
const groupedQuestions = ref([]);
const shuffledQuestions = ref([]);
const isTestCompleted = ref(false);
const correctAnswersCount = ref(0);
const renderKey = ref(0); // Ключ для полного ререндеринга

// Опции для выпадающего списка
const groupOptions = computed(() => {
  const options = [];
  for (let i = 1; i <= groupCount.value; i++) {
    if (i === 1) {
      options.push({ value: i, label: `${i} - ${i * numberOfTests.value}` });
    } else {
      options.push({
        value: i,
        label: `${(i - 1) * numberOfTests.value} - ${i * numberOfTests.value}`,
      });
    }
  }
  return options;
});

// Фильтрация вопросов для текущей группы
function filteredQuestions() {
  const start = (numberOfGroups.value.value - 1) * numberOfTests.value;
  const end = start + numberOfTests.value;

  groupedQuestions.value = list.slice(start, end).map((question) => ({
    ...question,
    options: shuffleArray([...question.options]),
  }));

  shuffledQuestions.value = shuffleArray([...groupedQuestions.value]);
  isTestCompleted.value = false;
  correctAnswersCount.value = 0;
  selectedAnswers.value = {};
  feedback.value = {};
  renderKey.value += 1; // Обновляем ключ, чтобы сбросить checked
}

// Инициализация теста
function startTest() {
  groupCount.value = Math.ceil(list.length / numberOfTests.value);
  filteredQuestions();
}

// Подсчёт результата
function calculateResult() {
  correctAnswersCount.value = groupedQuestions.value.reduce((count, question) => {
    if (selectedAnswers.value[question.question] === question.correctAnswer) {
      return count + 1;
    }
    return count;
  }, 0);
  isTestCompleted.value = true;
}

// Перемешивание массива
function shuffleArray(array) {
  return array.sort(() => Math.random() - 0.5);
}

// Обработка выбора ответа
function handleAnswerSelection(question, selectedOption) {
  if (!selectedAnswers.value[question.question]) {
    selectedAnswers.value[question.question] = selectedOption;
    if (Object.keys(selectedAnswers.value).length === groupedQuestions.value.length) {
      calculateResult();
    }
    if (selectedOption === question.correctAnswer) {
      feedback.value[question.question] = "Правильно!";
    } else {
      feedback.value[question.question] = "Неправильно";
    }
  }
}
</script>

<template>
  <div>
    <h1>Questions</h1>
    <div v-if="!isTestCompleted">
      <label for="test-count">Показать по:</label>

      <input
          id="test-count"
          type="number"
          v-model.number="numberOfTests"
          min="10"
          max="100"
      />
      <br></br>
      <Select
          v-model="numberOfGroups"
          :options="groupOptions"
          optionLabel="label"
          placeholder="Select a Group"
          class="w-full md:w-56"
      />
      <br>

      <Button @click="startTest">Start</Button>
    </div>
    <div v-if="!isTestCompleted">
      <ul>
        <li v-for="(question, index) in shuffledQuestions" :key="question.question + renderKey">
          <h2>{{ question.question }}</h2>
          <ul>
            <li v-for="option in question.options" :key="option + renderKey">
              <input
                  type="radio"
                  :name="question.question + renderKey"
                  :value="option"
                  @change="handleAnswerSelection(question, option)"
                  :disabled="selectedAnswers[question.question]"
              />
              <label>{{ option }}</label>
            </li>
          </ul>
          <p v-if="feedback[question.question]">{{ feedback[question.question] }}</p>
        </li>
      </ul>
    </div>
    <div v-else>
      <h2>Test Completed!</h2>
      <p>Вы правильно ответили на {{ correctAnswersCount }} вопроса из {{ numberOfTests   }}.</p>
      <button @click="startTest">Restart</button>
    </div>
  </div>
</template>

<style scoped>
/* Optional styling */
</style>
