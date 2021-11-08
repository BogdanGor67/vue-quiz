<template>
  <div v-if="getData">
    <div class="questions-container">
      <p v-html="questions[questionsIndex]" v-show="questionsIndex < questions.length" class="mainQuestion"></p>
      <ul class="answers-block" v-show="questionsIndex < questions.length">
        <li v-for="answer in answers[questionsIndex]" v-html="answer" @click="chooseAnswer(answer)" class="answer-item"></li>
      </ul>
      <div v-show="questionsIndex === questions.length" class="result-block">
          <p class="result">You result is {{rightAnswersCount}} right answers:</p>
        <ul class="answers-list">
         <li v-for="(stateAnswer, i) in yourAnswers" v-html="stateAnswer.answer" :key="stateAnswer.id" :class="{correct: stateAnswer.answer === correctAnswers[i], wrong: stateAnswer.answer !== correctAnswers[i]}" class="answers-list-item"></li>
       </ul>
      </div>
    </div>
  </div>
  <E404 v-else />
</template>

<script>
  
  import axios from 'axios';
  import E404 from '@/components/E404';

  const baseURL = 'https://opentdb.com/api.php?amount=10&type=multiple';
  const answersURL = 'http://localhost:3001/answers';

export default {
  name: 'App',
  components: {
    E404,
  },
  data() {
    return {
      getData: true,
      questionsIndex: null,
      questions: [],
      answers: [],
      correctAnswers: [],
      yourAnswers: [],
    }
  },
  computed: {
    rightAnswersCount() {
      if (this.questionsIndex === this.questions.length) {
        let n = 0;
        this.yourAnswers.forEach((item, i) => {
          if (item.answer === this.correctAnswers[i]) {
            n++;
          }
        });
        return n;
      }
    }
  },
  methods: {
    increaseQuestionsIndex() {
      if (this.questionsIndex < this.questions.length) {
        this.questionsIndex++;
      }
    },
    async chooseAnswer(answer) {
      if (this.questionsIndex < this.questions.length) {
        try {
          await axios.post(answersURL, {answer: answer})
            .then(() => this.increaseQuestionsIndex());
        } catch(e) {
          console.error(e);
        }
      }
    },
    async getAnswers() {
      try {
        let currentAnswers = await axios.get(answersURL);

        this.yourAnswers = currentAnswers.data;
      } catch(e) {
          console.error(e);
      }
    }
  },
  watch: {
    questionsIndex() {
      if (this.questionsIndex === this.questions.length) {
        this.getAnswers();
      }
    }
  },
  async created() {
    try {
      let response = await axios.get(baseURL);

      this.questions = response.data.results.reduce((acc, item) => {
        acc.push(item.question);
        return acc;
      }, []);
      this.questionsIndex = 0;
      this.answers = response.data.results.reduce((acc, item) => {
        let answers = [];
        answers.push(item.correct_answer);
        item.incorrect_answers.forEach(incorrectAnswer => answers.push(incorrectAnswer));
        acc.push(answers.sort());
        return acc;
      }, []);
      this.correctAnswers = response.data.results.reduce((acc, item) => {
        acc.push(item.correct_answer);
        return acc;
      }, []);

    } catch(e) {
      console.error(e);
      this.getData = false;
    }
  }
};

</script>

<style>

  *,
  html,
  body {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .questions-container {
    min-width: 100vw;
    min-height: 100vh;
    padding-top: 100px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
  }

  .mainQuestion {
    max-width: 600px;
    text-align: center;
    padding: 30px;
    font-size: 36px;
    border: 2px solid lightgrey;
  }

  .answers-block {
    margin-top: 50px;
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
    justify-content: space-between;
  }

  .answer-item {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    list-style: none;
    padding: 20px;
    font-size: 24px;
    cursor: pointer;
    border: 2px solid lightgrey;
  }

  .result {
    font-size: 36px;
    text-align: center;
    margin-bottom: 20px;
  }

  .answers-list {
    font-size: 18px;
    text-align: center;
  }

  .answers-list-item {
    font-size: 18px;
    margin: 5px 0;
    list-style: none;
  }

  .correct {
    color: green;
  }

  .wrong {
    color: red;
    text-decoration: line-through;
  }

  .end-btn {
    min-width: 300px;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px 0;
    font-size: 18px;
    margin: 50%;
    transform: translateX(-50%);
    margin-top: 20px;
    cursor: pointer;
    border-radius: 15px;
    background: #666565;
    border: none;
    text-transform: uppercase;
    color: #fff;
  }

</style>
