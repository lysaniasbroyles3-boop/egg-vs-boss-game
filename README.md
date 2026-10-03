# egg-vs-boss-game
Android Java egg boss game prototype with WASD controls and 100-level progression
      state.bossMaxHP = 100;
      state.bossHP = state.bossMaxHP;
      state.weaponLevel = 1;
      state.fireRate = 400; // Milliseconds between automatic shots
      state.damage = 10;
      state.projectiles.forEach(p => p.el.remove());
      state.bossProjectiles.forEach(p => p.el.remove());
      state.projectiles = [];
      state.bossProjectiles = [];
      state.lastShotTime = 0;
      state.lastBossShotTime = 0;
      
      // Update UI elements
      levelEl.textContent = state.level;
      playerHPEl.textContent = state.playerHP;
      bossHPEl.textContent = state.bossHP;
      weaponLevelEl.textContent = "LV " + state.weaponLevel;
      bossHealthFill.style.width = "100%";
      
      // Position entities
      player.style.left = state.playerX + 'px';
      state.bossX = 800;
      boss.style.left = state.bossX + 'px';
      boss.style.top = '200px';

      startScreen.classList.add('hidden');
      pauseScreen.classList.add('hidden');
      levelUpScreen.classList.add('hidden');
      gameOverScreen.classList.add('hidden');

      if (state.gameLoopId) cancelAnimationFrame(state.gameLoopId);
      state.gameLoopId = requestAnimationFrame(gameLoop);
    }

    // Toggle pause and adjust views safely
    function togglePause() {
      if (!state.playing || levelUpScreen.classList.contains('hidden') === false) return;
      state.paused = !state.paused;
      if (state.paused) {
        pauseScreen.classList.remove('hidden');
      } else {
        pauseScreen.classList.add('hidden');
        state.gameLoopId = requestAnimationFrame(gameLoop);
      }
    }

    // Input collection hooks
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') {
        e.preventDefault();
        if (!state.playing) {
          resetGame();
        } else {
          togglePause();
        }
      }
      state.keys[e.code] = true;
    });

    window.addEventListener('keyup', (e) => {
      state.keys[e.code] = false;
    });

    document.getElementById('startButton').addEventListener('click', resetGame);
    document.getElementById('restartButton').addEventListener('click', resetGame);
    document.getElementById('resumeButton').addEventListener('click', togglePause);
    document.getElementById('menuButton').addEventListener('click', () => {
      pauseScreen.classList.add('hidden');
      startScreen.classList.remove('hidden');
      state.playing = false;
    });

    // Level progression updates
    function nextLevel(upgradeType) {
      if (upgradeType === 'rate') {
        state.fireRate = Math.max(120, state.fireRate * 0.8);
      } else if (upgradeType === 'damage') {
        state.damage += 4;
      } else if (upgradeType === 'hp') {
        state.playerMaxHP += 50;
      }

      state.level++;
      state.bossMaxHP = Math.floor(100 * Math.pow(1.4, state.level));
      state.bossHP = state.bossMaxHP;
      state.playerHP = state.playerMaxHP;
      state.weaponLevel++;

      // UI reset tasks
      levelEl.textContent = state.level;
      playerHPEl.textContent = state.playerHP;
      bossHPEl.textContent = state.bossHP;
      weaponLevelEl.textContent = "LV " + state.weaponLevel;
      bossHealthFill.style.width = "100%";

      // Clear layout hazards
      state.projectiles.forEach(p => p.el.remove());
      state.bossProjectiles.forEach(p => p.el.remove());
      state.projectiles = [];
      state.bossProjectiles = [];

      levelUpScreen.classList.add('hidden');
      state.paused = false;
      state.gameLoopId = requestAnimationFrame(gameLoop);
    }

    // Wiring upgrade trigger inputs
    document.getElementById('upgrade1').addEventListener('click', () => nextLevel('rate'));
    document.getElementById('upgrade2').addEventListener('click', () => nextLevel('damage'));
    document.getElementById('upgrade3').addEventListener('click', () => nextLevel('hp'));

    function triggerGameOver() {
      state.playing = false;
      finalScoreText.textContent = "You reached Level: " + state.level;
      gameOverScreen.classList.remove('hidden');
    }

    function triggerLevelComplete() {
      state.paused = true;
      levelUpScreen.classList.remove('hidden');
    }

    // Core Animation Logic
    function gameLoop(timestamp) {
      if (!state.playing || state.paused) return;

      // Handle structural movement transformations (Bounds check)
      if (state.keys['ArrowLeft'] || state.keys['KeyA']) {
        state.playerX = Math.max(0, state.playerX - 6);
      }
      if (state.keys['ArrowRight'] || state.keys['KeyD']) {
        state.playerX = Math.min(450, state.playerX + 6); // Stay on left half
      }
      player.style.left = state.playerX + 'px';

      // Automatic Player Firing Control Subsystem
      if (timestamp - state.lastShotTime > state.fireRate) {
        const bullet = document.createElement('div');
        bullet.className = 'projectile';
        bullet.style.left = (state.playerX + 40) + 'px';
        bullet.style.bottom = '105px';
        game.appendChild(bullet);
        state.projectiles.push({ el: bullet, x: state.playerX + 40, y: 105 });
        state.lastShotTime = timestamp;
      }

      // Procedural Boss Flight Matrix Math Calculation
      const bossY = 230 + Math.sin(timestamp / 400) * 120;
      boss.style.top = bossY + 'px';

      // Automated Boss Weapon Layer
      const bossFireDelay = Math.max(300, 1000 - (state.level * 60));
      if (timestamp - state.lastBossShotTime > bossFireDelay) {
        const eBullet = document.createElement('div');
        eBullet.className = 'enemy-projectile';
        eBullet.style.left = (state.bossX) + 'px';
        eBullet.style.top = (bossY + 50) + 'px';
        game.appendChild(eBullet);
        
        // Dynamic angle calculation heading directly towards the player's core footprint
        const targetX = state.playerX + 20;
        const targetY = 520; // Player bottom floor offset coordinates
        const originX = state.bossX;
        const originY = bossY + 50;
        
        const angle = Math.atan2(targetY - originY, targetX - originX);
        state.bossProjectiles.push({
          el: eBullet,
          x: originX,
          y: originY,
          vx: Math.cos(angle) * 5,
          vy: Math.sin(angle) * 5
        });
        state.lastBossShotTime = timestamp;
      }

      // Update Player project structures
      for (let i = state.projectiles.length - 1; i >= 0; i--) {
        const p = state.projectiles[i];
        p.x += 8;
        p.el.style.left = p.x + 'px';

        // Collision Check: Projectile vs Boss Entity Box Bounds
        if (p.x >= state.bossX && p.x <= state.bossX + 80 && (600 - p.y) >= bossY && (600 - p.y) <= bossY + 100) {
          p.el.remove();
          state.projectiles.splice(i, 1);
          state.bossHP = Math.max(0, state.bossHP - state.damage);
          bossHPEl.textContent = state.bossHP;
          bossHealthFill.style.width = ((state.bossHP / state.bossMaxHP) * 100) + '%';

          if (state.bossHP <= 0) {
            triggerLevelComplete();
            return;
          }
          continue;
        }

        if (p.x > 1000) {
          p.el.remove();
          state.projectiles.splice(i, 1);
        }
      }

      // Update Boss tracking projectile matrices
      for (let i = state.bossProjectiles.length - 1; i >= 0; i--) {
        const p = state.bossProjectiles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.el.style.left = p.x + 'px';
        p.el.style.top = p.y + 'px';

        // Collision Check: Hazard vs Player Footprint Bounds
        if (p.x >= state.playerX && p.x <= state.playerX + 40 && p.y >= (600 - 130) && p.y <= 600) {
          p.el.remove();
          state.bossProjectiles.splice(i, 1);
          state.playerHP = Math.max(0, state.playerHP - 15);
          playerHPEl.textContent = state.playerHP;

          if (state.playerHP <= 0) {
            triggerGameOver();
            return;
          }
          continue;
        }

        if (p.x < 0 || p.x > 1000 || p.y > 600) {
          p.el.remove();
          state.bossProjectiles.splice(i, 1);
        }
      }

      state.gameLoopId = requestAnimationFrame(gameLoop);
    }
  </script>
</body>
</html>
