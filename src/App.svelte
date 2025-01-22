<script>
  // @ts-nocheck
  import { quintIn } from "svelte/easing";
  import Confetti from "svelte-confetti"; // For confetti animation

  let computerArchitecture =
    "https://sb-api-flame.vercel.app/quiz/computer_architecture";
  let questions = $state([]); // Initialize as an empty array
  let userAnswers = $state({}); // Store user's answers
  let currentQuestionIndex = $state(0); // Track the current question index
  let score = $state({ correct: 0, incorrect: 0 }); // Track the user's score
  let showConfetti = $state(false); // Control confetti animation
  let timeLeft = $state(15); // Timer for each question (15 seconds)
  let isQuizComplete = $state(false); // Track if the quiz is complete

  // Sound effects
  const correctSound = new Audio(
    "https://assets.mixkit.co/active_storage/sfx/2217/2217-preview.mp3"
  );
  const incorrectSound = new Audio(
    "https://assets.mixkit.co/active_storage/sfx/1087/1087-preview.mp3"
  );

  // Fetch questions
  $effect(() => {
    try {
      fetch(computerArchitecture)
        .then((res) => res.json())
        .then((data) => {
          questions = data[1]["quiz"];
          console.log(data[1]["quiz"]);
        })
        .catch((error) => console.error(error));
    } catch (error) {
      console.error(error);
    }
  });

  // Timer logic
  $effect(() => {
    if (currentQuestionIndex < questions.length) {
      const timer = setInterval(() => {
        if (timeLeft > 0) {
          timeLeft--;
        } else {
          // Time's up! Move to the next question
          handleSelection(null); // No answer selected
          clearInterval(timer);
        }
      }, 1000);

      return () => clearInterval(timer); // Cleanup timer
    }
  });

  // Function to handle user selection and move to the next question
  function handleSelection(selectedOption) {
    if (selectedOption !== null) {
      userAnswers[currentQuestionIndex] = selectedOption; // Save the user's answer

      // Update score and play sound
      if (selectedOption === questions[currentQuestionIndex]?.answer) {
        score.correct++;
        showConfetti = true; // Trigger confetti animation
        correctSound.play();
        setTimeout(() => (showConfetti = false), 2000); // Hide confetti after 2 seconds
      } else {
        score.incorrect++;
        incorrectSound.play();
      }
    }

    // Move to the next question after a short delay (e.g., 1 second)
    setTimeout(() => {
      if (currentQuestionIndex < questions.length - 1) {
        currentQuestionIndex++; // Move to the next question
        timeLeft = 15; // Reset timer for the next question
      } else {
        // Quiz is complete
        isQuizComplete = true;
      }
    }, 1000); // 1-second delay before moving to the next question
  }

  // Function to restart the quiz
  function restartQuiz() {
    currentQuestionIndex = 0;
    userAnswers = {};
    score = { correct: 0, incorrect: 0 };
    timeLeft = 15;
    isQuizComplete = false;
  }
</script>

