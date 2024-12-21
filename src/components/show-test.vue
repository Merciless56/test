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
  <div class="test-container">
    <h1 class="title">Тесты</h1>

    <div v-if="!isTestCompleted" class="controls">
      <div class="control-group">
        <label for="test-count">Количество вопросов:</label>
        <input
            id="test-count"
            type="number"
            v-model.number="numberOfTests"
            min="10"
            max="100"
            class="input-field"
        />
      </div>

      <div class="control-group">
        <label for="group-select">Выберите группу:</label>
        <Select
            id="group-select"
            v-model="numberOfGroups"
            :options="groupOptions"
            optionLabel="label"
            placeholder="Выберите группу"
            class="dropdown"
        />
      </div>

      <Button @click="startTest" label="Начать тест" class="start-button" />
    </div>

    <div v-if="!isTestCompleted" class="questions-list">
      <ul>
        <li
            v-for="(question, index) in shuffledQuestions"
            :key="question.question + renderKey"
            class="question-item"
        >
          <h2 class="question">{{ question.question }}</h2>
          <ul class="options-list">
            <li
                v-for="option in question.options"
                :key="option + renderKey"
                class="option-item"
            >
              <input
                  type="radio"
                  :name="question.question + renderKey"
                  :value="option"
                  @change="handleAnswerSelection(question, option)"
                  :disabled="selectedAnswers[question.question]"
                  class="radio-button"
              />
              <label class="option-label">{{ option }}</label>
            </li>
          </ul>
          <p
              v-if="feedback[question.question]"
              class="feedback"
              :class="{ correct: feedback[question.question] === 'Правильно!', incorrect: feedback[question.question] === 'Неправильно' }"
          >
            {{ feedback[question.question] }}
          </p>
        </li>
      </ul>
    </div>

    <div v-else class="result-container">
      <h2>Тест завершён!</h2>
      <p class="result-message">
        Вы правильно ответили на {{ correctAnswersCount }} из {{ numberOfTests }} вопросов.
      </p>
      <Button @click="startTest" label="Перезапустить" class="restart-button" />
    </div>
  </div>
</template>

<style scoped>
/* Основные стили */
.test-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

/* Стили для панели управления */
.controls {
  margin-bottom: 20px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-field {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.dropdown {
  width: 100%;
}

.start-button {
  align-self: center;
  background-color: #007ad9;
  color: white;
}

/* Стили для вопросов */
.questions-list {
  margin-top: 20px;
}

.question-item {
  margin-bottom: 20px;
  padding: 15px;
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.question {
  color: #444;
}

.options-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.option-item {
  margin: 5px 0;
}

.radio-button {
  width: 20px;
  height: 20px;
  margin-right: 10px;
  cursor: pointer;
}

.option-label {
  font-size: 18px; /* Увеличить размер шрифта для текста рядом с кнопками */
  cursor: pointer; /* Удобство для взаимодействия */
}


.feedback {
  margin-top: 10px;
  font-weight: bold;
}

.feedback.correct {
  color: green;
}

.feedback.incorrect {
  color: red;
}

/* Стили для результата */
.result-container {
  text-align: center;
}

.result-message {
  margin-top: 15px;
  font-size: 18px;
}

.restart-button {
  margin-top: 20px;
  background-color: #007ad9;
  color: white;
}
</style>

