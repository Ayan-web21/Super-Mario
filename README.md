<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Mini Mario Multi-Level</title>
<style>
  html, body {
    margin: 0; padding: 0; overflow: hidden; background: #5c94fc;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #game {
    background: #5c94fc;
    position: relative;
    width: 800px;
    height: 400px;
    border: 2px solid #000;
    overflow: hidden;
  }
  #player {
    position: absolute;
    width: 40px;
    height: 60px;
    background: red;
    border-radius: 8px;
  }
  .platform {
    position: absolute;
    background: #654321;
    height: 20px;
    border-radius: 4px;
  }
  #levelIndicator {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-family: Arial, sans-serif;
    font-size: 18px;
    text-shadow: 1px 1px 2px black;
    user-select: none;
  }
</style>
</head>
<body>

<div id="game">
  <div id="levelIndicator">Level: 1 (Press 1,2,3 to switch)</div>
  <div id="player"></div>
</div>

<script>
(() => {
  const game = document.getElementById('game');
  const player = document.getElementById('player');
  const levelIndicator = document.getElementById('levelIndicator');

  const gravity = 0.6;
  const jumpStrength = 15;
  const moveSpeed = 5;

  let keys = {};
  let currentLevel = 0;

  const levels = [
    // Level 1
    [
      { x: 0, y: 380, width: 800, height: 20 },     // ground
      { x: 300, y: 300, width: 150, height: 20 },
      { x: 500, y: 200, width: 100, height: 20 }
    ],
    // Level 2
    [
      { x: 0, y: 380, width: 800, height: 20 },     // ground
      { x: 100, y: 320, width: 100, height: 20 },
      { x: 300, y: 260, width: 200, height: 20 },
      { x: 600, y: 300, width: 150, height: 20 }
    ],
    // Level 3
    [
      { x: 0, y: 380, width: 800, height: 20 },     // ground
      { x: 150, y: 350, width: 100, height: 20 },
      { x: 350, y: 300, width: 120, height: 20 },
      { x: 550, y: 250, width: 80, height: 20 },
      { x: 700, y: 200, width: 60, height: 20 }
    ],
  ];

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

  function createPlatforms(levelPlatforms) {
    // Remove old platforms
    platformElements.forEach(el => game.removeChild(el));
    platformElements = [];

    // Add new platforms
    for (let plat of levelPlatforms) {
      const el = document.createElement('div');
      el.className = 'platform';
      el.style.left = plat.x + 'px';
      el.style.top = plat.y + 'px';
      el.style.width = plat.width + 'px';
      el.style.height = plat.height + 'px';
      game.appendChild(el);
      platformElements.push(el);
    }
  }

  function isColliding(rect1, rect2) {
    return !(rect1.x + rect1.width < rect2.x ||
             rect1.x > rect2.x + rect2.width ||
             rect1.y + rect1.height < rect2.y ||
             rect1.y > rect2.y + rect2.height);
  }

  function update() {
    // Horizontal movement
    if (keys['ArrowLeft'] || keys['a']) {
      playerState.velX = -moveSpeed;
    } else if (keys['ArrowRight'] || keys['d']) {
      playerState.velX = moveSpeed;
    } else {
      playerState.velX = 0;
    }

    // Jump
    if ((keys['ArrowUp'] || keys['w'] || keys[' ']) && playerState.onGround) {
      playerState.velY = -jumpStrength;
      playerState.onGround = false;
    }

    // Apply gravity
    playerState.velY += gravity;

    // Update position
    playerState.x += playerState.velX;
    playerState.y += playerState.velY;

    // Keep inside game bounds horizontally
    if (playerState.x < 0) playerState.x = 0;
    if (playerState.x + playerState.width > game.clientWidth) playerState.x = game.clientWidth - playerState.width;

    // Reset onGround and check platform collisions
    playerState.onGround = false;

    for (let plat of levels[currentLevel]) {
      let playerRect = {
        x: playerState.x,
        y: playerState.y,
        width: playerState.width,
        height: playerState.height
      };

      let platRect = {
        x: plat.x,
        y: plat.y,
        width: plat.width,
        height: plat.height
      };

      if (isColliding(playerRect, platRect)) {
        // Check if player is falling and above platform
        if (playerState.velY > 0 && (playerState.y + playerState.height) <= (plat.y + playerState.velY)) {
          playerState.y = plat.y - playerState.height;
          playerState.velY = 0;
          playerState.onGround = true;
        } else if (playerState.velY < 0 && playerState.y >= plat.y + plat.height) {
          // Head bump - prevent player from going through platform top from below
          playerState.y = plat.y + plat.height;
          playerState.velY = 0;
        } else {
          // Side collisions: prevent moving through platforms horizontally
          if (playerState.velX > 0) {
            playerState.x = plat.x - playerState.width;
          } else if (playerState.velX < 0) {
            playerState.x = plat.x + plat.width;
          }
          playerState.velX = 0;
        }
      }
    }

    // Prevent falling below floor
    if (playerState.y + playerState.height > game.clientHeight) {
      playerState.y = game.clientHeight - playerState.height;
      playerState.velY = 0;
      playerState.onGround = true;
    }

    // Apply position to player element
    player.style.left = playerState.x + 'px';
    player.style.top = playerState.y + 'px';
  }

  function gameLoop() {
    update();
    requestAnimationFrame(gameLoop);
  }

  // Switch levels with number keys 1, 2, 3
  window.addEventListener('keydown', e => {
    keys[e.key] = true;
    if (e.key >= '1' && e.key <= '3') {
      currentLevel = parseInt(e.key) - 1;
      levelIndicator.textContent = `Level: ${e.key} (Press 1,2,3 to switch)`;
      createPlatforms(levels[currentLevel]);
      // Reset player position
      playerState.x = 100;
      playerState.y = 0;
      playerState.velX = 0;
      playerState.velY = 0;
      playerState.onGround = false;
    }
  });

  window.addEventListener('keyup', e => {
    keys[e.key] = false;
  });

  // Initialize
  createPlatforms(levels[currentLevel]);
  player.style.top = playerState.y + 'px';
  player.style.left = playerState.x + 'px';

  gameLoop();
})();
</script>

</body>
</html>