<main class="container">
  {#if showConfetti}
    <Confetti />
  {/if}

  <h1 class="text-center">COSTUDY</h1>

  {#if questions.length > 0}
    {#if !isQuizComplete}
      <div class="quiz-wrapper">
        <!-- Progress Bar -->
        <div class="progress-bar">
          <div
            class="progress"
            style={`width: ${((currentQuestionIndex + 1) / questions.length) * 100}%`}
          ></div>
        </div>

        <!-- Timer -->
        <div class="timer">⏳ Time Left: {timeLeft}s</div>

        <!-- Question Card -->
        <div class="question-card">
          <p class="question-text">
            {questions[currentQuestionIndex].question}
          </p>
          <div class="options">
            {#each Object.entries(questions[currentQuestionIndex].options) as [key, value]}
              <label class="option">
                <input
                  type="radio"
                  name={`quiz-${currentQuestionIndex}`}
                  value={key}
                  checked={userAnswers[currentQuestionIndex] === key}
                  on:change={() => handleSelection(key)}
                />
                <span class="option-key">{key}:</span>
                {value}
              </label>
            {/each}
          </div>
          {#if userAnswers[currentQuestionIndex] !== undefined}
            <div class="feedback">
              {#if userAnswers[currentQuestionIndex] === questions[currentQuestionIndex].answer}
                <span class="correct">✅ Correct!</span>
              {:else}
                <span class="incorrect">
                  ❌ Incorrect. The correct answer is {questions[
                    currentQuestionIndex
                  ].answer}.
                </span>
              {/if}
            </div>
          {/if}
        </div>

        <!-- Score Tracker -->
        <div class="score">
          🏆 Correct: {score.correct} | ❌ Incorrect: {score.incorrect}
        </div>
      </div>
    {:else}
      <!-- Quiz Completion Screen -->
      <div class="completion-screen">
        <h2>🎉 Quiz Complete! 🎉</h2>
        <p>Your final score is:</p>
        <p class="final-score">
          ✅ Correct: {score.correct} | ❌ Incorrect: {score.incorrect}
        </p>
        <button on:click={restartQuiz} class="restart-button">
          Restart Quiz
        </button>
      </div>
    {/if}
  {:else}
    <p class="loading">Loading questions...</p>
  {/if}
  <br />

  <div>
    <p>Made with 💜 by Unrealrojo and AI</p>
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: "Poppins", sans-serif;
    background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
    color: #333;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }

  .container {
    max-width: 800px;
    width: 90%;
    margin: 20px;
    padding: 30px;
    background-color: rgba(255, 255, 255, 0.9);
    border-radius: 12px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
  }

  h1.text-center {
    text-align: center;
    font-size: 2.5rem;
    color: #2c3e50;
    margin-bottom: 30px;
  }

  .quiz-wrapper {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .progress-bar {
    width: 100%;
    height: 8px;
    background-color: #e0e0e0;
    border-radius: 4px;
    overflow: hidden;
  }

  .progress-bar .progress {
    height: 100%;
    background-color: #6a11cb;
    transition: width 0.3s ease;
  }

  .timer {
    font-size: 1.2rem;
    font-weight: bold;
    color: #dc3545;
    text-align: center;
  }

  .question-card {
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    padding: 20px;
    animation: fadeIn 0.5s ease-in-out;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .question-text {
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 15px;
    color: #2c3e50;
  }

  .options {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .option {
    display: flex;
    align-items: center;
    background-color: #f9f9f9;
    border: 1px solid #e0e0e0;
    border-radius: 4px;
    padding: 10px;
    font-size: 1rem;
    color: #555;
    transition:
      transform 0.2s ease,
      box-shadow 0.2s ease;
    cursor: pointer;
  }

  .option:hover {
    transform: translateY(-3px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .option input[type="radio"] {
    margin-right: 10px;
    cursor: pointer;
  }

  .option-key {
    font-weight: bold;
    color: #2c3e50;
    margin-right: 8px;
  }

  .feedback {
    margin-top: 15px;
    font-size: 1rem;
    font-weight: bold;
  }

  .correct {
    color: #28a745; /* Green for correct answers */
    animation: pulse 0.5s ease-in-out;
  }

  .incorrect {
    color: #dc3545; /* Red for incorrect answers */
  }

  @keyframes pulse {
    0% {
      transform: scale(1);
    }
    50% {
      transform: scale(1.1);
    }
    100% {
      transform: scale(1);
    }
  }

  .score {
    font-size: 1rem;
    color: #2c3e50;
    text-align: center;
  }

  .completion-screen {
    text-align: center;
    padding: 20px;
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  .final-score {
    font-size: 1.5rem;
    font-weight: bold;
    color: #2c3e50;
  }

  .restart-button {
    padding: 10px 20px;
    font-size: 1rem;
    color: #fff;
    background: linear-gradient(135deg, #6a11cb, #2575fc);
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition:
      transform 0.2s ease,
      box-shadow 0.2s ease;
  }

  .restart-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
  }

  .loading {
    text-align: center;
    font-size: 1.2rem;
    color: #666;
  }
</style>
