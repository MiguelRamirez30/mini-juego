jumper-cubecoin.html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>JUMPER CUBECOIN - Mini Juego Saltar Obstáculos y Monedas</title>
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />
<style>
  /* Reset and base styles */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  body, html {
    height: 100%;
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(135deg, #7c3aed, #5a189a);
    color: white;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    user-select: none;
    min-height: 100vh;
  }

  #game-container {
    position: relative;
    width: 100%;
    height: 70vh;
    max-width: 900px;
    margin: 20px auto 8px auto;
    background: linear-gradient(180deg, #7c3aed 0%, #3b0764 100%);
    border-radius: 16px;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0,0,0,0.7);
    display: flex;
    align-items: flex-end;
  }

  #game-canvas {
    background: transparent;
    width: 100%;
    height: 100%;
    display: block;
  }

  #scoreboard {
    max-width: 900px;
    margin: 0 auto 12px auto;
    font-size: 1.4rem;
    text-align: center;
    letter-spacing: 1.2px;
    min-height: 30px;
  }

  .controls-container {
    max-width: 900px;
    margin: 8px auto 32px auto;
    display: flex;
    justify-content: center;
    gap: 20px;
  }

  .btn-game {
    background: linear-gradient(135deg, #a855f7 0%, #be4bdb 100%);
    border: none;
    padding: 14px 32px;
    font-size: 1.2rem;
    border-radius: 12px;
    cursor: pointer;
    color: white;
    transition: background 0.3s ease;
    min-width: 120px;
    user-select: none;
    font-weight: 600;
    box-shadow: 0 4px 10px rgba(123, 54, 197, 0.7);
  }
  .btn-game:hover:not(:disabled) {
    background: linear-gradient(135deg, #be4bdb 0%, #a855f7 100%);
  }
  .btn-game:disabled {
    background: #633c9c;
    cursor: not-allowed;
    box-shadow: none;
  }

  /* Accessibility focus */
  .btn-game:focus {
    outline: 3px solid #e0aaff;
    outline-offset: 3px;
  }

  /* Info text */
  #instructions {
    max-width: 900px;
    margin: 0 auto 24px auto;
    font-weight: 600;
    font-size: 1rem;
    text-align: center;
    color: #d8b4fe;
  }

  /* Overlay styles */
  #pause-overlay {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0,0,0,0.65);
    color: #d8b4fe;
    font-size: 4rem;
    font-weight: 900;
    display: flex;
    align-items: center;
    justify-content: center;
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.3s ease;
    user-select: none;
    z-index: 10;
  }
  #pause-overlay.visible {
    pointer-events: all;
    opacity: 1;
  }
  
  #exit-overlay {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    background: #3b0764;
    color: #e0aaff;
    font-size: 2rem;
    font-weight: 700;
    display: flex;
    align-items: center;
    justify-content: center;
    user-select: none;
    z-index: 15;
    padding: 16px;
    text-align: center;
    display: none;
  }
  #exit-overlay.visible {
    display: flex;
  }

  header h1 {
    letter-spacing: 0.1em;
  }
  #subtitle {
    text-align: center;
    font-weight: 400;
    font-style: italic;
    font-size: 1.1rem;
    color: #d8b4fe;
    margin-top: 4px;
    margin-bottom: 18px;
  }
