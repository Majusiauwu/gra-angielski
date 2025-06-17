<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
  <title>Virus Translator Platformer</title>
  <style>
    * {
      box-sizing: border-box;
    }
    body, html {
      margin: 0;
      padding: 0;
      height: 100%;
      font-family: 'Arial', sans-serif;
      background-color: #ccefff;
      overflow: hidden;
    }
    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
      background: #ccefff;
    }
    #ui {
      position: absolute;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0,0,0,0.6);
      padding: 10px;
      border-radius: 10px;
      color: white;
      display: none;
      flex-direction: column;
      align-items: center;
      z-index: 10;
    }
    #ui input {
      padding: 5px;
      margin-top: 5px;
    }
    #ui button {
      margin-top: 5px;
      padding: 5px;
      background: #4caf50;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #mobile-controls {
      position: absolute;
      bottom: 10px;
      width: 100%;
      display: flex;
      justify-content: center;
      gap: 10px;
      z-index: 10;
    }
    .control-btn {
      padding: 20px;
      font-size: 24px;
      background: rgba(0, 0, 0, 0.4);
      color: white;
      border: none;
      border-radius: 10px;
      user-select: none;
      touch-action: manipulation;
    }
    #scoreDisplay {
      position: absolute;
      top: 10px;
      right: 10px;
      background: rgba(0,0,0,0.5);
      color: white;
      padding: 8px 12px;
      border-radius: 10px;
      font-size: 18px;
      z-index: 10;
    }
  </style>
</head>
<body>
<canvas id="gameCanvas"></canvas>

<div id="scoreDisplay">Score: 0</div>

<div id="ui">
  <div id="question"></div>
  <input id="answerInput" type="text" placeholder="Your answer..." />
  <button onclick="submitAnswer()">Submit</button>
  <div id="hint"></div>
</div>

<div id="mobile-controls">
  <button class="control-btn" id="leftBtn">⬅️</button>
  <button class="control-btn" id="upBtn">⬆️</button>
  <button class="control-btn" id="rightBtn">➡️</button>
</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let score = 0;
const scoreDisplay = document.getElementById('scoreDisplay');

const player = {
  x: 50,
  y: canvas.height - 100,
  width: 40,
  height: 40,
  color: 'green',
  dx: 0,
  dy: 0,
  grounded: false
};

const gravity = 0.8;
const keys = {};

const platforms = [
 {x: 0, y: canvas.height - 50, width: canvas.width, height: 50},
  {x: canvas.width * 0.15, y: canvas.height - 120, width: 100, height: 10},
  {x: canvas.width * 0.25, y: canvas.height - 270, width: 100, height: 10},
  {x: canvas.width * 0.27, y: canvas.height - 170, width: 100, height: 10},
  {x: canvas.width * 0.39, y: canvas.height - 220, width: 100, height: 10},
  {x: canvas.width * 0.51, y: canvas.height - 220, width: 100, height: 10},
  {x: canvas.width * 0.63, y: canvas.height - 270, width: 80, height: 10},
  {x: canvas.width * 0.73, y: canvas.height - 320, width: 80, height: 10}, 
  {x: canvas.width * 0.83, y: canvas.height - 330, width: 80, height: 10}, 
  {x: canvas.width * 0.93, y: canvas.height - 330, width: 80, height: 10}   
];



const enemies = [
  {x: canvas.width * 0.2 + 20, y: canvas.height - 140, width: 30, height: 30, asked: false, word: ['płuco', 'lung']},
  {x: canvas.width * 0.35 + 10, y: canvas.height - 190, width: 30, height: 30, asked: false, word: ['czaszka', 'skull']},
  {x: canvas.width * 0.5 + 10, y: canvas.height - 240, width: 30, height: 30, asked: false, word: ['mózg', 'brain']},
  {x: canvas.width * 0.65 + 10, y: canvas.height - 290, width: 30, height: 30, asked: false, word: ['kostka', 'ankle']},
  {x: canvas.width * 0.8 + 10, y: canvas.height - 340, width: 30, height: 30, asked: false, word: ['serce', 'heart']},
  {x: canvas.width - 50, y: canvas.height - 370, width: 30, height: 30, asked: false, word: ['krew', 'blood']},
  {x: 100, y: canvas.height - 90, width: 30, height: 30, asked: false, word: ['energiczny', 'vigourous']},
  {x: 200, y: canvas.height - 90, width: 30, height: 30, asked: false, word: ['infekcja bakteryjna', 'bacterial infection']},
  {x: 400, y: canvas.height - 300, width: 30, height: 30, asked: false, word: ['wirusy', 'viruses']},
  {x: 500, y: canvas.height - 100, width: 30, height: 30, asked: false, word: ['kaszel', 'cough']},
  {x: 600, y: canvas.height - 100, width: 30, height: 30, asked: false, word: ['podbite oko', 'black eye']}
];

