<template>
  <main>
    <h2 :class="{ hidden: didQuizEnd }">
      Question {{ currentQuizId }} out of {{ quizzes.length }}
    </h2>
    <h1 :class="{ shown: didQuizEnd }" class="correctCount">
      Correct answers {{ correctCount }} out of {{ quizzes.length }}
    </h1>
    <div>
      <Question :quiz-ended="didQuizEnd" :question="question"></Question>
      <p>
        <Answers
          :quiz-ended="didQuizEnd"
          :answers="answers"
          @submit-answer="handleAnswerSubmission($event)"
        ></Answers>
      </p>
    </div>
  </main>
</template>

<script setup lang="ts">
import axios from "axios";
import { ref } from "vue";

// components
import Answers from "./Answers.vue";
import Question from "./Question.vue";

type Quiz = {
  question: string;
  correct_answer: string;
  incorrect_answers: string[];
};

const answers = ref<string[]>([]);
const question = ref<string>();
const currentQuizId = ref<number>(0);
const quizzes = ref<Quiz[]>([]);
const correctCount = ref<number>(0);
const didQuizEnd = ref<boolean>(false);
let correctAnswer: string;

function handleAnswerSubmission(answer: string) {
  console.log(question.value);
  console.log(`You clicked ${answer} , correct answer is ${correctAnswer}`);
  console.log(`-------------------------------------------------------------`);
  if (answer === correctAnswer) {
    correctCount.value++;
  }
  nextQuestion();
}
// function delay(time: number) {
//   return new Promise((resolve) => setTimeout(resolve, time));
// }
// delay(1000).then(() => nextQuestion());

async function getQuestionsData() {
  const response = await axios.get(
    "https://opentdb.com/api.php?amount=5&category=9&difficulty=easy&type=multiple"
  );
  quizzes.value = response.data.results;
  nextQuestion();
}

function nextQuestion() {
  if (currentQuizId.value === quizzes.value.length) {
    didQuizEnd.value = true;
    console.log("Quiz is done");
    console.log(
      `Correct answers ${correctCount.value} out of ${quizzes.value.length}`
    );
  } else {
    const currentQuiz = quizzes.value[currentQuizId.value];
    question.value = currentQuiz.question;
    correctAnswer = currentQuiz.correct_answer;
    answers.value = [...currentQuiz.incorrect_answers, correctAnswer];
    function shuffle(array: string[]) {
      array.sort(() => Math.random() - 0.5);
    }
    shuffle(answers.value);
    currentQuizId.value++;
  }
}
getQuestionsData();
</script>

<style>
.correctCount {
  font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
    "Lucida Sans Unicode", Geneva, Verdana, sans-serif;
  margin: 10px;
  visibility: hidden;
}

.shown {
  visibility: visible;
}

.hidden {
  visibility: hidden;
}
</style>
