<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Space Defender - Galaga Style Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: white;
            overflow: hidden;
        }
        
        .game-container {
            position: relative;
            width: 800px;
            height: 600px;
            margin: 20px auto;
            border: 3px solid #4a47e0;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 0 20px rgba(74, 71, 224, 0.5);
        }
        
        #gameCanvas {
            background: #000;
            display: block;
        }
        
        .game-ui {
            display: flex;
            justify-content: space-between;
            width: 800px;
            margin-bottom: 15px;
            padding: 10px 20px;
            background: rgba(0, 0, 0, 0.6);
            border-radius: 8px;
            border: 2px solid #4a47e0;
        }
        
        .score-container, .lives-container {
            font-size: 1.5rem;
            font-weight: bold;
            text-shadow: 0 0 5px #00ffff;
        }
        
        .controls {
            margin-top: 15px;
            text-align: center;
            background: rgba(0, 0, 0, 0.6);
            padding: 15px;
            border-radius: 8px;
            border: 2px solid #4a47e0;
            width: 800px;
        }
        
        .controls p {
            margin: 5px 0;
            font-size: 1.1rem;
        }
        
        .game-title {
            font-size: 3rem;
            margin-bottom: 10px;
            text-align: center;
            text-shadow: 0 0 10px #4a47e0, 0 0 20px #4a47e0;
            color: #fff;
            letter-spacing: 2px;
        }
        
        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(0, 0, 0, 0.85);
            z-index: 10;
        }
        
        .btn {
            background: linear-gradient(to bottom, #4a47e0, #302b63);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 1.2rem;
            border-radius: 30px;
            cursor: pointer;
            margin-top: 20px;
            transition: all 0.3s;
            box-shadow: 0 0 15px rgba(74, 71, 224, 0.7);
        }
        
        .btn:hover {
            transform: scale(1.05);
            box-shadow: 0 0 20px rgba(74, 71, 224, 0.9);
        }
        
        .hidden {
            display: none;
        }
        
        .instructions {
            max-width: 700px;
            text-align: center;
            margin-bottom: 20px;
            line-height: 1.6;
        }
        
        .game-over {
            font-size: 4rem;
            color: #ff4d4d;
            text-shadow: 0 0 10px #ff0000;
            margin-bottom: 20px;
        }
        
        .final-score {
            font-size: 2rem;
            margin-bottom: 20px;
        }
        
        .stars {
            position: absolute;
            width: 100%;
            height: 100%;
            z-index: -1;
        }
        
        .star {
            position: absolute;
            background: white;
            border-radius: 50%;
        }
    </style>
</head>
<body>
    <h1 class="game-title">SPACE DEFENDER</h1>
    
    <div class="game-ui">
        <div class="score-container">SCORE: <span id="score">0</span></div>
        <div class="lives-container">LIVES: <span id="lives">3</span></div>
    </div>
    
    <div class="game-container">
        <div class="stars" id="stars"></div>
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        
        <div id="startScreen" class="screen">
            <h2>SPACE DEFENDER</h2>
            <div class="instructions">
                <p>Defend Earth from alien invaders!</p>
                <p>Use ← → keys to move and SPACEBAR to shoot.</p>
                <p>Destroy enemy formations and avoid their attacks.</p>
                <p>Survive as long as possible and achieve the highest score!</p>
            </div>
            <button id="startBtn" class="btn">START MISSION</button>
        </div>
        
        <div id="gameOverScreen" class="screen hidden">
            <h2 class="game-over">MISSION FAILED</h2>
            <p class="final-score">Final Score: <span id="finalScore">0</span></p>
            <button id="restartBtn" class="btn">TRY AGAIN</button>
        </div>
    </div>
    
    <div class="controls">
        <p>CONTROLS: ← → Arrow Keys to Move | SPACEBAR to Shoot</p>
        <p>Destroy enemies and avoid their attacks to score points!</p>
    </div>

    <script>
        // Game variables
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');
        const scoreElement = document.getElementById('score');
        const finalScoreElement = document.getElementById('finalScore');
        const livesElement = document.getElementById('lives');
        const starsContainer = document.getElementById('stars');
        
        let gameActive = false;
        let score = 0;
        let lives = 3;
        
        // Create background stars
        function createStars() {
            starsContainer.innerHTML = '';
            for (let i = 0; i < 100; i++) {
                const star = document.createElement('div');
                star.classList.add('star');
                star.style.width = Math.random() * 2 + 'px';
                star.style.height = star.style.width;
                star.style.left = Math.random() * 100 + '%';
                star.style.top = Math.random() * 100 + '%';
                star.style.opacity = Math.random() * 0.7 + 0.3;
                starsContainer.appendChild(star);
            }
        }
        
        // Player object
        const player = {
            x: canvas.width / 2 - 25,
            y: canvas.height - 80,
            width: 50,
            height: 60,
            speed: 7,
            color: '#4a47e0',
            draw: function() {
                // Draw player ship
                ctx.fillStyle = this.color;
                
                // Ship body
                ctx.beginPath();
                ctx.moveTo(this.x + this.width / 2, this.y);
                ctx.lineTo(this.x + this.width, this.y + this.height - 20);
                ctx.lineTo(this.x, this.y + this.height - 20);
                ctx.closePath();
                ctx.fill();
                
                // Ship wings
                ctx.fillRect(this.x - 10, this.y + this.height - 40, 70, 10);
                
                // Ship details
                ctx.fillStyle = '#00ffff';
                ctx.fillRect(this.x + this.width / 2 - 5, this.y + 10, 10, 20);
                
                // Engine glow
                const gradient = ctx.createRadialGradient(
                    this.x + this.width / 2, this.y + this.height, 0,
                    this.x + this.width / 2, this.y + this.height, 20
                );
                gradient.addColorStop(0, '#00ffff');
                gradient.addColorStop(1, 'transparent');
                
                ctx.fillStyle = gradient;
                ctx.fillRect(this.x + this.width / 2 - 15, this.y + this.height, 30, 25);
            },
            move: function(direction) {
                if (direction === 'left' && this.x > 0) {
                    this.x -= this.speed;
                }
                if (direction === 'right' && this.x + this.width < canvas.width) {
                    this.x += this.speed;
                }
            }
        };
        
        // Bullets array
        let bullets = [];
        
        // Enemy array
        let enemies = [];
        
        // Enemy bullets array
        let enemyBullets = [];
        
        // Key state tracking
        const keys = {
            ArrowLeft: false,
            ArrowRight: false,
            Space: false
        };
        
        // Event listeners
        window.addEventListener('keydown', (e) => {
            if (e.code in keys) {
                keys[e.code] = true;
                e.preventDefault();
            }
        });
        
        window.addEventListener('keyup', (e) => {
            if (e.code in keys) {
                keys[e.code] = false;
                
                // Fire bullet when spacebar is released
                if (e.code === 'Space' && gameActive) {
                    fireBullet();
                }
            }
        });
        
        // Fire a bullet
        function fireBullet() {
            bullets.push({
                x: player.x + player.width / 2 - 2,
                y: player.y,
                width: 4,
                height: 15,
                speed: 10,
                color: '#00ffff'
            });
        }
        
        // Create enemies
        function createEnemies() {
            enemies = [];
            const rows = 4;
            const cols = 8;
            const enemyWidth = 40;
            const enemyHeight = 40;
            const padding = 20;
            const offsetX = (canvas.width - (cols * (enemyWidth + padding))) / 2;
            const offsetY = 80;
            
            for (let row = 0; row < rows; row++) {
                for (let col = 0; col < cols; col++) {
                    enemies.push({
                        x: offsetX + col * (enemyWidth + padding),
                        y: offsetY + row * (enemyHeight + padding),
                        width: enemyWidth,
                        height: enemyHeight,
                        speed: 1,
                        direction: 1,
                        color: row === 0 ? '#ff4d4d' : row === 1 ? '#ff9933' : row === 2 ? '#ffff33' : '#33ff33',
                        alive: true
                    });
                }
            }
        }
        
        // Enemy firing
        function enemyFire() {
            if (!gameActive || enemies.length === 0) return;
            
            // Find the lowest enemies in each column for more strategic firing
            const columns = {};
            enemies.forEach(enemy => {
                if (enemy.alive) {
                    const col = Math.floor(enemy.x / 50);
                    if (!columns[col] || enemy.y > columns[col].y) {
                        columns[col] = enemy;
                    }
                }
            });
            
            // Randomly select an enemy to fire
            const columnKeys = Object.keys(columns);
            if (columnKeys.length > 0) {
                const randomCol = columnKeys[Math.floor(Math.random() * columnKeys.length)];
                const shooter = columns[randomCol];
                
                enemyBullets.push({
                    x: shooter.x + shooter.width / 2 - 2,
                    y: shooter.y + shooter.height,
                    width: 4,
                    height: 15,
                    speed: 5,
                    color: '#ff4d4d'
                });
            }
        }
        
        // Check collisions
        function checkCollisions() {
            // Player bullets vs enemies
            bullets.forEach((bullet, bulletIndex) => {
                enemies.forEach((enemy, enemyIndex) => {
                    if (enemy.alive &&
                        bullet.x < enemy.x + enemy.width &&
                        bullet.x + bullet.width > enemy.x &&
                        bullet.y < enemy.y + enemy.height &&
                        bullet.y + bullet.height > enemy.y) {
                        
                        // Collision detected
                        bullets.splice(bulletIndex, 1);
                        enemy.alive = false;
                        score += 100;
                        scoreElement.textContent = score;
                    }
                });
            });
            
            // Enemy bullets vs player
            enemyBullets.forEach((bullet, bulletIndex) => {
                if (bullet.x < player.x + player.width &&
                    bullet.x + bullet.width > player.x &&
                    bullet.y < player.y + player.height &&
                    bullet.y + bullet.height > player.y) {
                    
                    // Player hit
                    enemyBullets.splice(bulletIndex, 1);
                    lives--;
                    livesElement.textContent = lives;
                    
                    if (lives <= 0) {
                        gameOver();
                    }
                }
            });
            
            // Enemies vs player (game over condition)
            enemies.forEach(enemy => {
                if (enemy.alive &&
                    enemy.x < player.x + player.width &&
                    enemy.x + enemy.width > player.x &&
                    enemy.y < player.y + player.height &&
                    enemy.y + enemy.height > player.y) {
                    
                    gameOver();
                }
            });
        }
        
        // Update game state
        function update() {
            if (!gameActive) return;
            
            // Move player
            if (keys.ArrowLeft) player.move('left');
            if (keys.ArrowRight) player.move('right');
            
            // Move bullets
            bullets.forEach((bullet, index) => {
                bullet.y -= bullet.speed;
                if (bullet.y < 0) {
                    bullets.splice(index, 1);
                }
            });
            
            // Move enemy bullets
            enemyBullets.forEach((bullet, index) => {
                bullet.y += bullet.speed;
                if (bullet.y > canvas.height) {
                    enemyBullets.splice(index, 1);
                }
            });
            
            // Move enemies
            let changeDirection = false;
            enemies.forEach(enemy => {
                if (!enemy.alive) return;
                
                enemy.x += enemy.speed * enemy.direction;
                
                if (enemy.x <= 0 || enemy.x + enemy.width >= canvas.width) {
                    changeDirection = true;
                }
            });
            
            // Change enemy direction if needed
            if (changeDirection) {
                enemies.forEach(enemy => {
                    if (enemy.alive) {
                        enemy.direction *= -1;
                        enemy.y += 20;
                    }
                });
            }
            
            // Random enemy firing
            if (Math.random() < 0.02) {
                enemyFire();
            }
            
            // Check for collisions
            checkCollisions();
            
            // Check if all enemies are destroyed
            const aliveEnemies = enemies.filter(enemy => enemy.alive);
            if (aliveEnemies.length === 0) {
                createEnemies();
                score += 500; // Bonus for clearing a wave
                scoreElement.textContent = score;
            }
            
            // Check if enemies reached the bottom
            enemies.forEach(enemy => {
                if (enemy.alive && enemy.y + enemy.height >= canvas.height - 50) {
                    gameOver();
                }
            });
        }
        
        // Draw everything
        function draw() {
            // Clear canvas
            ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw player
            player.draw();
            
            // Draw bullets
            bullets.forEach(bullet => {
                ctx.fillStyle = bullet.color;
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
                
                // Add glow effect
                ctx.shadowColor = bullet.color;
                ctx.shadowBlur = 10;
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
                ctx.shadowBlur = 0;
            });
            
            // Draw enemy bullets
            enemyBullets.forEach(bullet => {
                ctx.fillStyle = bullet.color;
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
                
                // Add glow effect
                ctx.shadowColor = bullet.color;
                ctx.shadowBlur = 10;
                ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
                ctx.shadowBlur = 0;
            });
            
            // Draw enemies
            enemies.forEach(enemy => {
                if (!enemy.alive) return;
                
                ctx.fillStyle = enemy.color;
                
                // Enemy body
                ctx.beginPath();
                ctx.moveTo(enemy.x, enemy.y);
                ctx.lineTo(enemy.x + enemy.width, enemy.y);
                ctx.lineTo(enemy.x + enemy.width - 10, enemy.y + enemy.height);
                ctx.lineTo(enemy.x + 10, enemy.y + enemy.height);
                ctx.closePath();
                ctx.fill();
                
                // Enemy details
                ctx.fillStyle = '#000';
                ctx.fillRect(enemy.x + 5, enemy.y + 5, 10, 10);
                ctx.fillRect(enemy.x + enemy.width - 15, enemy.y + 5, 10, 10);
                ctx.fillRect(enemy.x + 15, enemy.y + 20, 10, 5);
                ctx.fillRect(enemy.x + enemy.width - 25, enemy.y + 20, 10, 5);
            });
        }
        
        // Game loop
        function gameLoop() {
            update();
            draw();
            if (gameActive) {
                requestAnimationFrame(gameLoop);
            }
        }
        
        // Start game
        function startGame() {
            gameActive = true;
            score = 0;
            lives = 3;
            bullets = [];
            enemyBullets = [];
            
            scoreElement.textContent = score;
            livesElement.textContent = lives;
            
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            
            createEnemies();
            gameLoop();
        }
        
        // Game over
        function gameOver() {
            gameActive = false;
            finalScoreElement.textContent = score;
            gameOverScreen.classList.remove('hidden');
        }
        
        // Event listeners for buttons
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);
        
        // Initialize stars
        createStars();
    </script>
</body>
</html>
