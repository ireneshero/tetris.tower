#tetris.tower
<!DOCTYPE html>
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
        <button class="control-btn" onclick="playerRotate()">↑</button>
      </div>
      <div class="control-row">
        <button class="control-btn" onclick="playerMove(-1)">←</button>
        <button class="control-btn" id="downBtn">↓</button>
        <button class="control-btn" onclick="playerMove(1)">→</button>
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
      if (!pause) {
        pauseBtn.textContent = 'Pause';
        update();
      } else {
        pauseBtn.textContent = 'Continue';
        cancelAnimationFrame(animationFrameId);
      }
    });

    const downBtn = document.getElementById('downBtn');
    let downInterval;
    downBtn.addEventListener('touchstart', () => {
      downInterval = setInterval(playerDrop, 50);
    });
    downBtn.addEventListener('touchend', () => {
      clearInterval(downInterval);
    });

    function arenaSweep() {
      let rowCount = 1;
      outer: for (let y = arena.length - 1; y >= 0; --y) {
        for (let x = 0; x < arena[y].length; ++x) {
          if (arena[y][x] === 0) {
            continue outer;
          }
        }
        const row = arena.splice(y, 1)[0].fill(0);
        arena.unshift(row);
        ++y;
        score += rowCount * 10;
        rowCount *= 2;
      }
    }

    function collide(arena, player) {
      const [m, o] = [player.matrix, player.pos];
      for (let y = 0; y < m.length; ++y) {
        for (let x = 0; x < m[y].length; ++x) {
          if (m[y][x] !== 0 &&
              (arena[y + o.y] &&
               arena[y + o.y][x + o.x]) !== 0) {
            return true;
          }
        }
      }
      return false;
    }

    function createMatrix(w, h) {
      const matrix = [];
      while (h--) matrix.push(new Array(w).fill(0));
      return matrix;
    }

    function createPiece(type) {
      const pieces = {
        T: [[0,0,0],[1,1,1],[0,1,0]],
        O: [[2,2],[2,2]],
        L: [[0,3,0],[0,3,0],[0,3,3]],
        J: [[0,4,0],[0,4,0],[4,4,0]],
        I: [[0,5,0,0],[0,5,0,0],[0,5,0,0],[0,5,0,0]],
        S: [[0,6,6],[6,6,0],[0,0,0]],
        Z: [[7,7,0],[0,7,7],[0,0,0]]
      };
      return pieces[type];
    }

    function drawMatrix(matrix, offset) {
      matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            context.fillStyle = '#333';
            context.fillRect(x + offset.x, y + offset.y, 1, 1);
          }
        });
      });
    }

    function draw() {
      context.fillStyle = '#fff';
      context.fillRect(0, 0, canvas.width, canvas.height);
      drawMatrix(arena, { x: 0, y: 0 });
      drawMatrix(player.matrix, player.pos);
    }

    function merge(arena, player) {
      player.matrix.forEach((row, y) => {
        row.forEach((value, x) => {
          if (value !== 0) {
            arena[y + player.pos.y][x + player.pos.x] = value;
          }
        });
      });
    }

    function showGameOver() {
      document.getElementById('game-over').style.display = 'block';
    }

    function hideGameOver() {
      document.getElementById('game-over').style.display = 'none';
    }

    function restartGame() {
      arena = createMatrix(12, 20);
      score = 0;
      updateScore();
      playerReset();
      hideGameOver();
      pause = false;
      dropCounter = 0;
      lastTime = 0;
      pauseBtn.textContent = 'Pause';
      update();
    }

    function playerDrop() {
      player.pos.y++;
      if (collide(arena, player)) {
        player.pos.y--;
        merge(arena, player);
        playerReset();
        arenaSweep();
        updateScore();
      }
      dropCounter = 0;
    }

    function playerMove(dir) {
      player.pos.x += dir;
      if (collide(arena, player)) {
        player.pos.x -= dir;
      }
    }

    function playerReset() {
      const pieces = 'TJLOSZI';
      player.matrix = createPiece(pieces[Math.floor(Math.random() * pieces.length)]);
      player.pos.y = 0;
      player.pos.x = Math.floor(arena[0].length / 2) - Math.floor(player.matrix[0].length / 2);
      if (collide(arena, player)) {
        showGameOver();
        pause = true;
      }
    }

    function playerRotate() {
      const m = player.matrix;
      for (let y = 0; y < m.length; ++y) {
        for (let x = 0; x < y; ++x) {
          [m[x][y], m[y][x]] = [m[y][x], m[x][y]];
        }
      }
      m.forEach(row => row.reverse());
      if (collide(arena, player)) {
        for (let i = 0; i < 3; ++i) playerRotate();
      }
    }

    let dropCounter = 0;
    let dropInterval = 1000;

    let lastTime = 0;
    function update(time = 0) {
      if (pause) return;
      const deltaTime = time - lastTime;
      lastTime = time;
      dropCounter += deltaTime;
      if (dropCounter > dropInterval) {
        playerDrop();
      }
      draw();
      animationFrameId = requestAnimationFrame(update);
    }

    document.addEventListener('keydown', event => {
      if (event.key === 'ArrowLeft') playerMove(-1);
      else if (event.key === 'ArrowRight') playerMove(1);
      else if (event.key === 'ArrowDown') playerDrop();
      else if (event.key === 'ArrowUp') playerRotate();
    });

    let arena = createMatrix(12, 20);
    const player = { pos: { x: 0, y: 0 }, matrix: null };

    playerReset();
    updateScore();
    update();
  </script>
</body>
</html>
