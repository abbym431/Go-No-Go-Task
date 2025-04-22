# Go-No-Go-Task
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Go/No-Go Arrow Task</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@400;700&display=swap');
    body {
      font-family: 'Roboto Slab', serif;
      background: linear-gradient(135deg, #a7ffeb 25%, #d0f7f1 100%);
      color: #333;
      text-align: center;
      padding: 50px 20px;
      height: 100vh;
      margin: 0;
    }
    #stimulus {
      font-size: 180px;
      font-weight: 900;
      margin-bottom: 20px;
      color: #003366;
    }
    #message {
      font-size: 28px;
      font-weight: bold;
      margin-bottom: 20px;
    }
    #startBtn, #playAgainBtn {
      padding: 10px 20px;
      font-size: 18px;
      background-color: #28a745;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #startBtn:hover, #playAgainBtn:hover {
      background-color: #218838;
    }
    #instructions {
      font-size: 18px;
      margin-bottom: 30px;
      max-width: 700px;
      margin-left: auto;
      margin-right: auto;
      background: #ffffffcc;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }
    #counter {
      font-size: 22px;
      margin-bottom: 10px;
      font-weight: bold;
    }
    #results {
      margin-top: 30px;
      font-size: 18px;
    }
    table {
      margin: 0 auto;
      border-collapse: collapse;
    }
    table, th, td {
      border: 1px solid #333;
    }
    th, td {
      padding: 10px 20px;
    }
    th {
      background-color: #f2f2f2;
    }
  </style>
