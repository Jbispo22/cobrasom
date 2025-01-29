<!doctype html>
<style>
    /* Estilo fixo para o ícone do jogo */
    #snakeGameIcon {
        position: fixed;
        top: 200px;
        left: 200px;
        z-index: 9999;
        background-color: rgba(0, 0, 0, 0.7);
        padding: 10px;
        border-radius: 8px;
        box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
        cursor: pointer;
    }

    /* Estilo fixo para o ícone de reset */
    #resetGameIcon {
        position: fixed;
        top: 240px;
        left: 200px;
        z-index: 9999;
        background-color: rgba(0, 0, 0, 0.7);
        padding: 10px;
        border-radius: 8px;
        box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
        cursor: pointer;
    }

    /* Estilo do container do jogo */
    #game-container {
        display: none;
        position: fixed;
        bottom: 10px;
        right: 50px;
        z-index: 9998;
        background-color: rgba(0, 0, 0, 0.7);
        padding: 30px;
        border-radius: 8px;
        box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
        transition: all 0.3s ease;
    }

    canvas {
        border: 2px solid #fff;
        width: 250px;
        height: 250px;
    }

    .score {
        position: absolute;
        top: 10px;
        left: 10px;
        color: white;
        font-size: 1.2rem;
    }

    /* Estilo do botão de "ESC" */
    .esc-button {
        padding: 10px 20px;
        font-size: 1.2rem;
        cursor: pointer;
        background-color: #f44336;
        border: none;
        color: white;
        border-radius: 5px;
        transition: 0.3s;
    }

    .esc-button:hover {
        background-color: #e53935;
    }

    /* Estilo da janela do jogo com efeito neon */
    #game-container {
        box-shadow: 0 0 20px 10px rgba(0, 255, 255, 0.7);
    }

    /* Controle de volume */
    .volume-control {
        margin-top: 10px;
        color: white;
    }

    .volume-control label {
        font-size: 1.2rem;
        margin-right: 10px;
    }

    .volume-control input {
        width: 100px;
    }
</style>

<!-- Ícone do Jogo -->
<div id="snakeGameIcon" onclick="toggleGame()">🐍</div>

<!-- Ícone de Reset -->
<div id="resetGameIcon" onclick="resetGame()">🔄</div>

<!-- Container do Jogo -->
<div id="game-container">
    <div class="score">Score: 0 | Tempo: 0s</div>
    <canvas id="gameCanvas"></canvas>
    <button class="esc-button" onclick="toggleGame()">ESC</button>

    <!-- Controle de volume -->
    <div class="volume-control">
        <label for="volume">Volume:</label>
        <input type="range" id="volume" min="0" max="1" step="0.01" value="1" onchange="adjustVolume()" />
    </div>
