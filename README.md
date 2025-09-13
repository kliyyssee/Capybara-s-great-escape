<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Alfrancis's Pet: Mark's Great Escape</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Comic+Neue:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color-top: #87ceeb; /* Sky blue */
            --bg-color-bottom: #fce4ec; /* Light pink */
            --ground-color: #a0c49d; /* Pastel green */
            --text-color: #634832; /* Chocolate brown */
            --platform-color: #d1b89a; /* Beige */
            --obstacle-color: #8b0000; /* Dark red */
            --button-bg: #ffe4e1; /* Misty rose */
            --button-border: #ffb6c1; /* Light pink */
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background: linear-gradient(to bottom, var(--bg-color-top), var(--bg-color-bottom));
            font-family: 'Comic Neue', cursive, sans-serif;
            color: var(--text-color);
            text-align: center;
            padding: 20px;
            box-sizing: border-box;
            overflow: hidden;
        }

        h1 {
            font-size: clamp(1.5rem, 5vw, 2.5rem);
            margin-bottom: 10px;
            color: var(--text-color);
            text-shadow: 2px 2px 0 #fff;
        }

        #gameContainer {
            position: relative;
            width: clamp(300px, 90vw, 800px);
            max-width: 800px;
            aspect-ratio: 16 / 9;
            background-color: #add8e6;
            border: 8px solid var(--text-color);
            border-radius: 20px;
            overflow: hidden;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
        }

        canvas {
            width: 100%;
            height: 100%;
            display: block;
        }

        #controls, #gameStatus {
            margin-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }

        #score, #message {
            font-size: clamp(1.2rem, 4vw, 2rem);
            font-weight: bold;
            color: var(--text-color);
            text-shadow: 2px 2px 0 #fff;
        }

        .button {
            padding: 10px 20px;
            font-size: clamp(1rem, 3vw, 1.5rem);
            font-weight: bold;
            color: var(--text-color);
            background-color: var(--button-bg);
            border: 2px solid var(--button-border);
            border-radius: 50px;
            box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .button:hover {
            transform: translateY(-2px);
            box-shadow: 0 7px 14px rgba(0, 0, 0, 0.2);
        }

        .button:active {
            transform: translateY(2px);
            box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
        }

        /* Mobile specific styles for controls */
        #mobileControls {
            display: none;
            flex-direction: row;
            justify-content: center;
            width: 100%;
            margin-top: 20px;
        }
        @media (max-width: 600px) {
            #mobileControls {
                display: flex;
            }
        }
    </style>
