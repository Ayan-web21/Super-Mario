<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Mini Mario Game with Coins, Enemies & Long Levels</title>
<style>
  html, body {
    margin: 0; padding: 0; overflow: hidden; background: #5c94fc;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    user-select: none;
  }
  #game {
    background: #5c94fc;
    position: relative;
    width: 1000px; /* viewport width */
    height: 400px;
    border: 2px solid #000;
    overflow: hidden;
    font-family: Arial, sans-serif;
  }
  #player {
    position: absolute;
    width: 40px;
    height: 60px;
    background: url('https://i.imgur.com/4Pm6nXh.png') no-repeat center bottom;
    background-size: contain;
    image-rendering: pixelated;
    transform: rotate(0deg);
    z-index: 20;
  }
  .platform {
    position: absolute;
    width: 50px;
    height: 50px;
    background: url('https://i.imgur.com/UvZ9IiX.png') no-repeat center;
    background-size: contain;
    image-rendering: pixelated;
  }
  .coin {
    position: absolute;
    width: 30px;
    height: 30px;
    background: url('https://i.imgur.com/dXzXhZR.png') no-repeat center;
    background-size: contain;
    image-rendering: pixelated;
    pointer-events: none;
    z-index: 15;
  }
  .enemy {
    position: absolute;
    width: 40px;
    height: 40px;
    background: url('https://i.imgur.com/b3bNEkH.png') no-repeat center bottom;
    background-size: contain;
    image-rendering: pixelated;
    z-index: 18;
  }
  #hud {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-size: 18px;
    text-shadow: 1px 1px 2px black;
    user-select: none;
    z-index: 30;
  }
  #message {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: yellow;
    font-size: 32px;
    font-weight: bold;
    text-shadow: 2px 2px 5px black;
    display: none;
    z-index: 40;
    user-select: none;
  }
</style>
</head>
<body>

<div id="game">
  <div id="hud">Level: 1 | Score: 0</div>
  <div id="message">Game Over! Press R to Restart</div>
  <div id="player"></div>
</div>

