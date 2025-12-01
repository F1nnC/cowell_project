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

<style>
  canvas {
    background: radial-gradient(circle, #111, #000);
    border: 2px solid white;
    display: block;
    margin: 30px auto;
  }
</style>

<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");

  // === GAME STATE ===
  let aiScore, envScore, gameOver, clickCount;
  let gameStarted = false;

  // === EVENT SYSTEM (EXTREME TRADEOFFS) ===
  const events = [
    {
      question: "A massive AI data megacenter will triple output but destroy a forest. Approve it?",
      yes: { ai: +25, env: -30 },  // PRO AI = HUGE ENV LOSS
      no:  { ai: -18, env: +25 }   // PRO ENV = HUGE AI LOSS
    },
    {
      question: "Global climate laws would heavily restrict your AI research. Accept?",
      yes: { ai: -22, env: +30 },
      no:  { ai: +20, env: -28 }
    },
    {
      question: "A foreign power offers unlimited servers with zero environmental regulations.",
      yes: { ai: +30, env: -35 },
      no:  { ai: -15, env: +20 }
    },
    {
      question: "Switch all infrastructure to renewable energy at great cost to progress?",
      yes: { ai: -20, env: +35 },
      no:  { ai: +18, env: -22 }
    }
  ];

  // === BUTTONS ===
  const mainButton = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    radius: 70
  };

  const restartButton = {
    x: canvas.width / 2,
    y: canvas.height / 2 + 80,
    radius: 40
  };

  // === RESET GAME ===
  function resetGame() {
    aiScore = 30;
    envScore = 70;
    gameOver = false;
    clickCount = 0;
    gameStarted = false;
  }

  resetGame();

  // === PASSIVE SYSTEM ===
  setInterval(() => {
    if (!gameStarted || gameOver) return;

    aiScore -= 2;
    envScore += 0.5;

    clampStats();
    checkGameOver();
    draw();
  }, 1000);

  // === EVENT POPUP WITH LIVE STATS ===
  function triggerRandomEvent() {
    const event = events[Math.floor(Math.random() * events.length)];

    const message =
      event.question +
      "\n\n" +
      "CURRENT STATUS:" +
      "\nAI: " + aiScore.toFixed(0) +
      "\nENV: " + envScore.toFixed(1) + "%" +
      "\n\nOK = YES\nCancel = NO";

    const choice = confirm(message);

    if (choice) {
      aiScore += event.yes.ai;
      envScore += event.yes.env;
    } else {
      aiScore += event.no.ai;
      envScore += event.no.env;
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
    }
  }

  // === DRAWING ===
  function drawBars() {
    ctx.fillStyle = "white";
    ctx.font = "14px Arial";

    ctx.fillStyle = "cyan";
    ctx.fillRect(20, 20, aiScore * 1.5, 10);
    ctx.strokeStyle = "white";
    ctx.strokeRect(20, 20, 150, 10);
    ctx.fillText("AI", 20, 45);

    ctx.fillStyle = "lime";
    ctx.fillRect(canvas.width - 170, 20, envScore * 1.5, 10);
    ctx.strokeRect(canvas.width - 170, 20, 150, 10);
    ctx.fillText("ENV", canvas.width - 170, 45);
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
    ctx.font = "14px Arial";
    ctx.textAlign = "center";
    ctx.fillText("RESTART", restartButton.x, restartButton.y + 5);
  }

  function drawStats() {
    ctx.fillStyle = "white";
    ctx.textAlign = "left";
    ctx.font = "16px Arial";
    ctx.fillText("AI Score: " + aiScore.toFixed(0), 20, canvas.height - 20);
    ctx.fillText("Environment: " + envScore.toFixed(1) + "%", canvas.width - 240, canvas.height - 20);
  }

  function drawStartScreen() {
    ctx.fillStyle = "white";
    ctx.font = "28px Arial";
    ctx.textAlign = "center";
    ctx.fillText("ARTIFICIAL INSANITY", canvas.width / 2, 110);

    ctx.font = "16px Arial";
    ctx.fillText("Click the AI core to increase intelligence.", canvas.width / 2, 170);
    ctx.fillText("Wait and the environment heals slowly.", canvas.width / 2, 195);
    ctx.fillText("But your AI decays without attention.", canvas.width / 2, 220);
    ctx.fillText("Every 4 clicks, a major decision appears.", canvas.width / 2, 255);
    ctx.fillText("AI = 0 → foreign takeover.", canvas.width / 2, 300);
    ctx.fillText("ENV = 0 → ecological collapse.", canvas.width / 2, 325);
    ctx.fillText("Click anywhere to begin.", canvas.width / 2, 360);
  }

  function drawEnd() {
    ctx.fillStyle = "red";
    ctx.font = "26px Arial";
    ctx.textAlign = "center";

    if (aiScore <= 0) {
      ctx.fillText("YOUR AI WAS TAKEN OVER", canvas.width / 2, 150);
      ctx.fillText("BY A FOREIGN COUNTRY", canvas.width / 2, 185);
    } else {
      ctx.fillText("NATURE HAS COLLAPSED", canvas.width / 2, 170);
    }

    drawRestartButton();
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (!gameStarted) {
      drawStartScreen();
      return;
    }

    drawBars();

    if (!gameOver) {
      drawMainButton();
    } else {
      drawEnd();
    }

    drawStats();
  }

  // === INPUT ===
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
      draw();
      return;
    }

    if (gameOver && isInside(mouseX, mouseY, restartButton)) {
      resetGame();
      draw();
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
      draw();
    }
  });

  draw();
</script>