</head>
<body>

    <h1>Alfrancis's Pet: Mark's Great Escape</h1>
    <div id="gameContainer">
        <canvas id="gameCanvas"></canvas>
    </div>
    <div id="gameStatus">
        <p id="score">Score: 0</p>
        <p id="message">Press 'Start Game' to play!</p>
    </div>
    <div id="controls">
        <button id="startGameBtn" class="button">Start Game</button>
    </div>
    <div id="mobileControls">
        <button id="jumpBtn" class="button">Jump!</button>
    </div>
    
    <script>
        // --- Game Setup ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startGameBtn = document.getElementById('startGameBtn');
        const jumpBtn = document.getElementById('jumpBtn');
        const scoreDisplay = document.getElementById('score');
        const messageDisplay = document.getElementById('message');

        let gameLoopId;
        let isGameOver = true;
        let score = 0;
        let worldOffset = 0; // Camera offset for the world scrolling
        const gameSpeed = 3;

        // --- Character Object (Mark) ---
        const mark = {
            emoji: '🦫', // Mark the capybara emoji
            width: 40,
            height: 40,
            x: 50,
            y: 0,
            velocityX: 0,
            velocityY: 0,
            jumpStrength: -15,
            gravity: 0.7,
            isJumping: false,
            // A simple animation frame counter
            animationFrame: 0,
            update() {
                // Apply horizontal movement
                this.x += this.velocityX;

                // Apply gravity
                this.velocityY += this.gravity;
                this.y += this.velocityY;

                // Simple wall collision
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvas.width) this.x = canvas.width - this.width;

                this.animationFrame++;
            },
            draw() {
                ctx.font = `${this.height}px serif`;
                ctx.textAlign = 'center';
                ctx.fillText(this.emoji, this.x + this.width / 2, this.y + this.height);
            }
        };

        // --- Game Elements ---
        let platforms = [];
        let obstacles = [];
        let rewards = [];

        // Function to create game elements
        function createLevel() {
            platforms = [];
            obstacles = [];
            rewards = [];

            // Ground platform
            platforms.push({ x: 0, y: canvas.height - 40, width: canvas.width, height: 40 });

            // Generate some platforms, obstacles, and rewards
            let currentX = canvas.width + 100;
            for (let i = 0; i < 10; i++) {
                // Platforms
                let platformWidth = 100 + Math.random() * 150;
                let platformY = (canvas.height - 150) - (Math.random() * 100);
                platforms.push({
                    x: currentX,
                    y: platformY,
                    width: platformWidth,
                    height: 20
                });

                // Obstacles on some platforms
                if (Math.random() > 0.5) {
                    obstacles.push({
                        x: currentX + (platformWidth / 2) - 10,
                        y: platformY - 20,
                        width: 20,
                        height: 20,
                        type: 'spike'
                    });
                }
                
                // Rewards
                if (Math.random() > 0.3) {
                     rewards.push({
                        emoji: '🍫',
                        width: 30,
                        height: 30,
                        x: currentX + (platformWidth / 2) - 15,
                        y: platformY - 50,
                        collected: false
                     });
                }
                
                currentX += platformWidth + 100 + (Math.random() * 50);
            }
        }
        
        // --- Collision Detection ---
        function checkCollision(objA, objB) {
            return objA.x < objB.x + objB.width &&
                   objA.x + objA.width > objB.x &&
                   objA.y < objB.y + objB.height &&
                   objA.y + objA.height > objB.y;
        }

        // --- Game Logic ---
        function updateGame() {
            if (isGameOver) return;

            // Update Mark's position
            mark.update();
            
            // Check for collision with ground
            const groundLevel = canvas.height - 40;
            if (mark.y + mark.height > groundLevel) {
                mark.y = groundLevel - mark.height;
                mark.velocityY = 0;
                mark.isJumping = false;
            }

            // Check for platform collisions
            let onPlatform = false;
            platforms.forEach(platform => {
                if (checkCollision(mark, platform) && mark.velocityY >= 0) {
                    // Check if landing on top of the platform
                    if (mark.y + mark.height <= platform.y + mark.velocityY) {
                        mark.y = platform.y - mark.height;
                        mark.velocityY = 0;
                        mark.isJumping = false;
                        onPlatform = true;
                    }
                }
            });

            // If not on a platform or ground, allow gravity to continue
            if (!onPlatform) {
                 mark.isJumping = mark.y + mark.height < canvas.height - 40;
            }
            
            // Move the world
            worldOffset -= gameSpeed;

            // Move and check for collisions with obstacles
            obstacles = obstacles.filter(obstacle => {
                if (checkCollision(mark, { ...obstacle, x: obstacle.x + worldOffset })) {
                    endGame();
                    return false;
                }
                return (obstacle.x + worldOffset) + obstacle.width > 0;
            });
            
            // Move and check for rewards
            rewards = rewards.filter(reward => {
                if (!reward.collected && checkCollision(mark, { ...reward, x: reward.x + worldOffset })) {
                    score += 10;
                    scoreDisplay.textContent = `Score: ${score}`;
                    reward.collected = true;
                    return false;
                }
                return (reward.x + worldOffset) + reward.width > 0;
            });
        }

        // --- Drawing Functions ---
        function drawGame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw platforms
            ctx.fillStyle = var(--platform-color);
            platforms.forEach(platform => {
                ctx.fillRect(platform.x + worldOffset, platform.y, platform.width, platform.height);
            });

            // Draw obstacles
            ctx.fillStyle = var(--obstacle-color);
            obstacles.forEach(obstacle => {
                if (obstacle.type === 'spike') {
                    ctx.fillStyle = var(--obstacle-color);
                    ctx.beginPath();
                    ctx.moveTo(obstacle.x + worldOffset, obstacle.y + obstacle.height);
                    ctx.lineTo(obstacle.x + obstacle.width / 2 + worldOffset, obstacle.y);
                    ctx.lineTo(obstacle.x + obstacle.width + worldOffset, obstacle.y + obstacle.height);
                    ctx.closePath();
                    ctx.fill();
                }
            });

            // Draw rewards
            rewards.forEach(reward => {
                if (!reward.collected) {
                    ctx.font = `${reward.width}px serif`;
                    ctx.textAlign = 'center';
                    ctx.fillText(reward.emoji, reward.x + worldOffset + reward.width / 2, reward.y + reward.height);
                }
            });

            // Draw Mark
            mark.draw();
        }
        
        // --- Game Loop and Control ---
        function gameLoop() {
            updateGame();
            drawGame();
            if (!isGameOver) {
                gameLoopId = requestAnimationFrame(gameLoop);
            }
        }

        function startGame() {
            isGameOver = false;
            score = 0;
            scoreDisplay.textContent = 'Score: 0';
            messageDisplay.textContent = 'Run and jump to collect chocolates!';
            
            // Reset player position and world offset
            mark.x = 50;
            mark.y = canvas.height - 40 - mark.height;
            mark.velocityY = 0;
            mark.isJumping = false;
            worldOffset = 0;
            
            createLevel();

            // Start game loop
            cancelAnimationFrame(gameLoopId);
            gameLoop();
            
            // Hide start button and show jump button if on mobile
            startGameBtn.style.display = 'none';
        }

        function endGame() {
            isGameOver = true;
            cancelAnimationFrame(gameLoopId);
            messageDisplay.textContent = `Game Over! You collected ${score} chocolate(s)!`;
            startGameBtn.textContent = 'Play Again';
            startGameBtn.style.display = 'block';
        }

        function handleJump() {
            if (!mark.isJumping && !isGameOver) {
                mark.isJumping = true;
                mark.velocityY = mark.jumpStrength;
            } else if (isGameOver) {
                startGame();
            }
        }

        // --- Event Listeners and Initial Setup ---
        window.addEventListener('load', () => {
            function resizeCanvas() {
                const container = document.getElementById('gameContainer');
                canvas.width = container.clientWidth;
                canvas.height = container.clientHeight;
                mark.y = canvas.height - 40 - mark.height; // Reset Mark's position on resize
            }
            resizeCanvas();
            window.addEventListener('resize', resizeCanvas);
            
            // Desktop keyboard controls
            document.addEventListener('keydown', (e) => {
                if (e.code === 'Space' || e.code === 'ArrowUp') {
                    handleJump();
                } else if (e.code === 'ArrowRight') {
                    mark.velocityX = 5;
                } else if (e.code === 'ArrowLeft') {
                    mark.velocityX = -5;
                }
            });
            document.addEventListener('keyup', (e) => {
                if (e.code === 'ArrowRight' || e.code === 'ArrowLeft') {
                    mark.velocityX = 0;
                }
            });

            
            // Mobile button controls
            jumpBtn.addEventListener('click', handleJump);
            startGameBtn.addEventListener('click', startGame);
        });
        
    </script>
</body>
</html>
# Capybara-s-great-escape