</head>
<body>
  <div id="instructions">
    <h2>Instructions</h2>
    <p>You will see arrows pointing <strong>up, down, left,</strong> or <strong>right</strong>.</p>
    <p><strong>Your task:</strong> Press the corresponding arrow key <strong>only if the arrow is NOT pointing down</strong>.</p>
    <p>If the arrow points <strong>down (↓)</strong>, <span style="color: red; font-weight: bold;">do NOT press any key</span>. The screen will advance automatically.</p>
    <p>Try to respond as quickly and accurately as possible. After 20 arrows, your results will be shown.</p>
  </div>

  <div id="counter"></div>
  <div id="stimulus"></div>
  <div id="message"></div>
  <button id="startBtn">Start</button>
  <div id="results"></div>
  <button id="playAgainBtn" style="display:none;">Play Again</button>

  <script>
    const directions = ['⇧', '⇩', '⇦', '⇨'];
    const directionKeys = {
      ArrowUp: '⇧',
      ArrowDown: '⇩',
      ArrowLeft: '⇦',
      ArrowRight: '⇨',
    };

    let consecutiveIncorrect = 0;
    let cooldown = false;
    let gameStarted = false;
    let waitingForResponse = false;
    let stimulusEl = document.getElementById('stimulus');
    let messageEl = document.getElementById('message');
    let startBtn = document.getElementById('startBtn');
    let instructionsEl = document.getElementById('instructions');
    let counterEl = document.getElementById('counter');
    let resultsEl = document.getElementById('results');
    let playAgainBtn = document.getElementById('playAgainBtn');

    let currentStimulus = '';
    let isGoStimulus = true;
    let responseStart = null;
    let responseData = [];
    let trialCount = 0;
    const totalTrials = 20;

    function getRandomStimulus() {
      return directions[Math.floor(Math.random() * directions.length)];
    }

    function showStimulus() {
      if (cooldown || trialCount >= totalTrials) return;

      currentStimulus = getRandomStimulus();
      isGoStimulus = currentStimulus !== '⇩';
      responseStart = performance.now();
      stimulusEl.textContent = currentStimulus;
      messageEl.textContent = 'Press the matching arrow key (unless it points down)';
      messageEl.style.color = '#555';
      waitingForResponse = true;
      counterEl.textContent = `Arrow ${trialCount + 1} of ${totalTrials}`;

      if (!isGoStimulus) {
        setTimeout(() => {
          if (waitingForResponse) {
            trialCount++;
            responseData.push({ stimulus: currentStimulus, key: null, correct: true, rt: null });
            messageEl.textContent = 'Correctly withheld!';
            messageEl.style.color = 'green';
            waitingForResponse = false;
            if (trialCount >= totalTrials) {
              endGame();
            } else {
              setTimeout(showStimulus, 1000);
            }
          }
        }, 1500);
      }
    }

    function checkCooldown() {
      if (consecutiveIncorrect >= 3) {
        cooldown = true;
        waitingForResponse = false;
        stimulusEl.textContent = '';
        messageEl.textContent = 'Cooldown - Take a moment to refocus';
        let count = 5;
        const cooldownInterval = setInterval(() => {
          messageEl.textContent = `Cooldown - Resuming in ${count}...`;
          count--;
          if (count < 0) {
            clearInterval(cooldownInterval);
            messageEl.textContent = '';
            consecutiveIncorrect = 0;
            cooldown = false;
            showStimulus();
          }
        }, 1000);
      }
    }

    function endGame() {
      gameStarted = false;
      stimulusEl.textContent = '';
      messageEl.textContent = '';

      const correctCount = responseData.filter(d => d.correct).length;
      const avgRTs = responseData.filter(d => d.rt !== null);
      const averageRT = avgRTs.reduce((acc, d) => acc + d.rt, 0) / avgRTs.length;

      resultsEl.innerHTML = `
        <h3>Results Summary</h3>
        <p>Total Correct: ${correctCount} / ${totalTrials}</p>
        <p>Average Reaction Time: ${Math.round(averageRT)} ms</p>
        <h4>Detailed Responses:</h4>
        <table>
          <tr><th>Trial</th><th>Stimulus</th><th>Key</th><th>Correct</th><th>Reaction Time</th></tr>
          ${responseData.map((d, i) => 
            `<tr>
              <td>${i + 1}</td>
              <td>${d.stimulus}</td>
              <td>${d.key || '—'}</td>
              <td>${d.correct ? '✅' : '❌'}</td>
              <td>${d.rt !== null ? Math.round(d.rt) + ' ms' : '—'}</td>
            </tr>`
          ).join('')}
        </table>
      `;
      playAgainBtn.style.display = 'inline-block';
    }

    window.addEventListener('keydown', (e) => {
      if (!gameStarted || cooldown || !waitingForResponse) return;

      const keyPressed = e.key;
      const reactionTime = performance.now() - responseStart;
      waitingForResponse = false;
      trialCount++;

      let correct = false;
      let rt = null;

      if (isGoStimulus && directionKeys[keyPressed] === currentStimulus) {
        correct = true;
        rt = reactionTime;
        messageEl.textContent = `Correct! RT: ${Math.round(reactionTime)}ms`;
        messageEl.style.color = 'green';
        consecutiveIncorrect = 0;
      } else {
        correct = false;
        rt = isGoStimulus ? reactionTime : null;
        messageEl.textContent = isGoStimulus ? 'Incorrect key!' : 'Incorrect - You should have withheld.';
        messageEl.style.color = 'red';
        consecutiveIncorrect++;
        checkCooldown();
      }

      responseData.push({ stimulus: currentStimulus, key: keyPressed, correct, rt });

      if (trialCount >= totalTrials) {
        endGame();
      } else if (!cooldown) {
        setTimeout(showStimulus, 1000);
      }
    });

    startBtn.addEventListener('click', () => {
      startBtn.style.display = 'none';
      instructionsEl.style.display = 'none';
      resultsEl.innerHTML = '';
      playAgainBtn.style.display = 'none';
      gameStarted = true;
      responseData = [];
      trialCount = 0;
      consecutiveIncorrect = 0;
      cooldown = false;
      showStimulus();
    });

    playAgainBtn.addEventListener('click', () => {
      playAgainBtn.style.display = 'none';
      resultsEl.innerHTML = '';
      counterEl.textContent = ''; // Hide the "Arrow 20 out of 20" text
      instructionsEl.style.display = 'block';
      startBtn.style.display = 'inline-block';
    });
  </script>
</body>
</html>
