<template>
  <div>
    <template v-if="this.question || this.numberRounds < 6">
      <ScoreBoard
        :loseCount="this.loseCount"
        :winCount="this.winCount"
        :round="this.numberRounds"
      />

      <div class="main">
        <h1 v-html="this.question"></h1>

        <template v-for="(answer, index) in this.answers" :key="index">
          <div class="inputRadio">
            <input
              :disabled="this.answerSubmited"
              type="radio"
              :id="index"
              name="options"
              :value="answer"
              v-model="this.chosenAnswer"
            />
            <label :for="index" v-html="answer"></label>
          </div>
        </template>

        <button
          v-if="!this.answerSubmited"
          @click="this.submitAnswer()"
          class="send"
          type="button"
        >
          Send
        </button>

        <section v-if="this.answerSubmited" class="result">
          <button class="send" type="button" @click="this.getNextQuestion()">
            Next Question
          </button>
          <h4 v-if="this.chosenAnswer == this.correctAnswer">
            &#9989; Congratulations, the answer "<span
              v-html="this.correctAnswer"
            ></span
            >" is correct!
          </h4>
          <h4 v-else>
            &#10060; I'm sorry, you picked the wrong answer. The correct is
            "<span v-html="this.correctAnswer"></span>"
          </h4>
        </section>
      </div>
    </template>
    <template v-else>
      <h1 v-if="this.winCount > this.loseCount">You Win!</h1>
      <h1 v-else-if="this.winCount < this.loseCount">You Lose!</h1>
      <button class="send" type="button" @click="this.restartGame()">
        Restart
      </button>
    </template>
  </div>
</template>

<script>
import ScoreBoard from "./components/ScoreBoard.vue";

export default {
  name: "App",
  components: {
    ScoreBoard,
  },
  data() {
    return {
      question: undefined,
      incorrectAnswers: undefined,
      correctAnswer: undefined,
      chosenAnswer: undefined,
      answerSubmited: false,
      winCount: 0,
      loseCount: 0,
      numberRounds: 0,
    };
  },

  computed: {
    answers() {
      let answers = JSON.parse(JSON.stringify(this.incorrectAnswers));
      answers.splice(
        parseInt(Math.random() * (answers.length + 1)),
        0,
        this.correctAnswer
      );
      return answers;
    },
  },

  methods: {
    submitAnswer() {
      if (!this.chosenAnswer) {
        alert("Escolha uma resposta!");
      } else {
        this.answerSubmited = true;
        if (this.chosenAnswer == this.correctAnswer) {
          this.winCount++;
        } else {
          this.loseCount++;
        }
      }
    },

    restartGame(){
      this.question = undefined;
      this.incorrectAnswers = undefined
      this.correctAnswer = undefined;
      this.chosenAnswer = undefined;
      this.answerSubmited = false;
      this.winCount = 0;
      this.loseCount = 0;
      this.numberRounds = 0;

      this.getNextQuestion();

    },

    getNextQuestion() {
      this.answerSubmited = false;
      this.chosenAnswer = undefined;
      this.question = undefined;
      this.numberRounds++;

      if (this.numberRounds < 6) {
        fetch("https://opentdb.com/api.php?amount=1&category=21")
          .then((res) => res.json())
          .then((json) => {
            this.question = json.results[0].question;
            this.incorrectAnswers = json.results[0].incorrect_answers;
            this.correctAnswer = json.results[0].correct_answer;
          })
          .catch((err) => console.log(err));        
      }
    },
  },

  created() {
    this.getNextQuestion();
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 960px;

  .main {
    padding: 20px;
  }

  #input[type="radio"] {
    margin: 22px 4px;
  }

  label {
    cursor: pointer;
    padding: 20px;
  }

  .inputRadio {
    margin: 30px auto;
  }

  button.send {
    margin-top: 12px;
    height: 40px;
    min-width: 120px;
    padding: 0 16px;
    color: #fff;
    background-color: #1867c0;
    border: 1px solid #1867c0;
    cursor: pointer;
  }
}
</style>
