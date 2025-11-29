<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>扫雷游戏</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            background: #000;
            color: #fff;
            padding: 10px;
            min-height: 100vh;
        }
        .container {
            max-width: 100%;
            margin: 0 auto;
        }
        .game-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 5px;
            padding: 8px 12px;
            background: #c0c0c0;
            border: 3px outset #c0c0c0;
            border-radius: 4px;
        }
        .mines-counter, .timer {
            font-family: 'Digital', monospace;
            background: #000;
            color: #f00;
            padding: 6px 10px;
            border-radius: 3px;
            font-size: 16px;
            font-weight: bold;
            min-width: 70px;
            text-align: center;
        }
        .smiley-btn {
            width: 40px;
            height: 40px;
            font-size: 18px;
            border: 2px outset #c0c0c0;
            border-radius: 4px;
            background: #c0c0c0;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .game-area {
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .field {
            display: inline-grid;
            gap: 1px;
            background: #bdbdbd;
            padding: 8px;
            border: 3px outset #bdbdbd;
            border-radius: 2px;
            margin: 0 auto;
        }
        .cell {
            width: 24px;
            height: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #bdbdbd;
            border: 2px outset #bdbdbd;
            font-weight: bold;
            cursor: pointer;
            user-select: none;
            font-size: 14px;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .cell:active {
            border: 1px solid #bdbdbd;
        }
        .cell.revealed {
            border: 1px solid #bdbdbd;
            background: #e0e0e0;
        }
        .cell.mine.revealed {
            background: #ff4444;
            border: 1px solid #ff4444;
        }
        .cell.flag::before {
            content: "旗";
            color: #f00;
        }
        .cell.question::before {
            content: "?";
            color: #000;
        }
        .cell.number-1 { color: #0000ff; }
        .cell.number-2 { color: #008000; }
        .cell.number-3 { color: #ff0000; }
        .cell.number-4 { color: #000080; }
        .cell.number-5 { color: #800000; }
        .cell.number-6 { color: #008080; }
        .cell.number-7 { color: #000000; }
        .cell.number-8 { color: #808080; }
        
        .game-controls {
            margin-top: 15px;
            padding: 12px;
            background: #333;
            border-radius: 8px;
            width: 100%;
            max-width: 400px;
        }
        .control-group {
            margin: 8px 0;
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .control-group label {
            color: #ccc;
            font-size: 14px;
            min-width: 80px;
        }
        .control-group input {
            background: #222;
            color: #fff;
            border: 1px solid #555;
            border-radius: 4px;
            padding: 6px;
            width: 60px;
            text-align: center;
        }
        .mode-buttons {
            display: flex;
            gap: 8px;
            margin: 8px 0;
        }
        .mode-btn {
            flex: 1;
            padding: 8px;
            background: #555;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 12px;
        }
        .mode-btn.active {
            background: #007bff;
        }
        .action-buttons {
            display: flex;
            gap: 8px;
            margin: 10px 0;
        }
        .action-btn {
            flex: 1;
            padding: 8px;
            background: #444;
            color: #fff;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        .seed-input {
            background: #222;
            color: #fff;
            border: 1px solid #555;
            border-radius: 4px;
            padding: 6px;
            width: 100%;
            margin-top: 5px;
        }
        .game-info {
            margin-top: 10px;
            padding: 8px;
            background: #333;
            border-radius: 4px;
            font-size: 12px;
            color: #ccc;
        }
        .seed-info {
            font-family: monospace;
            background: #222;
            padding: 4px;
            border-radius: 3px;
            word-break: break-all;
            margin-top: 4px;
            font-size: 11px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="game-header">
            <div class="mines-counter">
                雷: <span id="remainingMines">10</span>
            </div>
            <button class="smiley-btn" onclick="startNewGame()">阿!</button>
            <div class="timer">
                时: <span id="gameTimer">0</span>
            </div>
        </div>

        <div class="game-area">
            <div id="gameField"></div>
        </div>

        <div class="game-controls">
            <div class="control-group">
                <label>宽度:</label>
                <input type="number" id="width" value="9" min="5" max="15">
                
                <label>高度:</label>
                <input type="number" id="height" value="9" min="5" max="15">
            </div>
            
            <div class="control-group">
                <label>地雷数:</label>
                <input type="number" id="mines" value="10" min="1" max="30">
            </div>

            <div class="mode-buttons">
                <button class="mode-btn active" onclick="setMode('reveal')">点击模式</button>
                <button class="mode-btn" onclick="setMode('flag')">标记模式</button>
            </div>

            <div class="action-buttons">
                <button class="action-btn" onclick="startNewGame()">新游戏</button>
                <button class="action-btn" onclick="generateWithSeed()">种子游戏</button>
            </div>

            <input type="text" id="customSeed" class="seed-input" placeholder="输入种子号">

            <div class="game-info">
                <div>已开: <span id="revealedCells">0</span>/<span id="totalCells">81</span></div>
                <div>种子: <span class="seed-info" id="currentSeed">-</span></div>
            </div>
        </div>
    </div>

    <script>
        // 这里 вставляем весь JavaScript код из предыдущей версии
        // (полный код игры без изменений в логике)
        class MinesweeperGame {
            constructor(width = 9, height = 9, minesCount = 10) {
                this.width = width;
                this.height = height;
                this.minesCount = minesCount;
                this.seed = null;
                this.gameState = 'playing';
                this.revealedCount = 0;
                this.flaggedPositions = new Set();
                this.questionPositions = new Set();
                this.firstClick = true;
                this.startTime = null;
                this.timer = null;
                this.currentMode = 'reveal';
                this.cellStates = {};
            }
            
            generateGame(seed = null) {
                if (seed) {
                    this.seed = seed;
                } else {
                    this.seed = this.generateUniqueSeed();
                }
                
                this.setSeed(this.seed);
                this.gameState = 'playing';
                this.revealedCount = 0;
                this.flaggedPositions.clear();
                this.questionPositions.clear();
                this.firstClick = true;
                this.cellStates = {};
                this.stopTimer();
                this.startTime = null;
                
                const field = Array(this.height).fill().map(() => 
                    Array(this.width).fill(0)
                );
                
                return {
                    field: field,
                    seed: this.seed,
                    width: this.width,
                    height: this.height
                };
            }
            
            generateUniqueSeed() {
                return Date.now() + Math.floor(Math.random() * 1000000);
            }
            
            setSeed(seed) {
                let x = Math.sin(seed) * 10000;
                this.random = () => {
                    x = Math.sin(x * 10000) * 10000;
                    return x - Math.floor(x);
                };
            }
            
            placeMines(firstX, firstY, field) {
                const minesPositions = [];
                const totalCells = this.width * this.height;
                
                if (this.minesCount >= totalCells) {
                    this.minesCount = totalCells - 1;
                }
                
                while (minesPositions.length < this.minesCount) {
                    const x = Math.floor(this.random() * this.width);
                    const y = Math.floor(this.random() * this.height);
                    
                    if ((x === firstX && y === firstY) || 
                        Math.abs(x - firstX) <= 1 && Math.abs(y - firstY) <= 1) {
                        continue;
                    }
                    
                    const position = `${x},${y}`;
                    if (!minesPositions.includes(position)) {
                        minesPositions.push(position);
                        field[y][x] = -1;
                    }
                }
                
                this.updateNumbers(field, minesPositions);
                return minesPositions;
            }
            
            updateNumbers(field, minesPositions) {
                minesPositions.forEach(pos => {
                    const [x, y] = pos.split(',').map(Number);
                    
                    for (let dx = -1; dx <= 1; dx++) {
                        for (let dy = -1; dy <= 1; dy++) {
                            if (dx === 0 && dy === 0) continue;
                            
                            const nx = x + dx;
                            const ny = y + dy;
                            
                            if (nx >= 0 && nx < this.width && 
                                ny >= 0 && ny < this.height && 
                                field[ny][nx] !== -1) {
                                field[ny][nx]++;
                            }
                        }
                    }
                });
            }
            
            revealCell(x, y, field) {
                if (this.gameState !== 'playing') return false;
                if (this.flaggedPositions.has(`${x},${y}`)) return false;
                
                if (this.firstClick) {
                    this.placeMines(x, y, field);
                    this.firstClick = false;
                    this.startTimer();
                }
                
                if (field[y][x] === -1) {
                    this.gameState = 'gameover';
                    this.stopTimer();
                    return true;
                }
                
                if (!this.cellStates[`${x},${y}`]) {
                    this.cellStates[`${x},${y}`] = 'revealed';
                    this.revealedCount++;
                    
                    if (field[y][x] === 0) {
                        this.revealEmptyArea(x, y, field);
                    }
                }
                
                this.checkWinCondition(field);
                return true;
            }
            
            revealEmptyArea(x, y, field) {
                const queue = [[x, y]];
                const visited = new Set();
                
                while (queue.length > 0) {
                    const [cx, cy] = queue.shift();
                    const key = `${cx},${cy}`;
                    
                    if (visited.has(key)) continue;
                    if (this.flaggedPositions.has(key)) continue;
                    
                    visited.add(key);
                    
                    if (!this.cellStates[key]) {
                        this.cellStates[key] = 'revealed';
                        this.revealedCount++;
                    }
                    
                    if (field[cy][cx] === 0) {
                        for (let dx = -1; dx <= 1; dx++) {
                            for (let dy = -1; dy <= 1; dy++) {
                                const nx = cx + dx;
                                const ny = cy + dy;
                                
                                if (nx >= 0 && nx < this.width && 
                                    ny >= 0 && ny < this.height && 
                                    !visited.has(`${nx},${ny}`)) {
                                    queue.push([nx, ny]);
                                }
                            }
                        }
                    }
                }
            }
            
            handleCellClick(x, y) {
                if (this.gameState !== 'playing') return;
                
                const key = `${x},${y}`;
                
                if (this.currentMode === 'reveal') {
                    if (!this.flaggedPositions.has(key) && !this.questionPositions.has(key)) {
                        this.revealCell(x, y, gameData.field);
                    }
                } else if (this.currentMode === 'flag') {
                    this.cycleCellMark(x, y);
                }
            }
            
            cycleCellMark(x, y) {
                const key = `${x},${y}`;
                
                if (this.cellStates[key] === 'revealed') return;
                
                if (this.flaggedPositions.has(key)) {
                    this.flaggedPositions.delete(key);
                    this.questionPositions.add(key);
                } else if (this.questionPositions.has(key)) {
                    this.questionPositions.delete(key);
                } else {
                    this.flaggedPositions.add(key);
                }
            }
            
            checkWinCondition(field) {
                const totalSafeCells = this.width * this.height - this.minesCount;
                if (this.revealedCount === totalSafeCells) {
                    this.gameState = 'win';
                    this.stopTimer();
                }
            }
            
            getRemainingMines() {
                return this.minesCount - this.flaggedPositions.size;
            }
            
            startTimer() {
                this.startTime = Date.now();
                this.timer = setInterval(updateTimer, 1000);
            }
            
            stopTimer() {
                if (this.timer) {
                    clearInterval(this.timer);
                    this.timer = null;
                }
            }
            
            getElapsedTime() {
                if (!this.startTime) return 0;
                return Math.floor((Date.now() - this.startTime) / 1000);
            }
            
            setMode(mode) {
                this.currentMode = mode;
            }
        }

        let currentGame = null;
        let gameData = null;
        const game = new MinesweeperGame();

        function startNewGame() {
            const width = parseInt(document.getElementById('width').value) || 9;
            const height = parseInt(document.getElementById('height').value) || 9;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            game.width = width;
            game.height = height;
            game.minesCount = mines;
            
            gameData = game.generateGame();
            displayGame();
        }

        function generateWithSeed() {
            const seedInput = document.getElementById('customSeed').value;
            if (!seedInput) {
                alert('请输入种子号');
                return;
            }
            
            const width = parseInt(document.getElementById('width').value) || 9;
            const height = parseInt(document.getElementById('height').value) || 9;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            game.width = width;
            game.height = height;
            game.minesCount = mines;
            
            const seed = parseInt(seedInput) || seedInput.hashCode();
            gameData = game.generateGame(seed);
            displayGame();
        }

        function setMode(mode) {
            game.setMode(mode);
            document.querySelectorAll('.mode-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            event.target.classList.add('active');
        }

        function updateTimer() {
            if (game.gameState === 'playing' && game.startTime) {
                document.getElementById('gameTimer').textContent = game.getElapsedTime();
            }
        }

        function displayGame() {
            const gameField = document.getElementById('gameField');
            const currentSeed = document.getElementById('currentSeed');
            const totalCells = document.getElementById('totalCells');
            const remainingMines = document.getElementById('remainingMines');
            const revealedCells = document.getElementById('revealedCells');
            const gameTimer = document.getElementById('gameTimer');
            
            currentSeed.textContent = gameData.seed;
            totalCells.textContent = game.width * game.height;
            remainingMines.textContent = game.getRemainingMines();
            revealedCells.textContent = game.revealedCount;
            gameTimer.textContent = game.getElapsedTime();
            
            const fieldElement = document.createElement('div');
            fieldElement.className = 'field';
            fieldElement.style.gridTemplateColumns = `repeat(${game.width}, 24px)`;
            
            for (let y = 0; y < game.height; y++) {
                for (let x = 0; x < game.width; x++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.dataset.x = x;
                    cell.dataset.y = y;
                    
                    const key = `${x},${y}`;
                    const value = gameData.field[y][x];
                    const cellState = game.cellStates[key];
                    
                    if (cellState === 'revealed') {
                        cell.classList.add('revealed');
                        if (value === -1) {
                            cell.classList.add('mine');
                            cell.textContent = '雷';
                        } else if (value > 0) {
                            cell.classList.add(`number-${value}`);
                            cell.textContent = value;
                        }
                    } else {
                        if (game.flaggedPositions.has(key)) {
                            cell.classList.add('flag');
                        } else if (game.questionPositions.has(key)) {
                            cell.classList.add('question');
                        }
                    }
                    
                    if (game.gameState === 'gameover' && value === -1 && !game.flaggedPositions.has(key)) {
                        cell.classList.add('revealed', 'mine');
                        cell.textContent = '雷';
                    }
                    
                    cell.addEventListener('click', () => handleCellClick(x, y));
                    cell.addEventListener('contextmenu', (e) => {
                        e.preventDefault();
                        game.cycleCellMark(x, y);
                        displayGame();
                    });
                    
                    fieldElement.appendChild(cell);
                }
            }
            
            gameField.innerHTML = '';
            gameField.appendChild(fieldElement);
        }

        function handleCellClick(x, y) {
            game.handleCellClick(x, y, gameData.field);
            displayGame();
        }

        String.prototype.hashCode = function() {
            let hash = 0;
            for (let i = 0; i < this.length; i++) {
                const char = this.charCodeAt(i);
                hash = ((hash << 5) - hash) + char;
                hash = hash & hash;
            }
            return hash;
        };

        window.onload = startNewGame;
    </script>
</body>
</html>
