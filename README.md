<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>Game Boy Tetris</title>
  <style>
    body {
      margin: 0;
      font-family: 'Courier New', Courier, monospace;
      background-color: #d3d3d3;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      flex-direction: column;
    }
    #game-container {
      background-color: #f5f5f5;
      border: 8px solid #333;
      border-radius: 12px;
      padding: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
      box-shadow: 0 0 30px #aaa;
      position: relative;
    }
    canvas {
      background-color: #fff;
      border: 2px solid #000;
    }
    #score, #high-score {
      font-size: 1.5rem;
      margin: 5px 0;
      color: #333;
    }
    button {
      padding: 8px 16px;
      margin: 10px 5px 0 5px;
      font-size: 1rem;
      cursor: pointer;
      background-color: #444;
      color: white;
      border: none;
      border-radius: 5px;
    }
    #controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 10px;
    }
    .control-row {
      display: flex;
      justify-content: center;
      margin: 2px 0;
    }
    .control-btn {
      width: 50px;
      height: 50px;
      margin: 0 5px;
      font-size: 24px;
      background: none;
      border: none;
    }
    .control-btn img {
      width: 100%;
      height: 100%;
      object-fit: contain;
    }
    #game-over {
      position: absolute;
      top: 40%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 20px;
      border-radius: 10px;
      text-align: center;
      display: none;
      z-index: 10;
    }
  </style>
</head>
<body>
  <div id="game-container">
    <div id="score">Score: 0</div>
    <div id="high-score">High Score: 0</div>
    <canvas id="tetris" width="240" height="400"></canvas>
    <div id="game-over">
      <h2>GAME OVER</h2>
      <button onclick="restartGame()">Restart</button>
    </div>
    <button id="pauseBtn">Pause</button>
    <div id="controls">
      <div class="control-row">
        <button class="control-btn" onclick="playerRotate()"><img src="https://cdn-icons-png.flaticon.com/512/60/60525.png" alt="rotate"></button>
      </div>
      <div class="control-row">
        <button class="control-btn" id="leftBtn"><img src="https://cdn-icons-png.flaticon.com/512/271/271220.png" alt="left"></button>
        <button class="control-btn" id="downBtn"><img src="https://cdn-icons-png.flaticon.com/512/271/271210.png" alt="down"></button>
        <button class="control-btn" id="rightBtn"><img src="https://cdn-icons-png.flaticon.com/512/271/271228.png" alt="right"></button>
      </div>
    </div>
  </div>
  <script>
    document.addEventListener('dblclick', e => e.preventDefault(), { passive: false });

    const canvas = document.getElementById('tetris');
    const context = canvas.getContext('2d');
    context.scale(20, 20);

    let pause = false;
    let score = 0;
    let highScore = localStorage.getItem('tetris-highscore') || 0;
    let animationFrameId = null;

    const scoreElement = document.getElementById('score');
    const highScoreElement = document.getElementById('high-score');
    const pauseBtn = document.getElementById('pauseBtn');
    highScoreElement.innerText = 'High Score: ' + highScore;

    function updateScore() {
      scoreElement.innerText = 'Score: ' + score;
      if (score > highScore) {
        highScore = score;
        localStorage.setItem('tetris-highscore', highScore);
        highScoreElement.innerText = 'High Score: ' + highScore;
      }
    }

    pauseBtn.addEventListener('click', () => {
      pause = !pause;
      pauseBtn.textContent = pause ? 'Continue' : 'Pause';
      if (!pause) update();
      else cancelAnimationFrame(animationFrameId);
    });

    const downBtn = document.getElementById('downBtn');
    let downInterval;
    downBtn.addEventListener('touchstart', () => {
      downInterval = setInterval(playerDrop, 50);
    });
    downBtn.addEventListener('touchend', () => {
      clearInterval(downInterval);
    });

    function moveToEdge(dir) {
      do {
        player.pos.x += dir;
      } while (!collide(arena, player));
      player.pos.x -= dir;
    }

    const leftBtn = document.getElementById('leftBtn');
    const rightBtn = document.getElementById('rightBtn');

    leftBtn.addEventListener('touchstart', () => moveToEdge(-1));
    rightBtn.addEventListener('touchstart', () => moveToEdge(1));

    document.addEventListener('keydown', event => {
      if (event.key === 'ArrowLeft') playerMove(-1);
      else if (event.key === 'ArrowRight') playerMove(1);
      else if (event.key === 'ArrowDown') playerDrop();
      else if (event.key === 'ArrowUp') playerRotate();
    });

    function playerMove(dir) {
      player.pos.x += dir;
      if (collide(arena, player)) {
        player.pos.x -= dir;
      }
    }

    // 其餘程式碼保留
  </script>
</body>
</html>