</div>
<script>
    let gameActive = false;
    let score = 0;
    let time = 0;
    let gameLoopInterval;
    let snake;
    let food;
    let dx;
    let dy;
    let timerInterval;
    let gameSpeed = 300; // Velocidade inicial do jogo
    let accelerationFactor = 1.02; // Fator de aceleração progressiva
    let lastTime = 0; // Tempo do último aumento de velocidade
    let highScore = 0; // Inicializando o recorde com 0, em vez de buscar no localStorage

    const animals = ["🐭", "🐸", "🐦", "🦎", "🐁"];
    const animalScores = {
        "🐭": 6,
        "🐸": 8,
        "🐦": 10,
        "🦎": 8,
        "🐁": 4
    };

    const eatingSound = new Audio('eat-sound.mp3'); // Adicione um arquivo de som 'eat-sound.mp3'

    function toggleGame() {
        const gameContainer = document.getElementById("game-container");
        if (gameContainer.style.display === "block") {
            gameContainer.style.display = "none";
            stopGameLoop();
            stopTimer();
        } else {
            gameContainer.style.display = "block";
            startGame();
        }
    }

    function startGame() {
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const gridSize = 10;
        snake = [{ x: 120, y: 120 }];
        food = randomPosition();
        dx = gridSize;
        dy = 0;
        score = 0; // Garantindo que o score inicie com 0
        time = 0;
        updateScore();
        updateTime();

        document.addEventListener("keydown", changeDirection);
        canvas.addEventListener("touchstart", handleTouchStart, false);

        canvas.width = 250;
        canvas.height = 250;

        startGameLoop();
        startTimer();

        function startGameLoop() {
            gameLoopInterval = setInterval(() => {
                if (gameOver()) return;
                clearCanvas();
                moveSnake();
                drawSnake();
                drawFood();
                updateScore();
                checkAcceleration();
            }, gameSpeed);
        }

        function stopGameLoop() {
            clearInterval(gameLoopInterval);
        }

        function clearCanvas() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }

        function moveSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            snake.unshift(head);

            if (checkCollision(head, food)) {
                score += animalScores[food.animal];
                food = randomPosition();
                updateScore();
                eatingSound.play(); // Reproduz o som ao comer
            } else {
                snake.pop();
            }
        }

        function drawSnake() {
            ctx.fillStyle = "#00FF00";
            for (const segment of snake) {
                ctx.fillRect(segment.x, segment.y, gridSize, gridSize);
            }
        }

        function drawFood() {
            ctx.font = "20px sans-serif";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(food.animal, food.x + gridSize / 2, food.y + gridSize / 2);
        }

        function randomPosition() {
            let animal = animals[Math.floor(Math.random() * animals.length)];
            const x = Math.floor(Math.random() * (canvas.width / gridSize)) * gridSize;
            const y = Math.floor(Math.random() * (canvas.height / gridSize)) * gridSize;

            return {
                x: x,
                y: y,
                animal: animal,
            };
        }

        function checkCollision(snakeHead, food) {
            const foodArea = {
                x: food.x,
                y: food.y,
                width: gridSize,
                height: gridSize
            };

            return snakeHead.x >= foodArea.x && snakeHead.x < foodArea.x + foodArea.width &&
                snakeHead.y >= foodArea.y && snakeHead.y < foodArea.y + foodArea.height;
        }

        function changeDirection(event) {
            switch (event.keyCode) {
                case 37: // Esquerda
                    if (dx === 0) {
                        dx = -gridSize;
                        dy = 0;
                    }
                    break;
                case 38: // Cima
                    if (dy === 0) {
                        dx = 0;
                        dy = -gridSize;
                    }
                    break;
                case 39: // Direita
                    if (dx === 0) {
                        dx = gridSize;
                        dy = 0;
                    }
                    break;
                case 40: // Baixo
                    if (dy === 0) {
                        dx = 0;
                        dy = gridSize;
                    }
                    break;
            }

            event.preventDefault();
        }

        function handleTouchStart(evt) {
            const touchX = evt.touches[0].clientX - canvas.getBoundingClientRect().left;
            const touchY = evt.touches[0].clientY - canvas.getBoundingClientRect().top;
            const head = snake[0];
            const newX = Math.floor(touchX / gridSize) * gridSize;
            const newY = Math.floor(touchY / gridSize) * gridSize;

            if (newX === head.x && newY === head.y) {
                return;
            }

            if (newX > head.x && dx === 0) {
                dx = gridSize;
                dy = 0;
            } else if (newX < head.x && dx === 0) {
                dx = -gridSize;
                dy = 0;
            } else if (newY > head.y && dy === 0) {
                dy = gridSize;
                dx = 0;
            } else if (newY < head.y && dy === 0) {
                dy = -gridSize;
                dx = 0;
            }

            evt.preventDefault();
        }

        function checkAcceleration() {
            let now = Date.now();
            if (now - lastTime > 5000) { // Aumenta a velocidade a cada 5 segundos
                gameSpeed /= accelerationFactor;
                lastTime = now;
            }
        }

        function gameOver() {
            const head = snake[0];
            if (head.x < 0 || head.x >= canvas.width || head.y < 0 || head.y >= canvas.height) {
                resetGame();
                return true;
            }

            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    resetGame();
                    return true;
                }
            }
            return false;
        }

        function resetGame() {
            clearInterval(gameLoopInterval);
            snake = [{ x: 120, y: 120 }];
            food = randomPosition();
            dx = gridSize;
            dy = 0;
            score = 0; // Reseta o score para 0
            time = 0;
            updateScore();
            updateTime();
            if (score > highScore) {
                highScore = score; // Atualiza o recorde
            }
            startGame();
        }

        function updateScore() {
            const scoreElement = document.querySelector(".score");
            scoreElement.innerText = `Score: ${score} | Tempo: ${time}s | Recorde: ${highScore}`;
        }

        function updateTime() {
            if (timerInterval) clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                time++;
                updateScore();
            }, 1000);
        }
    }

    function adjustVolume() {
        const volumeSlider = document.getElementById("volume");
        eatingSound.volume = volumeSlider.value;
    }
</script>