<script>
(() => {
  const game = document.getElementById('game');
  const player = document.getElementById('player');
  const hud = document.getElementById('hud');
  const message = document.getElementById('message');

  const gravity = 0.6;
  const jumpStrength = 15;
  const moveSpeed = 5;
  const blockSize = 50;

  let keys = {};
  let currentLevel = 0;
  let score = 0;
  let gameOver = false;

  // Levels longer: 60 blocks wide.
  // Platforms: x,y,widthBlocks
  // Coins: placed on platforms
  // Enemies: patrol platform between two ends

  const levels = [
    { // Level 1
      platforms: [
        { x: 0, y: 350, widthBlocks: 60 },
        { x: 10, y: 280, widthBlocks: 5 },
        { x: 20, y: 220, widthBlocks: 3 },
        { x: 30, y: 270, widthBlocks: 4 },
        { x: 45, y: 230, widthBlocks: 3 }
      ],
      coins: [
        { x: 12 * blockSize + 10, y: 240 },
        { x: 21 * blockSize + 10, y: 180 },
        { x: 31 * blockSize + 10, y: 230 },
        { x: 46 * blockSize + 10, y: 190 },
        { x: 50 * blockSize + 10, y: 320 }
      ],
      enemies: [
        { xStart: 15 * blockSize, xEnd: 20 * blockSize, y: 300, speed: 2 },
        { xStart: 35 * blockSize, xEnd: 39 * blockSize, y: 320, speed: 3 }
      ]
    },
    { // Level 2
      platforms: [
        { x: 0, y: 350, widthBlocks: 60 },
        { x: 5, y: 310, widthBlocks: 6 },
        { x: 15, y: 260, widthBlocks: 4 },
        { x: 25, y: 220, widthBlocks: 5 },
        { x: 40, y: 280, widthBlocks: 4 },
        { x: 50, y: 240, widthBlocks: 3 }
      ],
      coins: [
        { x: 6 * blockSize + 10, y: 270 },
        { x: 16 * blockSize + 10, y: 210 },
        { x: 26 * blockSize + 10, y: 170 },
        { x: 41 * blockSize + 10, y: 240 },
        { x: 51 * blockSize + 10, y: 200 }
      ],
      enemies: [
        { xStart: 10 * blockSize, xEnd: 15 * blockSize, y: 330, speed: 2.5 },
        { xStart: 45 * blockSize, xEnd: 49 * blockSize, y: 310, speed: 3 }
      ]
    },
    { // Level 3
      platforms: [
        { x: 0, y: 350, widthBlocks: 60 },
        { x: 8, y: 320, widthBlocks: 5 },
        { x: 18, y: 280, widthBlocks: 5 },
        { x: 28, y: 240, widthBlocks: 4 },
        { x: 38, y: 200, widthBlocks: 4 },
        { x: 48, y: 160, widthBlocks: 3 }
      ],
      coins: [
        { x: 9 * blockSize + 10, y: 280 },
        { x: 19 * blockSize + 10, y: 240 },
        { x: 29 * blockSize + 10, y: 200 },
        { x: 39 * blockSize + 10, y: 160 },
        { x: 49 * blockSize + 10, y: 120 }
      ],
      enemies: [
        { xStart: 15 * blockSize, xEnd: 20 * blockSize, y: 330, speed: 3 },
        { xStart: 35 * blockSize, xEnd: 40 * blockSize, y: 300, speed: 2 }
      ]
    }
  ];

  const levelWidthPx = 60 * blockSize;

  let playerState = {
    x: 100,
    y: 0,
    width: 40,
    height: 60,
    velX: 0,
    velY: 0,
    onGround: false
  };

  let platformElements = [];
  let platformRects = [];

  let coinElements = [];
  let coins = [];

  let enemyElements = [];
  let enemies = [];

  function createPlatforms(levelPlatforms) {
    platformElements.forEach(el => game.removeChild(el));
    platformElements = [];
    platformRects = [];

    for (let plat of levelPlatforms) {
      for (let i = 0; i < plat.widthBlocks; i++) {
        const block = document.createElement('div');
        block.className = 'platform';
        block.style.left = (plat.x * blockSize + i * blockSize) + 'px';
        block.style.top = plat.y + 'px';
        game.appendChild(block);
        platformElements.push(block);

        platformRects.push({
          x: plat.x * blockSize + i * blockSize,
          y: plat.y,
          width: blockSize,
          height: blockSize
        });
      }
    }
  }

  function createCoins(levelCoins) {
    coinElements.forEach(el => game.removeChild(el));
    coinElements = [];
    coins = [];

    for (let c of levelCoins) {
      const coin = document.createElement('div');
      coin.className = 'coin';
      coin.style.left = c.x + 'px';
      coin.style.top = c.y + 'px';
      game.appendChild(coin);
      coinElements.push(coin);
      coins.push({
        x: c.x,
        y: c.y,
        width: 30,
        height: 30,
        collected: false
      });
    }
  }

  function createEnemies(levelEnemies) {
    enemyElements.forEach(el => game.removeChild(el));
    enemyElements = [];
    enemies = [];

    for (let e of levelEnemies) {
      const enemy = document.createElement('div');
      enemy.className = 'enemy';
      enemy.style.left = e.xStart + 'px';
      enemy.style.top = e.y + 'px';
      game.appendChild(enemy);
      enemyElements.push(enemy);
      enemies.push({
        xStart: e.xStart,
        xEnd: e.xEnd,
        y: e.y,
        x: e.xStart,
        width: 40,
        height: 40,
        speed: e.speed,
        direction: 1 // 1 = right, -1 = left
      });
    }
  }

  function isColliding(rect1, rect2) {
    return !(rect1.x + rect1.width <= rect2.x ||
             rect1.x >= rect2.x + rect2.width ||
             rect1.y + rect1.height <= rect2.y ||
             rect1.y >= rect2.y + rect2.height);
  }

  function update() {
    if (gameOver) return;

    if (keys['ArrowLeft'] || keys['a']) {
      playerState.velX = -moveSpeed;
    } else if (keys['ArrowRight'] || keys['d']) {
      playerState.velX = moveSpeed;
    } else {
      playerState.velX = 0;
    }

    if ((keys['ArrowUp'] || keys['w'] || keys[' ']) && playerState.onGround) {
      playerState.velY = -jumpStrength;
      playerState.onGround = false;
    }

    playerState.velY += gravity;

    playerState.x += playerState.velX;
    playerState.y += playerState.velY;

    // Boundaries horizontally
    if (playerState.x < 0) playerState.x = 0;
    if (playerState.x + playerState.width > levelWidthPx) playerState.x = levelWidthPx - playerState.width;

    playerState.onGround = false;

    // Platform collision
    for (let plat of platformRects) {
      let playerRect = {
        x: playerState.x,
        y: playerState.y,
        width: playerState.width,
        height: playerState.height
      };

      if (isColliding(playerRect, plat)) {
        if (playerState.velY > 0 && (playerState.y + playerState.height) <= (plat.y + playerState.velY)) {
          playerState.y = plat.y - playerState.height;
          playerState.velY = 0;
          playerState.onGround = true;
        } else if (playerState.velY < 0 && playerState.y >= plat.y + plat.height) {
          playerState.y = plat.y + plat.height;
          playerState.velY = 0;
        } else {
          if (playerState.velX > 0) {
            playerState.x = plat.x - playerState.width;
          } else if (playerState.velX < 0) {
            playerState.x = plat.x + plat.width;
          }
          playerState.velX = 0;
        }
      }
    }

    // Ground limit bottom
    if (playerState.y + playerState.height > game.clientHeight) {
      playerState.y = game.clientHeight - playerState.height;
      playerState.velY = 0;
      playerState.onGround = true;
    }

    // Collect coins
    coins.forEach((coin, i) => {
      if (!coin.collected) {
        if (isColliding({
          x: playerState.x,
          y: playerState.y,
          width: playerState.width,
          height: playerState.height
        }, coin)) {
          coin.collected = true;
          coinElements[i].style.display = 'none';
          score++;
          hud.textContent = `Level: ${currentLevel + 1} | Score: ${score}`;
        }
      }
    });

    // Enemy movement and collision
    for (let i = 0; i < enemies.length; i++) {
      let enemy = enemies[i];
      enemy.x += enemy.speed * enemy.direction;

      if (enemy.x > enemy.xEnd) {
        enemy.x = enemy.xEnd;
        enemy.direction = -1;
      }
      if (enemy.x < enemy.xStart) {
        enemy.x = enemy.xStart;
        enemy.direction = 1;
      }

      enemyElements[i].style.left = enemy.x + 'px';

      if (isColliding({
        x: playerState.x,
        y: playerState.y,
        width: playerState.width,
        height: playerState.height
      }, enemy)) {
        gameOver = true;
        message.style.display = 'block';
      }
    }

    // Check level end (reach right edge)
    if (!gameOver && playerState.x >= levelWidthPx - playerState.width) {
      currentLevel++;
      if (currentLevel >= levels.length) currentLevel = 0;
      loadLevel(currentLevel);
    }
  }

  function gameLoop() {
    update();

    // Camera scrolling - keep player roughly centered horizontally
    const camX = Math.min(Math.max(playerState.x - game.clientWidth / 2 + playerState.width / 2, 0), levelWidthPx - game.clientWidth);

    platformElements.forEach((block, i) => {
      block.style.left = (platformRects
