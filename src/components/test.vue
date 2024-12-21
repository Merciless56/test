<script setup>
import { ref } from "vue";
import Textarea from "primevue/textarea";
import Button from "primevue/button"; // Make sure to import the Button component

const inputFile = ref(""); // Initialize as an empty string to store the text input
const questions = ref([]); // Use `ref` so it's reactive
let currentQuestion = {};
let buffer = "";

function createTest() {
  const lines = inputFile.value.split("\n"); // Split the input by lines
  questions.value = []; // Reset questions for new input
  currentQuestion = {}; // Reset current question for fresh parsing
  buffer = ""; // Reset buffer

  lines.forEach((line) => {
    const trimmedLine = line.trim();

    if (trimmedLine.startsWith("?")) {
      if (Object.keys(currentQuestion).length > 0) {
        if (buffer) currentQuestion.question += " " + buffer.trim();
        questions.value.push(currentQuestion);
      }

      currentQuestion = {question: trimmedLine.slice(1).trim(), correctAnswer: null, options: []};
      buffer = "";
    } else if (trimmedLine.startsWith("+")) {
      if (buffer) currentQuestion.question += " " + buffer.trim();
      buffer = "";
      currentQuestion.correctAnswer = trimmedLine.slice(1).trim();
      currentQuestion.options.push(currentQuestion.correctAnswer);
    } else if (trimmedLine.startsWith("=")) {
      if (buffer) currentQuestion.question += " " + buffer.trim();
      buffer = "";
      currentQuestion.options.push(trimmedLine.slice(1).trim());
    } else if (trimmedLine) {
      buffer += " " + trimmedLine; // Append multi-line content
    }
  });

  if (Object.keys(currentQuestion).length > 0) {
    if (buffer) currentQuestion.question += " " + buffer.trim();
    questions.value.push(currentQuestion); // Add the last question
  }
  console.log(questions )
}
</script>

<template>
  <div>
    <Textarea
        v-model="inputFile"
        rows="5"
        cols="30"
        placeholder="Paste your questions here..."
    />
    <Button @click="createTest" label="Submit"/>
  </div>
</template>

<style scoped>
/* Optional styling */
</style>
