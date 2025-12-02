---
layout: post 
title: Artificial Insanity 
hide: true
show_reading_time: false
---

# Artificial Insanity  
*Progress always asks a question.*

## Need to add
- good ending
- background design that is effected
    - the player has to go through each decesion, will update background for each depdening on what happens on the decesions
    - then if player gets through all decesions, he can negoicate ai terms, basically win game
- change rules at begining
- make the bars decrease smoother and increase smoother

---

<canvas id="gameCanvas" width="700" height="420"></canvas>

<script>
  // === BACKGROUND IMAGES ===
  const bgStart = null; // solid black
  const bg1 = new Image();
  bg1.src = "./images/1forest.png";

  const bg2 = new Image();
  bg2.src = "./images/2forest.png";

  const bg3 = new Image();
  bg3.src = "./images/3forest.png";

  let currentBG = bgStart;

  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");

  // === GAME STATE ===
  let aiScore, envScore, gameOver, clickCount;
  let gameStarted = false;
  let proCorrectCount = 0;
  let wrongOnce = false;
  let goodEnding = false;

  let displayAI = 0;
  let displayENV = 0;

  // === EVENT SYSTEM ===
  const events = [
    {
      question: "A massive AI data megacenter will triple output but destroy a forest. Approve it?",
      yes: { ai: +25, env: -30, proCorrect: false },
      no:  { ai: -18, env: +25, proCorrect: true }
    },
    {
      question: "Will you lobby the government for less regulation on AI?",
      yes: { ai: +22, env: -30, proCorrect: false },
      no:  { ai: -20, env: +28, proCorrect: true }
    },
    {
      question: "A new kind of battery could be used, but it may damage rivers.",
      yes: { ai: +30, env: -35, proCorrect: false },
      no:  { ai: -15, env: +20, proCorrect: true }
    },
    {
      question: "Switch all infrastructure to renewable energy at great cost?",
      yes: { ai: -20, env: +35, proCorrect: true },
      no:  { ai: +18, env: -22, proCorrect: false }
    },
    {
      // ❌ IMPOSSIBLE QUESTION (ONLY TRIGGERS IF PLAYER FAILED EARLIER)
      question: "AI has already surpassed human control. Can you stop it?",
      yes: { ai: -100, env: -100 },
      no:  { ai: -100, env: -100 }
    }
  ];

  const mainButton = { x: canvas.width / 2, y: canvas.height / 2, radius: 70 };
  const restartButton = { x: canvas.width / 2, y: canvas.height / 2 + 80, radius: 40 };

  function resetGame() {
    aiScore = 30;
    envScore = 70;
    displayAI = aiScore;
    displayENV = envScore;
    gameOver = false;
    clickCount = 0;
    gameStarted = false;
    proCorrectCount = 0;
    wrongOnce = false;
    goodEnding = false;
    currentBG = bgStart;
  }

  resetGame();

  // === PASSIVE SYSTEM (FASTER AI DECAY) ===
  setInterval(() => {
    if (!gameStarted || gameOver) return;
    aiScore -= 4;
    envScore += 0.6;
    clampStats();
    checkGameOver();
  }, 1000);

  // === EVENTS ===
  function triggerRandomEvent() {
    let event;

    if (proCorrectCount < 4) {
      event = events[proCorrectCount];
    } else {
      if (proCorrectCount === 4) {
        goodEnding = true;
        gameOver = true;
        return;
      } else {
        event = events[4]; // impossible question
      }
    }

    const choice = confirm(event.question + "\n\nOK = YES\nCancel = NO");
    const result = choice ? event.yes : event.no;

    aiScore += result.ai;
    envScore += result.env;

    if (result.proCorrect === false) {
      wrongOnce = true;
      currentBG = bg2; // switch to polluted forest
    }

    if (result.proCorrect === true) {
      proCorrectCount++;
    }

    // Force impossible ending if player already failed once
    if (wrongOnce && proCorrectCount >= 4) {
      aiScore = 0;
      envScore = 0;
    }

    clampStats();
  }

  function clampStats() {
    aiScore = Math.max(0, Math.min(100, aiScore));
    envScore = Math.max(0, Math.min(100, envScore));
  }

  function checkGameOver() {
    if (aiScore <= 0 || envScore <= 0) {
      gameOver = true;
      currentBG = bg3; // dead forest
    }
  }

  // === SMOOTH BAR ANIMATION ===
  function smoothBars() {
    displayAI += (aiScore - displayAI) * 0.08;
    displayENV += (envScore - displayENV) * 0.08;
  }

  // === BARS + LIVE % DISPLAY ===
  function drawBars() {
    ctx.fillStyle = "cyan";
    ctx.fillRect(20, 20, displayAI * 1.5, 10);
    ctx.strokeStyle = "white";
    ctx.strokeRect(20, 20, 150, 10);
    ctx.fillStyle = "white";
    ctx.fillText("AI: " + Math.round(displayAI) + "%", 20, 50);

    ctx.fillStyle = "lime";
    ctx.fillRect(canvas.width - 170, 20, displayENV * 1.5, 10);
    ctx.strokeRect(canvas.width - 170, 20, 150, 10);
    ctx.fillText("ENV: " + Math.round(displayENV) + "%", canvas.width - 170, 50);
  }

  function drawMainButton() {
    ctx.beginPath();
    ctx.arc(mainButton.x, mainButton.y, mainButton.radius, 0, Math.PI * 2);
    ctx.fillStyle = "cyan";
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.stroke();
    ctx.fillStyle = "black";
    ctx.font = "20px Arial";
    ctx.textAlign = "center";
    ctx.fillText("UPGRADE", mainButton.x, mainButton.y + 7);
  }

  function drawRestartButton() {
    ctx.beginPath();
    ctx.arc(restartButton.x, restartButton.y, restartButton.radius, 0, Math.PI * 2);
    ctx.fillStyle = "red";
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.stroke();
    ctx.fillStyle = "white";
    ctx.fillText("RESTART", restartButton.x, restartButton.y + 5);
  }

  function drawStartScreen() {
    ctx.fillStyle = "black";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = "white";
    ctx.font = "26px Arial";
    ctx.textAlign = "center";
    ctx.fillText("ARTIFICIAL INSANITY", canvas.width / 2, 160);
    ctx.fillText("Click to Begin", canvas.width / 2, 220);
  }

  function drawEnd() {
    ctx.textAlign = "center";

    if (goodEnding) {
      ctx.fillStyle = "lime";
      ctx.fillText("GLOBAL AI REGULATIONS PASSED", canvas.width / 2, 180);
      ctx.fillText("THE PLANET RECOVERS", canvas.width / 2, 220);
    } else {
      ctx.fillStyle = "red";
      ctx.fillText("TOTAL COLLAPSE", canvas.width / 2, 200);
    }

    drawRestartButton();
  }

  function drawBackground() {
    if (currentBG) {
      ctx.drawImage(currentBG, 0, 0, canvas.width, canvas.height);
    } else {
      ctx.fillStyle = "black";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    smoothBars();

    if (!gameStarted) {
      drawStartScreen();
      return;
    }

    drawBackground();
    drawBars();

    if (!gameOver) drawMainButton();
    else drawEnd();
  }

  function isInside(x, y, obj) {
    const dx = x - obj.x;
    const dy = y - obj.y;
    return Math.sqrt(dx * dx + dy * dy) < obj.radius;
  }

  canvas.addEventListener("click", (e) => {
    const rect = canvas.getBoundingClientRect();
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;

    if (!gameStarted) {
      gameStarted = true;
      currentBG = bg1;
      return;
    }

    if (gameOver && isInside(mouseX, mouseY, restartButton)) {
      resetGame();
      return;
    }

    if (!gameOver && isInside(mouseX, mouseY, mainButton)) {
      aiScore += 10;
      envScore -= 5;
      clickCount++;

      if (clickCount % 4 === 0) {
        triggerRandomEvent();
      }

      clampStats();
      checkGameOver();
    }
  });

  function gameLoop() {
    draw();
    requestAnimationFrame(gameLoop);
  }

  gameLoop();
</script>
