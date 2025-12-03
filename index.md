---
layout: post 
title: Artificial Insanity 
hide: true
show_reading_time: false
---

# Artificial Insanity  
1. Don't let your AI or Environment bar hit zero 
2. Answer the questions correctly
3. Click to begin



<canvas id="gameCanvas" width="700" height="420"></canvas>

<script>
  // === BACKGROUND IMAGES ===
  const bgStart = null;
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
  let questionIndex = 0;
  let failedOnce = false;
  let displayAI = 0;
  let displayENV = 0;
  let goodEnding = false;

  ctx.font = "20px Trebuchet MS";

  // === QUESTIONS ===
  const events = [
    {
      question: "A massive AI data megacenter will triple output but destroy a forest. Approve it?",
      yes: { ai: +25, env: -30, correct: false },
      no:  { ai: -18, env: +25, correct: true }
    },
    {
      question: "Will you lobby the government for less regulation on AI?",
      yes: { ai: +22, env: -30, correct: false },
      no:  { ai: -20, env: +28, correct: true }
    },
    {
      question: "A new kind of battery could be used, but it may damage rivers.",
      yes: { ai: +30, env: -35, correct: false },
      no:  { ai: -15, env: +20, correct: true }
    },
    {
      question: "Switch all infrastructure to renewable energy at great cost?",
      yes: { ai: -20, env: +35, correct: true },
      no:  { ai: +18, env: -22, correct: false }
    },
    {
      question: "AI has already surpassed human control. There is no stopping it now.",
      yes: { ai: -100, env: -100 },
      no:  { ai: -100, env: -100 }
    }
  ];

  const mainButton = { x: canvas.width / 2, y: canvas.height / 2, radius: 70 };
  const restartButton = { x: canvas.width / 2, y: canvas.height / 2 + 90, radius: 45 };

  function resetGame() {
    aiScore = 30;
    envScore = 70;
    displayAI = aiScore;
    displayENV = envScore;
    gameOver = false;
    clickCount = 0;
    gameStarted = false;
    questionIndex = 0;
    failedOnce = false;
    goodEnding = false;
    currentBG = bgStart;
  }

  resetGame();

  // === PASSIVE AI DECAY ===
  setInterval(() => {
    if (!gameStarted || gameOver) return;
    aiScore -= 4;
    envScore += 0.6;
    clampStats();
    checkGameOver();
  }, 1000);

  // === QUESTION SYSTEM (ALWAYS PROGRESSES) ===
  function triggerEvent() {
    let event = events[questionIndex];
    const choice = confirm(event.question + "\n\nOK = YES\nCancel = NO");
    const result = choice ? event.yes : event.no;

    aiScore += result.ai;
    envScore += result.env;

    if (questionIndex < 4 && result.correct === false) {
      failedOnce = true;
      currentBG = bg2;
    }

    questionIndex++;

    // ✅ If player cleared first 4 perfectly → WIN
    if (questionIndex === 4 && !failedOnce) {
      goodEnding = true;
      gameOver = true;
      return;
    }

    // ❌ If final question reached and failure occurred → AUTO LOSS
    if (questionIndex === 5 && failedOnce) {
      aiScore = 0;
      envScore = 0;
      clampStats();
      checkGameOver();
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
      currentBG = bg3;
    }
  }

  // === SMOOTH BAR ANIMATION ===
  function smoothBars() {
    displayAI += (aiScore - displayAI) * 0.08;
    displayENV += (envScore - displayENV) * 0.08;
  }

  // === BARS (CLEAN + CENTERED TEXT) ===
  function drawBars() {
    ctx.font = "18px Trebuchet MS";

    ctx.fillStyle = "cyan";
    ctx.fillRect(20, 20, displayAI * 1.5, 12);
    ctx.strokeRect(20, 20, 150, 12);
    ctx.textAlign = "center";
    ctx.fillStyle = "white";
    ctx.fillText("AI: " + Math.round(displayAI) + "%", 95, 55);

    ctx.fillStyle = "lime";
    ctx.fillRect(canvas.width - 170, 20, displayENV * 1.5, 12);
    ctx.strokeRect(canvas.width - 170, 20, 150, 12);
    ctx.fillText("ENV: " + Math.round(displayENV) + "%", canvas.width - 95, 55);
  }

  function drawMainButton() {
    ctx.beginPath();
    ctx.arc(mainButton.x, mainButton.y, mainButton.radius, 0, Math.PI * 2);
    ctx.fillStyle = "cyan";
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.stroke();
    ctx.fillStyle = "black";
    ctx.font = "22px Trebuchet MS";
    ctx.textAlign = "center";
    ctx.fillText("UPGRADE", mainButton.x, mainButton.y + 8);
  }

  function drawRestartButton() {
    ctx.beginPath();
    ctx.arc(restartButton.x, restartButton.y, restartButton.radius, 0, Math.PI * 2);
    ctx.fillStyle = "red";
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.stroke();
    ctx.fillStyle = "white";
    ctx.font = "20px Trebuchet MS";
    ctx.fillText("RESTART", restartButton.x, restartButton.y + 7);
  }

  function drawStartScreen() {
    ctx.fillStyle = "black";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "white";
    ctx.font = "36px Trebuchet MS";
    ctx.textAlign = "center";
    ctx.fillText("ARTIFICIAL INSANITY", canvas.width / 2, 180);

    ctx.font = "22px Trebuchet MS";
    ctx.fillText("Click to Begin", canvas.width / 2, 235);
  }

  function drawEnd() {
    ctx.textAlign = "center";

    ctx.font = "30px Trebuchet MS";
    if (goodEnding) {
      ctx.fillStyle = "lime";
      ctx.fillText("HUMANITY RETAINS CONTROL", canvas.width / 2, 200);
    } else {
      ctx.fillStyle = "red";
      ctx.fillText("AI HAS TAKEN OVER THE WORLD", canvas.width / 2, 200);
    }

    drawRestartButton();
  }

  function drawBackground() {
    if (currentBG) ctx.drawImage(currentBG, 0, 0, canvas.width, canvas.height);
    else {
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
        triggerEvent();
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
