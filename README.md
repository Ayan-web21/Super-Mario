<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Mini Mario Platformer</title>
<style>
  html, body {
    margin:0; padding:0; overflow: hidden; background: #5c94fc;
  }
  #game {
    background: #5c94fc;
    display: block;
    margin: 0 auto;
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
    bottom: 0;
    left: 100px;
    border-radius: 8px;
  }
  .platform {
    position: absolute;
    background: #654321;
    height: 20px;
  }
</style>
</head>
<body>

<div id="game">
  <div id="player"></div>
  <div class="platform" style="width: 800px; bottom: 0; left: 0;"></div>
  <div class="platform" style="width: 150px; bottom: 100px; left: 300px;"></div>
  <div class="platform" style="width: 100px; bottom: 200px; left: 500px;"></div>
</div>

<script>
(() => {
  const game = document.getElementById('game');
  const player = document.getElementById('player');
  const gravity = 0.6;
  const jumpStrength = 15;
  const moveSpeed = 5;

  let keys = {};
  let playerState = {
    x: 100,
    y: 0,
    width: 40,
    height: 60,
    velX: 0,
    velY: 0,
    onGround: false
  };

  const platforms = [...document.getElementsByClassName('platform')].map(el => ({
    x: el.offsetLeft,
    y: game.clientHeight - el.offsetTop - el.offsetHeight,
    width: el.offsetWidth,
    height: el.offsetHeight
  }));

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

    // Floor collision
    playerState.onGround = false;

    for (let plat of platforms) {
      // Player bounding box
      let playerRect = {
        x: playerState.x,
        y: playerState.y,
        width: playerState.width,
        height: playerState.height
      };

      // Platform bounding box
      let platRect = {
        x: plat.x,
        y: plat.y,
        width: plat.width,
        height: plat.height
      };

      // Check collision from above
      if (isColliding(playerRect, platRect)) {
        // Simple fix: if falling, put player on top of platform
        if (playerState.velY > 0 && playerState.y + playerState.height <= plat.y + playerState.velY) {
          playerState.y = plat.y - playerState.height;
          playerState.velY = 0;
          playerState.onGround = true;
        }
      }
    }

    // Prevent falling below floor
    if (playerState.y + playerState.height > game.clientHeight) {
      playerState.y = game.clientHeight - playerState.height;
      playerState.velY = 0;
      playerState.onGround = true;
    }

    // Apply to player div
    player.style.left = playerState.x + 'px';
    player.style.bottom = playerState.y + 'px';
  }

  function gameLoop() {
    update();
    requestAnimationFrame(gameLoop);
  }

  window.addEventListener('keydown', e => {
    keys[e.key] = true;
  });

  window.addEventListener('keyup', e => {
    keys[e.key] = false;
  });

  gameLoop();
})();
</script>

</body>
</html>