</style>
</head>
<body>
  <header>
    <h1>JUMPER CUBECOIN</h1>
    <div id="subtitle" aria-label="Subtítulo del juego">Thebigmike.store creator</div>
    <!-- Alternative subtitle if needed: Thebigmike estudios -->
  </header>

  <div id="game-container" role="main" aria-label="Área del juego, presiona espacio para saltar">
    <canvas id="game-canvas" width="900" height="400" aria-live="polite" aria-atomic="true"></canvas>
    <div id="pause-overlay" role="alert" aria-live="assertive" aria-atomic="true">PAUSA</div>
    <div id="exit-overlay" role="alert" aria-live="assertive" aria-atomic="true">Juego finalizado. Gracias por jugar.</div>
  </div>

  <div id="scoreboard" aria-live="polite" aria-atomic="true" aria-label="Monedas recogidas: 0">Monedas: 0</div>

  <div class="controls-container" role="group" aria-label="Controles del juego">
    <button class="btn-game" id="start-btn" aria-label="Iniciar o reanudar juego">Iniciar</button>
    <button class="btn-game" id="pause-btn" aria-label="Pausar juego" disabled>Pausa</button>
    <button class="btn-game" id="exit-btn" aria-label="Salir del juego" disabled>Salir</button>
  </div>

  <div id="instructions" tabindex="0">Presiona la tecla <kbd>Espacio</kbd> para saltar y evitar obstáculos. ¡Recoge todas las monedas! Recoge el escudo para ser inmune temporalmente.</div>

  <script>
    (() => {
      const canvas = document.getElementById('game-canvas');
      const ctx = canvas.getContext('2d');
      const startBtn = document.getElementById('start-btn');
      const pauseBtn = document.getElementById('pause-btn');
      const exitBtn = document.getElementById('exit-btn');
      const scoreboard = document.getElementById('scoreboard');
      const pauseOverlay = document.getElementById('pause-overlay');
      const exitOverlay = document.getElementById('exit-overlay');

      // Constants
      const GRAVITY = 0.8;
      const JUMP_POWER = 15;
      const OBSTACLE_WIDTH = 40;
      const OBSTACLE_GAP_MIN = 1200; // ms
      const COIN_SIZE = 24;
      const PLAYER_WIDTH = 50;
      const PLAYER_HEIGHT = 50;
      const SHIELD_SIZE = 40;
      const SHIELD_DURATION = 7000; // 7 seconds immunity
      const SHIELD_COOLDOWN = 10000; // cooldown before next shield spawn

      // Game state
      let player = {
        x: 100,
        y: 0,
        width: PLAYER_WIDTH,
        height: PLAYER_HEIGHT,
        vy: 0,
        isJumping: false,
        onGround: true,
        shieldActive: false,
        shieldEnd: 0,
      };

      let obstacles = [];
      let coins = [];
      let shields = [];
      let lastObstacleTime = 0;
      let lastCoinTime = 0;
      let lastShieldTime = 0;
      let groundY = canvas.height - 80;
      let score = 0;
      let gameOver = false;
      let gameSpeed = 6;

      let isRunning = false;
      let isPaused = false;
      let animationFrameId = null;

      // Draw player
      const playerColor = '#f0abfc';

      // Images placeholders (coin, shield)
      const coinImg = new Image();
      coinImg.src = "https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/c86be084-5dcb-4ac3-aa3a-c95307ca8e75.png";
      coinImg.alt = "Moneda dorada brillante con brillo";

      const shieldImg = new Image();
      shieldImg.src = "https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/c5e865ee-4b43-40ca-b812-6f785817c708.png";
      shieldImg.alt = "Escudo protector azul brillante";

      // Draw player with shield glow and eyes
      function drawPlayer() {
        ctx.save();
        if (player.shieldActive) {
          ctx.shadowColor = 'rgba(59, 130, 246, 0.8)';
          ctx.shadowBlur = 20;
        }
        ctx.fillStyle = playerColor;
        ctx.fillRect(player.x, player.y, player.width, player.height);
        ctx.restore();

        ctx.fillStyle = '#3b0764';
        ctx.beginPath();
        ctx.ellipse(player.x + 12, player.y + 15, 6, 9, 0, 0, 2 * Math.PI);
        ctx.fill();
        ctx.beginPath();
        ctx.ellipse(player.x + player.width - 18, player.y + 15, 6, 9, 0, 0, 2 * Math.PI);
        ctx.fill();

        if (player.shieldActive) {
          ctx.save();
          ctx.globalAlpha = 0.25 + 0.25 * Math.sin(Date.now() / 250);
          ctx.strokeStyle = '#3b82f6';
          ctx.lineWidth = 6;
          ctx.beginPath();
          ctx.ellipse(player.x + player.width/2, player.y + player.height/2, player.width * 0.9, player.height * 0.9, 0, 0, 2 * Math.PI);
          ctx.stroke();
          ctx.restore();
        }
      }

      // Draw obstacles with types: spike, platform, block
      function drawObstacle(ob) {
        ctx.save();
        switch(ob.type) {
          case 'spike':
            ctx.fillStyle = '#666666';
            ctx.shadowColor = '#a1a1aa';
            ctx.shadowBlur = 8;
            ctx.beginPath();
            ctx.moveTo(ob.x, groundY);
            ctx.lineTo(ob.x + ob.width/2, groundY - ob.height);
            ctx.lineTo(ob.x + ob.width, groundY);
            ctx.closePath();
            ctx.fill();
            break;
          case 'platform':
            ctx.fillStyle = '#a855f7';
            ctx.shadowColor = '#7c3aed';
            ctx.shadowBlur = 12;
            ctx.fillRect(ob.x, groundY - ob.height, ob.width, 15);
            break;
          default:
            ctx.fillStyle = '#a855f7';
            ctx.shadowColor = '#7c3aed';
            ctx.shadowBlur = 12;
            ctx.fillRect(ob.x, groundY - ob.height, ob.width, ob.height);
        }
        ctx.restore();
      }

      // Draw coins
      function drawCoin(coin) {
        if (coinImg.complete) {
          ctx.drawImage(coinImg, coin.x, coin.y, COIN_SIZE, COIN_SIZE);
        } else {
          ctx.save();
          ctx.fillStyle = '#ffef00';
          ctx.beginPath();
          ctx.arc(coin.x + COIN_SIZE/2, coin.y + COIN_SIZE/2, COIN_SIZE/2, 0, Math.PI*2);
          ctx.fill();
          ctx.restore();
        }
      }
      // Draw shield
      function drawShield(shield) {
        if (shieldImg.complete) {
          ctx.drawImage(shieldImg, shield.x, shield.y, SHIELD_SIZE, SHIELD_SIZE);
        } else {
          ctx.save();
          ctx.fillStyle = '#3b82f6';
          ctx.beginPath();
          ctx.arc(shield.x + SHIELD_SIZE/2, shield.y + SHIELD_SIZE/2, SHIELD_SIZE/2, 0, Math.PI*2);
          ctx.fill();
          ctx.restore();
        }
      }

      // Create obstacles randomly
      function createObstacle() {
        const types = ['block', 'spike', 'platform'];
        const type = types[Math.floor(Math.random() * types.length)];
        let height;
        switch(type) {
          case 'spike':
            height = 30 + Math.random() * 20;
            break;
          case 'platform':
            height = 60 + Math.random() * 40;
            break;
          default:
            height = 40 + Math.random() * 40;
        }
        obstacles.push({
          x: canvas.width,
          width: OBSTACLE_WIDTH,
          height: height,
          type: type,
        });
      }

      // Create coins randomly
      function createCoin() {
        const yPos = groundY - COIN_SIZE - (Math.random() * 80);
        coins.push({
          x: canvas.width,
          y: yPos,
          size: COIN_SIZE,
        });
      }

      // Create shield power-ups randomly
      function createShield() {
        const yPos = groundY - SHIELD_SIZE - 30 - (Math.random() * 70);
        shields.push({
          x: canvas.width,
          y: yPos,
          size: SHIELD_SIZE,
        });
      }

      // Reset game state
      function resetGame() {
        player.y = groundY - player.height;
        player.vy = 0;
        player.isJumping = false;
        player.onGround = true;
        player.shieldActive = false;
        player.shieldEnd = 0;
        obstacles = [];
        coins = [];
        shields = [];
        lastObstacleTime = 0;
        lastCoinTime = 0;
        lastShieldTime = 0;
        score = 0;
        gameOver = false;
        isRunning = true;
        isPaused = false;
        pauseOverlay.classList.remove('visible');
        exitOverlay.classList.remove('visible');
        startBtn.textContent = 'Reanudar';
        pauseBtn.disabled = false;
        exitBtn.disabled = false;
        scoreboard.textContent = 'Monedas: 0';
        requestAnimationFrame(gameLoop);
      }

      // Collision detection helper
      function detectCollision(rect1, rect2) {
        return (
          rect1.x < (rect2.x + (rect2.width||rect2.size)) &&
          (rect1.x + rect1.width) > rect2.x &&
          rect1.y < (rect2.y + (rect2.height||rect2.size)) &&
          (rect1.y + rect1.height) > rect2.y
        );
      }

      // Update player vertical movement and platform landing logic
      function updatePlayer() {
        player.vy += GRAVITY;
        player.y += player.vy;

        let onAnyPlatform = false;
        for (const ob of obstacles) {
          if (ob.type === 'platform') {
            const platformTop = groundY - ob.height;
            if (
              player.x + player.width > ob.x &&
              player.x < ob.x + ob.width &&
              player.y + player.height > platformTop &&
              player.y + player.height < platformTop + 20 &&
              player.vy >= 0
            ) {
              player.y = platformTop - player.height;
              player.vy = 0;
              player.isJumping = false;
              player.onGround = true;
              onAnyPlatform = true;
              break;
            }
          }
        }
        if (!onAnyPlatform) {
          if (player.y >= groundY - player.height) {
            player.y = groundY - player.height;
            player.vy = 0;
            player.isJumping = false;
            player.onGround = true;
          } else {
            player.onGround = false;
          }
        }
      }

      // Update obstacles and spawn new ones
      function updateObstacles() {
        obstacles.forEach(ob => {
          ob.x -= gameSpeed;
        });
        obstacles = obstacles.filter(ob => ob.x + ob.width > 0);
        if (Date.now() - lastObstacleTime > OBSTACLE_GAP_MIN + Math.random()*800) {
          createObstacle();
          lastObstacleTime = Date.now();
        }
      }

      // Update coins and spawn new ones
      function updateCoins() {
        coins.forEach(c => {
          c.x -= gameSpeed;
        });
        coins = coins.filter(c => c.x + c.size > 0);
        if (Date.now() - lastCoinTime > 1500 + Math.random() * 1500) {
          createCoin();
          lastCoinTime = Date.now();
        }
      }

      // Update shields and spawn with cooldown
      function updateShields() {
        shields.forEach(s => {
          s.x -= gameSpeed;
        });
        shields = shields.filter(s => s.x + s.size > 0);
        if (!player.shieldActive && Date.now() - lastShieldTime > SHIELD_COOLDOWN + Math.random() * 10000) {
          createShield();
          lastShieldTime = Date.now();
        }
        if (player.shieldActive && Date.now() > player.shieldEnd) {
          player.shieldActive = false;
        }
      }

      // Main game loop draws everything and checks collisions
      function gameLoop() {
        if (!isRunning || isPaused) return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        ctx.strokeStyle = '#be4bdb';
        ctx.lineWidth = 3;
        ctx.beginPath();
        ctx.moveTo(0, groundY);
        ctx.lineTo(canvas.width, groundY);
        ctx.stroke();

        updatePlayer();
        updateObstacles();
        updateCoins();
        updateShields();

        drawPlayer();
        obstacles.forEach(drawObstacle);
        coins.forEach(drawCoin);
        shields.forEach(drawShield);

        for (const ob of obstacles) {
          if (ob.type !== 'platform' && detectCollision(player, { x: ob.x, y: groundY - ob.height, width: ob.width, height: ob.height })) {
            if (!player.shieldActive) {
              gameOver = true;
              endGame();
              return;
            }
          }
        }

        for (let i = coins.length - 1; i >= 0; i--) {
          if (detectCollision(player, {x: coins[i].x, y: coins[i].y, width: COIN_SIZE, height: COIN_SIZE})) {
            score++;
            scoreboard.textContent = 'Monedas: ' + score;
            coins.splice(i, 1);
          }
        }

        for (let i = shields.length - 1; i >= 0; i--) {
          if (detectCollision(player, {x: shields[i].x, y: shields[i].y, width: SHIELD_SIZE, height: SHIELD_SIZE})) {
            player.shieldActive = true;
            player.shieldEnd = Date.now() + SHIELD_DURATION;
            shields.splice(i, 1);
          }
        }

        animationFrameId = requestAnimationFrame(gameLoop);
      }

      function endGame() {
        isRunning = false;
        pauseBtn.disabled = true;
        startBtn.textContent = 'Reiniciar';
        exitBtn.disabled = true;
        pauseOverlay.classList.remove('visible');
        exitOverlay.classList.add('visible');
        cancelAnimationFrame(animationFrameId);
      }

      function togglePause() {
        if (!isRunning) return;
        if (isPaused) {
          isPaused = false;
          pauseOverlay.classList.remove('visible');
          startBtn.disabled = false;
          pauseBtn.textContent = 'Pausa';
          requestAnimationFrame(gameLoop);
        } else {
          isPaused = true;
          pauseOverlay.classList.add('visible');
          startBtn.disabled = false;
          pauseBtn.textContent = 'Continuar';
          cancelAnimationFrame(animationFrameId);
        }
      }

      function exitGame() {
        isRunning = false;
        gameOver = true;
        isPaused = false;
        pauseOverlay.classList.remove('visible');
        exitOverlay.classList.add('visible');
        startBtn.disabled = true;
        pauseBtn.disabled = true;
        exitBtn.disabled = true;
        cancelAnimationFrame(animationFrameId);
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        scoreboard.textContent = 'Juego cerrado';
      }

      window.addEventListener('keydown', (e) => {
        if (!isRunning || isPaused) return;
        if (e.code === 'Space' || e.key === ' ') {
          e.preventDefault();
          if (!player.isJumping && player.onGround) {
            player.vy = -JUMP_POWER;
            player.isJumping = true;
            player.onGround = false;
          }
        }
      });

      startBtn.addEventListener('click', () => {
        if (!isRunning || gameOver) {
          resetGame();
        } else if (isPaused) {
          togglePause();
        }
      });

      pauseBtn.addEventListener('click', () => {
        togglePause();
      });

      exitBtn.addEventListener('click', () => {
        exitGame();
      });

      resetGame();
      togglePause();

    })();
  </script>
</body>
</html>




