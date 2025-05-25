<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Змейка на весь экран</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      background: #111;
      overflow: hidden;
      font-family: Arial, sans-serif;
      color: white;
    }

    #menu {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
    }

    button {
      padding: 12px 24px;
      margin: 8px;
      font-size: 18px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
    }

    #score, #musicBtn {
      position: absolute;
      top: 10px;
      left: 10px;
      font-size: 18px;
      background: rgba(0, 255, 0, 0.1);
      padding: 10px;
      border-radius: 8px;
    }

    #musicBtn {
      left: auto;
      right: 10px;
      background: rgba(0, 150, 0, 0.5);
      color: white;
    }

    canvas {
      display: block;
    }
  </style>
</head>
<body>

<div id="menu">
  <h2>Выберите уровень сложности</h2>
  <button onclick="startGame(150)">🟢 Лёгкий</button>
  <button onclick="startGame(100)">🟡 Средний</button>
  <button onclick="startGame(60)">🔴 Сложный</button>
</div>

<div id="score">Счёт: 0 | Рекорд: 0</div>
<button id="musicBtn" onclick="toggleMusic()">🔊 Музыка: Включена</button>

<canvas id="game"></canvas>

<audio id="music" loop>
  <source src="https://www.bensound.com/bensound-music/bensound-creativeminds.mp3" type="audio/mpeg">
</audio>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const music = document.getElementById('music');
  const musicBtn = document.getElementById('musicBtn');
  const scoreDisplay = document.getElementById('score');

  let width = window.innerWidth;
  let height = window.innerHeight;
  canvas.width = width;
  canvas.height = height;

  let box = Math.floor(Math.min(width, height) / 30); // адаптивный размер ячейки
  let cols = Math.floor(width / box);
  let rows = Math.floor(height / box);

  let snake, direction, food, score, game;
  let highScore = localStorage.getItem('snakeHighScore') || 0;
  let isMusicPlaying = true;

  document.addEventListener('keydown', e => {
    if (e.key === 'ArrowLeft' && direction !== 'RIGHT') direction = 'LEFT';
    if (e.key === 'ArrowUp' && direction !== 'DOWN') direction = 'UP';
    if (e.key === 'ArrowRight' && direction !== 'LEFT') direction = 'RIGHT';
    if (e.key === 'ArrowDown' && direction !== 'UP') direction = 'DOWN';
  });

  function startGame(speed) {
    document.getElementById('menu').style.display = 'none';
    music.play();

    snake = [{ x: Math.floor(cols / 2), y: Math.floor(rows / 2) }];
    direction = null;
    score = 0;
    updateScore();

    food = {
      x: Math.floor(Math.random() * cols),
      y: Math.floor(Math.random() * rows)
    };

    if (game) clearInterval(game);
    game = setInterval(drawGame, speed);
  }

  function updateScore() {
    if (score > highScore) {
      highScore = score;
      localStorage.setItem('snakeHighScore', highScore);
    }
    scoreDisplay.textContent = `Счёт: ${score} | Рекорд: ${highScore}`;
  }

  function drawGame() {
    ctx.fillStyle = '#111';
    ctx.fillRect(0, 0, width, height);

    // Змейка
    for (let i = 0; i < snake.length; i++) {
      ctx.fillStyle = i === 0 ? 'lime' : 'green';
      ctx.fillRect(snake[i].x * box, snake[i].y * box, box, box);
    }

    // Еда
    ctx.fillStyle = 'red';
    ctx.fillRect(food.x * box, food.y * box, box, box);

    let headX = snake[0].x;
    let headY = snake[0].y;

    if (direction === 'LEFT') headX--;
    if (direction === 'UP') headY--;
    if (direction === 'RIGHT') headX++;
    if (direction === 'DOWN') headY++;

    if (
      headX < 0 || headX >= cols ||
      headY < 0 || headY >= rows ||
      collision({ x: headX, y: headY }, snake)
    ) {
      clearInterval(game);
      music.pause();
      alert('Игра окончена! Ваш счёт: ' + score);
      location.reload();
      return;
    }

    if (headX === food.x && headY === food.y) {
      score++;
      updateScore();
      food = {
        x: Math.floor(Math.random() * cols),
        y: Math.floor(Math.random() * rows)
      };
    } else {
      snake.pop();
    }

    let newHead = { x: headX, y: headY };
    snake.unshift(newHead);
  }

  function collision(head, array) {
    for (let i = 0; i < array.length; i++) {
      if (head.x === array[i].x && head.y === array[i].y) return true;
    }
    return false;
  }

  function toggleMusic() {
    isMusicPlaying = !isMusicPlaying;
    if (isMusicPlaying) {
      music.play();
      musicBtn.textContent = '🔊 Музыка: Включена';
    } else {
      music.pause();
      musicBtn.textContent = '🔇 Музыка: Выключена';
    }
  }

  updateScore();
</script>

</body>
</html>
