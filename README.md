<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pong Game</title>
  <style>
    :root {
      --bg: #0b1020;
      --panel: #121a2b;
      --line: rgba(255,255,255,0.25);
      --text: #eaf2ff;
      --paddle: #7dd3fc;
      --ball: #f8fafc;
      --score: #fbbf24;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      display: grid;
      place-items: center;
      min-height: 100vh;
      background: radial-gradient(circle at top, #1a2540, var(--bg));
      color: var(--text);
      font-family: Arial, sans-serif;
    }

    .game-wrap {
      width: 820px;
      max-width: 90vw;
      background: var(--panel);
      border: 2px solid rgba(255,255,255,0.12);
      border-radius: 14px;
      box-shadow: 0 20px 60px rgba(0,0,0,0.35);
      padding: 16px;
    }

    .scoreboard {
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 2rem;
      font-weight: 700;
      letter-spacing: 2px;
      padding: 0 12px 12px;
      color: var(--score);
    }

    .scoreboard .name {
      font-size: 0.9rem;
      letter-spacing: 1px;
      opacity: 0.8;
      color: var(--text);
    }

    canvas {
      display: block;
      width: 100%;
      height: auto;
      background: rgba(0,0,0,0.15);
      border: 2px solid var(--line);
      border-radius: 10px;
    }
  </style>
</head>
<body>
  <div class="game-wrap">
    <div class="scoreboard">
      <div>
        <div class="name">YOU</div>
        <span id="score-left">0</span>
      </div>
      <div>
        <div class="name">CPU</div>
        <span id="score-right">0</span>
      </div>
    </div>

    <canvas id="gameCanvas" width="800" height="500" aria-label="Pong game"></canvas>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreLeftEl = document.getElementById("score-left");
    const scoreRightEl = document.getElementById("score-right");

    const paddleWidth = 14;
    const paddleHeight = 100;
    const paddleSpeed = 7;

    const leftPaddle = {
      x: 20,
      y: canvas.height / 2 - paddleHeight / 2,
      width: paddleWidth,
      height: paddleHeight,
      color: "#7dd3fc"
    };

    const rightPaddle = {
      x: canvas.width - 20 - paddleWidth,
      y: canvas.height / 2 - paddleHeight / 2,
      width: paddleWidth,
      height: paddleHeight,
      color: "#f9a8d4"
    };

    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      radius: 8,
      vx: 5,
      vy: 4,
      speed: 5
    };

    const keys = {
      up: false,
      down: false
    };

    let mouseY = null;
    let scoreLeft = 0;
    let scoreRight = 0;

    function clamp(value, min, max) {
      return Math.min(Math.max(value, min), max);
    }

    function resetBall(direction) {
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      const angle = (Math.random() * 1.2 - 0.6); // roughly -0.6 to 0.6 radians
      const speed = 5 + Math.random() * 1.5;

      ball.vx = direction * speed * Math.cos(angle);
      ball.vy = speed * Math.sin(angle);
    }

    function updateLeftPaddle() {
      if (keys.up) {
        leftPaddle.y -= paddleSpeed;
      }
      if (keys.down) {
        leftPaddle.y += paddleSpeed;
      }

      if (mouseY !== null) {
        leftPaddle.y = mouseY - leftPaddle.height / 2;
      }

      leftPaddle.y = clamp(leftPaddle.y, 0, canvas.height - leftPaddle.height);
    }

    function updateRightPaddle() {
      const targetY = ball.y - rightPaddle.height / 2;
      const move = (targetY - rightPaddle.y) * 0.12;
      rightPaddle.y += move;
      rightPaddle.y = clamp(rightPaddle.y, 0, canvas.height - rightPaddle.height);
    }

    function updateBall() {
      ball.x += ball.vx;
      ball.y += ball.vy;

      // Wall collision
      if (ball.y - ball.radius <= 0) {
        ball.y = ball.radius;
        ball.vy *= -1;
      }

      if (ball.y + ball.radius >= canvas.height) {
        ball.y = canvas.height - ball.radius;
        ball.vy *= -1;
      }

      // Left paddle collision
      if (
        ball.x - ball.radius <= leftPaddle.x + leftPaddle.width &&
        ball.y >= leftPaddle.y &&
        ball.y <= leftPaddle.y + leftPaddle.height &&
        ball.x > leftPaddle.x
      ) {
        ball.x = leftPaddle.x + leftPaddle.width + ball.radius;
        const relativeIntersect = (ball.y - (leftPaddle.y + leftPaddle.height / 2)) / (leftPaddle.height / 2);
        const bounceAngle = relativeIntersect * (Math.PI / 3);
        const speed = Math.hypot(ball.vx, ball.vy) + 0.3;
        ball.vx = speed * Math.cos(bounceAngle);
        ball.vy = speed * Math.sin(bounceAngle);
      }

      // Right paddle collision
      if (
        ball.x + ball.radius >= rightPaddle.x &&
        ball.y >= rightPaddle.y &&
        ball.y <= rightPaddle.y + rightPaddle.height &&
        ball.x < rightPaddle.x + rightPaddle.width
      ) {
        ball.x = rightPaddle.x - ball.radius;
        const relativeIntersect = (ball.y - (rightPaddle.y + rightPaddle.height / 2)) / (rightPaddle.height / 2);
        const bounceAngle = relativeIntersect * (Math.PI / 3);
        const speed = Math.hypot(ball.vx, ball.vy) + 0.3;
        ball.vx = -speed * Math.cos(bounceAngle);
        ball.vy = speed * Math.sin(bounceAngle);
      }

      // Score
      if (ball.x + ball.radius < 0) {
        scoreRight += 1;
        scoreRightEl.textContent = scoreRight;
        resetBall(1);
      }

      if (ball.x - ball.radius > canvas.width) {
        scoreLeft += 1;
        scoreLeftEl.textContent = scoreLeft;
        resetBall(-1);
      }
    }

    function drawRect(x, y, width, height, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x, y, width, height);
    }

    function drawBall(x, y, radius, color) {
      ctx.fillStyle = color;
      ctx.beginPath();
      ctx.arc(x, y, radius, 0, Math.PI * 2);
      ctx.fill();
    }

    function drawCenterLine() {
      ctx.strokeStyle = "rgba(255,255,255,0.22)";
      ctx.lineWidth = 2;
      ctx.setLineDash([10, 12]);
      ctx.beginPath();
      ctx.moveTo(canvas.width / 2, 0);
      ctx.lineTo(canvas.width / 2, canvas.height);
      ctx.stroke();
      ctx.setLineDash([]);
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      drawCenterLine();
      drawRect(leftPaddle.x, leftPaddle.y, leftPaddle.width, leftPaddle.height, leftPaddle.color);
      drawRect(rightPaddle.x, rightPaddle.y, rightPaddle.width, rightPaddle.height, rightPaddle.color);
      drawBall(ball.x, ball.y, ball.radius, "#f8fafc");
    }

    function gameLoop() {
      updateLeftPaddle();
      updateRightPaddle();
      updateBall();
      draw();
      requestAnimationFrame(gameLoop);
    }

    document.addEventListener("keydown", (event) => {
      if (event.key === "ArrowUp") {
        keys.up = true;
      }
      if (event.key === "ArrowDown") {
        keys.down = true;
      }
    });

    document.addEventListener("keyup", (event) => {
      if (event.key === "ArrowUp") {
        keys.up = false;
      }
      if (event.key === "ArrowDown") {
        keys.down = false;
      }
    });

    canvas.addEventListener("mousemove", (event) => {
      const rect = canvas.getBoundingClientRect();
      const mouseYRelative = event.clientY - rect.top;
      mouseY = mouseYRelative;
    });

    resetBall(1);
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