let currentQuestion = null;

function drawRect(obj, color) {
  ctx.fillStyle = color;
  ctx.fillRect(obj.x, obj.y, obj.width, obj.height);
}

function drawEmoji(obj, emoji) {
  ctx.font = `${obj.height}px Arial`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'top';
  ctx.fillText(emoji, obj.x + obj.width / 2, obj.y);
}

function update() {
  player.dy += gravity;
  player.x += player.dx;
  player.y += player.dy;
  player.grounded = false;

  for (let platform of platforms) {
    if (
      player.x < platform.x + platform.width &&
      player.x + player.width > platform.x &&
      player.y + player.height <= platform.y + 10 &&
      player.y + player.height + player.dy >= platform.y
    ) {
      player.y = platform.y - player.height;
      player.dy = 0;
      player.grounded = true;
    }
  }

  for (let enemy of enemies) {
    if (
      !enemy.asked &&
      player.x < enemy.x + enemy.width &&
      player.x + player.width > enemy.x &&
      player.y < enemy.y + enemy.height &&
      player.y + player.height > enemy.y
    ) {
      enemy.asked = true;
      showQuestion(enemy.word);
    }
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawRect(player, player.color);
  platforms.forEach(p => drawRect(p, '#654321'));
  enemies.forEach(e => drawEmoji(e, '🦠'));
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

function showQuestion(word) {
  currentQuestion = word;
  document.getElementById('question').innerText = `Translate: ${word[0]}`;
  document.getElementById('hint').innerText = '';
  document.getElementById('answerInput').value = '';
  document.getElementById('ui').style.display = 'flex';
}

function submitAnswer() {
  const input = document.getElementById('answerInput').value.trim().toLowerCase();
  const correct = currentQuestion[1];
  if (levenshtein(input, correct) <= 1) {
    score++;
    scoreDisplay.innerText = `Score: ${score}`;
    document.getElementById('ui').style.display = 'none';
  } else {
    const hint = correct.slice(0, input.length + 1);
    document.getElementById('hint').innerText = `Hint: ${hint}`;
  }
}

function levenshtein(a, b) {
  const dp = Array.from({ length: a.length + 1 }, () => Array(b.length + 1).fill(0));
  for (let i = 0; i <= a.length; i++) dp[i][0] = i;
  for (let j = 0; j <= b.length; j++) dp[0][j] = j;
  for (let i = 1; i <= a.length; i++) {
    for (let j = 1; j <= b.length; j++) {
      if (a[i - 1] === b[j - 1]) dp[i][j] = dp[i - 1][j - 1];
      else dp[i][j] = 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
    }
  }
  return dp[a.length][b.length];
}

document.addEventListener('keydown', e => {
  keys[e.key] = true;
});

document.addEventListener('keyup', e => {
  keys[e.key] = false;
});

// Mobile control events
function addMobileControl(buttonId, key) {
  const btn = document.getElementById(buttonId);
  btn.addEventListener('pointerdown', () => keys[key] = true);
  btn.addEventListener('pointerup', () => keys[key] = false);
  btn.addEventListener('pointerleave', () => keys[key] = false);
  btn.addEventListener('touchend', () => keys[key] = false);
}

addMobileControl('leftBtn', 'ArrowLeft');
addMobileControl('upBtn', 'ArrowUp');
addMobileControl('rightBtn', 'ArrowRight');

function handleInput() {
  player.dx = 0;
  if (keys['ArrowLeft']) player.dx = -4;
  if (keys['ArrowRight']) player.dx = 4;
  if (keys['ArrowUp'] && player.grounded) player.dy = -12;
  requestAnimationFrame(handleInput);
}

loop();
handleInput();
</script>
</body>
</html>
