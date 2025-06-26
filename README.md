# Cruiser-KH
*Credit by PeeKhak
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Crash Game with Pixel Cruiser and Coins</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
            margin: 0;
            padding: 20px;
        }
        #game-container {
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            text-align: center;
            width: 640px;
        }
        #multiplier {
            font-size: 2.5em;
            color: #333;
            margin: 10px 0;
            transition: transform 0.5s ease-in-out;
        }
        #multiplier.glow {
            animation: glow 1.5s ease-in-out infinite;
        }
        @keyframes glow {
            0% { transform: scale(1); text-shadow: 0 0 5px #3498db, 0 0 10px #3498db; }
            50% { transform: scale(1.1); text-shadow: 0 0 10px #3498db, 0 0 20px #3498db; }
            100% { transform: scale(1); text-shadow: 0 0 5px #3498db, 0 0 10px #3498db; }
        }
        #status {
            font-size: 1.5em;
            color: #e74c3c;
            margin: 10px 0;
        }
        #coins {
            font-size: 1.5em;
            color: #2ecc71;
            margin: 10px 0;
        }
        #results {
            margin-top: 20px;
            text-align: left;
            max-height: 300px;
            overflow-y: auto;
            border-top: 1px solid #ccc;
            padding-top: 10px;
        }
        button {
            padding: 12px 24px;
            font-size: 1.2em;
            cursor: pointer;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
            margin: 5px;
        }
        button:disabled {
            background-color: #95a5a6;
            cursor: not-allowed;
        }
        select {
            padding: 12px;
            font-size: 1.2em;
            margin: 5px;
            border-radius: 5px;
        }
        canvas {
            border: 1px solid #333;
            image-rendering: pixelated;
            background: linear-gradient(to bottom, #87ceeb, #4682b4);
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <h1>Crash Game</h1>
        <canvas id="game-canvas" width="600" height="200"></canvas>
        <div id="multiplier">1.00x</div>
        <div id="status">Waiting...</div>
        <div id="coins">Coins: 100</div>
        <select id="bet-amount">
            <option value="5">Bet 5 Coins</option>
            <option value="10">Bet 10 Coins</option>
            <option value="20">Bet 20 Coins</option>
        </select>
        <button id="start-btn">Start Play</button>
        <button id="cashout-btn" disabled>Cash Out</button>
        <div id="results"></div>
    </div>

    <script>
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const multiplierDisplay = document.getElementById('multiplier');
        const statusDisplay = document.getElementById('status');
        const coinsDisplay = document.getElementById('coins');
        const startBtn = document.getElementById('start-btn');
        const cashoutBtn = document.getElementById('cashout-btn');
        const betSelect = document.getElementById('bet-amount');
        const resultsDisplay = document.getElementById('results');

        let currentMultiplier = 1;
        let gameRunning = false;
        let crashPoint = 1;
        let animationFrame;
        let playCount = 0;
        let totalPlays = 10;
        let results = [];
        let seaOffset = 0;
        let seaOffsetSecondary = 0;
        let shipX = 20;
        let shipY = 120;
        let isCrashed = false;
        let coins = 100;
        let currentBet = 5;
        let particles = [];
        let showLoseMessage = false;
        let loseMessageTimer = 0;
        let showWinMessage = false;
        let winMessageTimer = 0;
        let winMessageText = '';
        let countdown = 0;
        let countdownTimer = 0;
        let animationTime = 0;
        let stars = [];

        // Initialize stars
        for (let i = 0; i < 20; i++) {
            stars.push({
                x: Math.random() * canvas.width,
                y: Math.random() * 100,
                radius: Math.random() * 2 + 1,
                alpha: Math.random(),
                twinkleSpeed: Math.random() * 0.02 + 0.01
            });
        }

        // New pixel-art cruiser sprite (16x16, inspired by PNGTree cruise ship)
        const cruiserSprite = [
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
            [0,0,0,0,0,0,2,2,2,2,0,0,0,0,0,0],
            [0,0,0,0,0,2,2,2,2,2,2,0,0,0,0,0],
            [0,0,0,0,0,1,1,1,1,1,1,1,0,0,0,0],
            [0,0,0,0,1,1,1,1,1,1,1,1,1,0,0,0],
            [0,0,0,0,1,1,1,1,1,1,1,1,1,1,0,0],
            [0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0],
            [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
            [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
            [0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
            [0,0,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
            [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
        ];

        // Explosion particles
        function createExplosion(x, y) {
            for (let i = 0; i < 30; i++) {
                particles.push({
                    x: x + 48,
                    y: y + 48,
                    vx: (Math.random() - 0.5) * 6,
                    vy: (Math.random() - 0.5) * 6,
                    life: 30,
                    color: ['#ff4500', '#ffff00', '#ff0000'][Math.floor(Math.random() * 3)]
                });
            }
        }

        function updateParticles() {
            particles = particles.filter(p => p.life > 0);
            particles.forEach(p => {
                p.x += p.vx;
                p.y += p.vy;
                p.life--;
            });
        }

        function drawPixelArt(sprite, x, y, scale, crashed = false) {
            const pixelSize = scale;
            for (let i = 0; i < sprite.length; i++) {
                for (let j = 0; j < sprite[i].length; j++) {
                    if (sprite[i][j] === 1) {
                        ctx.fillStyle = crashed ? '#e74c3c' : '#ffffff'; // White body
                        ctx.fillRect(x + j * pixelSize, y + i * pixelSize, pixelSize, pixelSize);
                    } else if (sprite[i][j] === 2) {
                        ctx.fillStyle = crashed ? '#b22222' : '#00b7eb'; // Blue accents
                        ctx.fillRect(x + j * pixelSize, y + i * pixelSize, pixelSize, pixelSize);
                    }
                }
            }
        }

        function drawStars() {
            stars.forEach(star => {
                ctx.beginPath();
                ctx.arc(star.x, star.y, star.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(255, 255, 255, ${star.alpha})`;
                ctx.fill();
                star.alpha += star.twinkleSpeed;
                if (star.alpha > 1 || star.alpha < 0) star.twinkleSpeed = -star.twinkleSpeed;
            });
        }

        function drawSea() {
            ctx.fillStyle = '#4682b4';
            ctx.fillRect(0, 100, canvas.width, canvas.height - 100);
            ctx.fillStyle = '#5f9ea0';
            for (let x = 0; x < canvas.width; x += 20) {
                let waveHeight = Math.sin((x + seaOffset) * 0.025) * 10;
                ctx.fillRect(x, 100 + waveHeight, 20, 20);
            }
            ctx.fillStyle = 'rgba(95, 158, 160, 0.5)';
            for (let x = 0; x < canvas.width; x += 20) {
                let waveHeight = Math.sin((x + seaOffsetSecondary) * 0.03) * 8;
                ctx.fillRect(x, 110 + waveHeight, 20, 15);
            }
            seaOffset = (seaOffset + 1) % canvas.width;
            seaOffsetSecondary = (seaOffsetSecondary + 1.5) % canvas.width;
        }

        function drawParticles() {
            particles.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.fillRect(p.x, p.y, 8, 8);
            });
        }

        function drawCountdown() {
            if (countdown > 0) {
                ctx.font = '48px "Press Start 2P", monospace';
                ctx.fillStyle = '#ffffff';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(countdown.toString(), canvas.width / 2, canvas.height / 2);
            }
        }

        function drawLoseMessage() {
            if (showLoseMessage) {
                let scale = 1 + Math.sin(loseMessageTimer * 0.2) * 0.1;
                ctx.font = `${48 * scale}px "Press Start 2P", monospace`;
                ctx.fillStyle = `rgba(231, 76, 60, ${loseMessageTimer / 25})`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('LOSE!', canvas.width / 2, canvas.height / 2);
            }
        }

        function drawWinMessage() {
            if (showWinMessage) {
                let scale = 1 + Math.sin(winMessageTimer * 0.2) * 0.1;
                ctx.font = `${32 * scale}px "Press Start 2P", monospace`;
                ctx.fillStyle = `rgba(46, 204, 113, ${winMessageTimer / 25})`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(winMessageText, canvas.width / 2, canvas.height / 2);
            }
        }

        function render() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawStars();
            drawSea();
            if (!isCrashed && countdown === 0) {
                let bob = Math.sin(animationTime * 0.05) * 5;
                drawPixelArt(cruiserSprite, shipX, shipY + bob, 6, isCrashed);
            }
            drawParticles();
            drawLoseMessage();
            drawWinMessage();
            drawCountdown();
            animationTime++;
        }

        function getRandomCrashPoint() {
            const r = Math.random();
            return Math.max(1, -Math.log(r) * 3.5);
        }

        function updateMultiplier() {
            if (countdown > 0) {
                countdownTimer--;
                if (countdownTimer <= 0) {
                    countdown--;
                    countdownTimer = 50;
                    if (countdown === 0) {
                        cashoutBtn.disabled = false;
                        multiplierDisplay.classList.add('glow');
                    }
                }
                render();
                animationFrame = requestAnimationFrame(updateMultiplier);
                return;
            }

            if (!gameRunning) return;
            currentMultiplier += 0.01;
            multiplierDisplay.textContent = `${currentMultiplier.toFixed(2)}x`;

            shipX = 20 + (currentMultiplier - 1) * 100;
            if (isCrashed) {
                shipY += 4;
                if (shipY > 160) shipY = 160;
            }

            if (showLoseMessage) {
                loseMessageTimer--;
                if (loseMessageTimer <= 0) showLoseMessage = false;
            }

            if (showWinMessage) {
                winMessageTimer--;
                if (winMessageTimer <= 0) showWinMessage = false;
            }

            updateParticles();
            render();

            if (currentMultiplier >= crashPoint) {
                gameRunning = false;
                isCrashed = true;
                showLoseMessage = true;
                loseMessageTimer = 25;
                createExplosion(shipX, shipY);
                statusDisplay.textContent = `Crashed at ${crashPoint.toFixed(2)}x!`;
                cashoutBtn.disabled = true;
                multiplierDisplay.classList.remove('glow');
                coins -= currentBet;
                coinsDisplay.textContent = `Coins: ${coins}`;
                results.push(`Play ${playCount}: Crashed at ${crashPoint.toFixed(2)}x (-${currentBet} coins)`);
                updateResults();
                setTimeout(nextPlay, 1000);
            } else {
                animationFrame = requestAnimationFrame(updateMultiplier);
            }
        }

        function startGame() {
            if (playCount >= totalPlays) {
                statusDisplay.textContent = 'All 10 plays completed!';
                startBtn.disabled = true;
                return;
            }
            if (coins < currentBet) {
                statusDisplay.textContent = 'Not enough coins!';
                return;
            }

            playCount++;
            gameRunning = true;
            currentMultiplier = 1;
            crashPoint = getRandomCrashPoint();
            multiplierDisplay.textContent = '1.00x';
            statusDisplay.textContent = `Play ${playCount}: Starting...`;
            cashoutBtn.disabled = true;
            startBtn.disabled = true;
            shipX = 20;
            shipY = 120;
            isCrashed = false;
            particles = [];
            showLoseMessage = false;
            showWinMessage = false;
            countdown = 3;
            countdownTimer = 50;

            const autoCashout = crashPoint < 2 ? crashPoint - 0.01 : Math.min(
                2 + Math.random() * 3,
                crashPoint - 0.01
            );
            setTimeout(() => {
                if (gameRunning && currentMultiplier < crashPoint) {
                    cashOut(autoCashout);
                }
            }, (autoCashout - 1) * 1000 / 0.01 + 3000);

            animationFrame = requestAnimationFrame(updateMultiplier);
        }

        function cashOut(cashoutPoint) {
            if (!gameRunning || currentMultiplier >= crashPoint) return;
            gameRunning = false;
            cancelAnimationFrame(animationFrame);
            const winnings = Math.floor(currentBet * currentMultiplier);
            statusDisplay.textContent = `Cashed out at ${currentMultiplier.toFixed(2)}x!`;
            cashoutBtn.disabled = true;
            multiplierDisplay.classList.remove('glow');
            coins += winnings;
            coinsDisplay.textContent = `Coins: ${coins}`;
            winMessageText = `WIN! +${winnings} coins`;
            showWinMessage = true;
            winMessageTimer = 25;
            results.push(`Play ${playCount}: Cashed out at ${currentMultiplier.toFixed(2)}x (+${winnings} coins)`);
            updateResults();
            setTimeout(nextPlay, 1000);
        }

        function nextPlay() {
            if (playCount < totalPlays && coins >= currentBet) {
                startBtn.disabled = false;
                betSelect.disabled = false;
            } else {
                statusDisplay.textContent = coins < currentBet ? 'Not enough coins to continue!' : 'All 10 plays completed!';
                startBtn.disabled = true;
            }
        }

        function updateResults() {
            resultsDisplay.innerHTML = results.map(result => `<div>${result}</div>`).join('');
        }

        startBtn.addEventListener('click', () => {
            if (!gameRunning && playCount < totalPlays && coins >= currentBet) {
                currentBet = parseInt(betSelect.value);
                betSelect.disabled = true;
                startGame();
            }
        });

        cashoutBtn.addEventListener('click', () => {
            cashOut(currentMultiplier);
        });

        betSelect.addEventListener('change', () => {
            currentBet = parseInt(betSelect.value);
        });

        render();
    </script>
</body>
</html>
