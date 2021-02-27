<script>
  import _ from "lodash";

  let categories = [
    { id: 0, text: "Category" },
    { id: 18, text: "Science: Computers" },
    { id: 23, text: "History" },
    { id: 21, text: "Sports" },
    { id: 12, text: "Music"}
  ];

  let difficulties = ["Difficulty", "easy", "medium", "hard"];

  let category = "";
  let difficulty = "";
  let result;
  let count = 0;

  $: if (count > 9) {
    alert("Quiz is over. Please refresh to play again.");
    count = 10;
  }

  $: if (count < 0) {
    count = 0;
  }

  function handleNext() {
    count += 1;
    isAnswered = false;
  }

  function handleBack() {
    count -= 1;
  }

  async function fetchQuestions() {
    let response = await fetch(
      `https://opentdb.com/api.php?amount=10&category=${category.id}&difficulty=${difficulty}`
    );

    let result = await response.json();
    let questions = result.results;

    let data = [];

    questions.forEach((question) => {
      let answers = [];

      question.incorrect_answers.forEach((answer) => {
        answers.push({ answer: answer, correct: false });
      });

      answers.push({ answer: question.correct_answer, correct: true });

      let shuffle = _.shuffle(answers);

      data.push({ question: question.question, answers: shuffle });
    });
    
    return { questions, data };
  }

  let isCorrect;
  let isAnswered = false;
  let numOfCorr = 0;

  function checkAnswer(correct) {
    isAnswered = true;
    isCorrect = correct;

    if(isCorrect === true) {
      console.log('correct answer')
      numOfCorr += 1
      console.log(numOfCorr)
    } else {
      console.log('wrong answer')
    }

    return numOfCorr
  }


  function handleSubmit(e) {
    if (category.text === "Category") {
      alert("Please select a category.");
    } else {
      result = fetchQuestions();
    }
  }
</script>

<main>
  <h1>Quiz</h1>
  <div>
    <form on:submit|preventDefault={handleSubmit}>
      <select bind:value={category}>
        {#each categories as category}
          <option value={category}>
            {category.text}
          </option>
        {/each}
      </select>
      <select bind:value={difficulty}>
        {#each difficulties as difficulty}
          <option value={difficulty}>
            {difficulty}
          </option>
        {/each}
      </select>
      <button class="get-started" type="submit"> Get Started </button>
    </form>
  </div>
  <div class="content">
    {#if result === undefined}
      <p />
    {:else}
      {#await result}
        <p>...Waiting</p>
      {:then { data }}
        <div class="question">
          <p>{@html _.unescape(data[count].question)}</p>
        </div>
        <div class="answers">
          <div class="feedback">
            <h3 class="correct">{numOfCorr}/10 correct</h3>

            {#if isAnswered}
              <h3>{isCorrect ? "Correct!" : "Wrong!"}</h3>
            {/if}
          </div>
          {#each data[count].answers as answers}
            <button class="answer" on:click={() => checkAnswer(answers.correct)}
              >{@html _.unescape(answers.answer)}</button
            >
          {/each}
        </div>

        <div class="navigation-btns">
          <button class="navigate" on:click={handleBack}>BACK</button>
          <p>{count + 1}/10</p>
          <button class="navigate" on:click={handleNext}>NEXT</button>
        </div>
      {:catch error}
        {error.message}
      {/await}
    {/if}
  </div>
</main>

<style>
  main {
    text-align: center;
    margin: 0 auto;
    max-width: 600px;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 600;
    text-shadow: 2px 2px #0000002e;
  }

  select {
    border: 1px solid black;
    background-color: transparent;
    padding: 10px 15px;
    margin: 0px 10px;
  }

  .get-started {
    border: 1px solid black;
    background-color: black;
    color: #f9ba45;
    padding: 10px 15px;
    margin: 0px 10px;
  }

  .get-started:hover {
    background-color: #f9ba45;
    color: black;
  }

  button:hover {
    cursor: pointer;
  }

  .content {
    display: flex;
    flex-direction: column;
    
  }

  .feedback {
    display: flex;
    justify-content: space-between;
  }

  .correct {

  }

  .answers {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .answer {
    background-color: transparent;
    border: 1px solid black;
    padding: 20px;
    border-radius: 25px;
    margin: 10px;
    font-size: 18px;
  }

  .answer:hover {
    background-color: black;
    color: #f9ba45;
  }

  .question {
    font-size: 24px;
    color: #ff3e00;
    text-shadow: 1px 1px #0000002e;
    height: 120px;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .navigation-btns {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .navigate {
    background-color: transparent;
    margin: 20px 10px;
    border: 1px solid black;
  }
  .navigate:hover {
    background-color: black;
    color: #f9ba45;
  }
</style>
