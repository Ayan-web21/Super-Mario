<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Simple Side-Scrolling Mario Game</title>
<style>
  html, body {
    margin: 0; padding: 0; overflow: hidden; background: #5c94fc; font-family: Arial, sans-serif;
    user-select:none;
  }
  #game {
    position: relative;
    width: 1000px;
    height: 400px;
    margin: 20px auto;
    border: 3px solid black;
    background: #5c94fc;
    overflow: hidden;
  }
  #viewport {
    position: absolute;
    top: 0; left: 0; height: 100%;
    width: 100000px;
    will-change: transform;
  }
  #player {
    position: absolute;
    width: 40px;
    height: 60px;
    bottom: 0;
    background: url('https://i.imgur.com/4Pm6nXh.png') no-repeat center bottom;
    background-size: contain;
    image-rendering: pixelated;
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
    <div id="viewport"></div>
    <div id="player"></div>
  </div>

<script>
(() => {
  const game = document.getElementById('game');
  const viewport = document.getElementById('viewport');
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

  const levels = [
    {
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
    {
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
    {
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
    platformElements.forEach(el => viewport.removeChild(el));
    platformElements = [];
    platformRects = [];

    for (let plat of levelPlatforms) {
      for (let i = 0; i < plat.widthBlocks; i++) {
        const block = document.createElement('div');
        block.className = 'platform';
        block.style.left = (plat.x * blockSize + i * blockSize) + 'px';
        block.style.top = plat.y + 'px';
        viewport.appendChild(block);
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
    coinElements.forEach(el => viewport.removeChild(el));
    coinElements = [];
    coins = [];

    for (let c of levelCoins) {
      const coin = document.createElement('div');
      coin.className = 'coin';
      coin.style.left = c.x + 'px';
      coin.style.top = c.y + 'px';
      viewport.appendChild(coin);
      coinElements.push(coin);
      coins.push({ x: c.x, y: c.y, width: 30, height: 30, collected: false });
    }
  }

  function createEnemies(levelEnemies) {
    enemyElements.forEach(el => viewport.removeChild(el));
    enemyElements = [];
    enemies = [];

    for (let e of levelEnemies) {
      const enemy = document.createElement('div');
      enemy.className = 'enemy';
      enemy.style.left = e.xStart + 'px';
      enemy.style.top = e.y + 'px';
      viewport.appendChild(enemy);
      enemyElements.push(enemy);
      enemies.push({
        x: e.xStart,
        y: e.y,
        width: 40,
        height: 40,
        speed: e.speed,
        xStart: e.xStart,
        xEnd: e.xEnd,
        direction: 1
      });
    }
  }

  function rectsOverlap(r1, r2) {
    return !(r2.x > r1.x + r1.width ||
             r2.x + r2.width < r1.x ||
             r2.y > r1.y + r1.height ||
             r2.y + r2.height < r1.y);
  }

  function resetLevel() {
    const level = levels[currentLevel];
    createPlatforms(level.platforms);
    createCoins(level.coins);
    createEnemies(level.enemies);

    playerState.x = 100;
    playerState.y = 0;
    playerState.velX = 0;
    playerState.velY = 0;
    playerState.onGround = false;

    viewport.style.transform = `translateX(0px)`;
    score = 0;
    gameOver = false;
    message.style.display = 'none';

    updateHUD();
  }

  function updateHUD() {
    hud.textContent = `Level: ${currentLevel + 1} | Score: ${score}`;
  }

  function update() {
    if (gameOver) return;

    // Apply gravity
    playerState.velY += gravity;
    playerState.y += playerState.velY;

    // Horizontal movement
    if (keys['ArrowLeft'] || keys['a']) {
      playerState.velX = -moveSpeed;
    } else if (keys['ArrowRight'] || keys['d']) {
      playerState.velX = moveSpeed;
    } else {
      playerState.velX = 0;
    }
    playerState.x += playerState.velX;

    // Collision with platforms (vertical)
    playerState.onGround = false;
    for (let plat of platformRects) {
      // Check vertical collision only if player is falling or moving up
      if (
        playerState.x + playerState.width > plat.x &&
        playerState.x < plat.x + plat.width
      ) {
        // falling down collision
        if (
          playerState.y + playerState.height > plat.y &&
          playerState.y + playerState.height - playerState.velY <= plat.y
        ) {
          playerState.y = plat.y - playerState.height;
          playerState.velY = 0;
          playerState.onGround = true;
        }
      }
    }

    // Prevent player from falling below ground (bottom of game)
    if (playerState.y + playerState.height > game.clientHeight) {
      playerState.y = game.clientHeight - playerState.height;
      playerState.velY = 0;
      playerState.onGround = true;
    }

    // Prevent player from going left beyond 
