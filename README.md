<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>扫雷游戏</title>
    <style>
        body {
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f0f0f0;
            text-align: center;
        }
        .container {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .controls {
            margin-bottom: 20px;
            padding: 15px;
            background: #e8f4fd;
            border-radius: 5px;
        }
        .control-group {
            margin: 10px 0;
        }
        label {
            display: inline-block;
            width: 100px;
            text-align: right;
            margin-right: 10px;
        }
        input, button, select {
            padding: 8px 12px;
            margin: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            background: #45a049;
        }
        .mode-selector {
            margin: 10px 0;
        }
        .mode-btn {
            background: #6c757d;
            margin: 2px;
        }
        .mode-btn.active {
            background: #007bff;
        }
        .field {
            display: inline-grid;
            gap: 1px;
            margin: 20px auto;
            background: #bdbdbd;
            padding: 8px;
            border: 3px outset #bdbdbd;
            border-radius: 2px;
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
            content: "🚩";
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
        .game-info {
            margin: 10px 0;
            padding: 10px;
            background: #f8f9fa;
            border-radius: 5px;
        }
        .seed-info {
            font-family: monospace;
            background: #eee;
            padding: 5px;
            border-radius: 3px;
            word-break: break-all;
        }
        .game-status {
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
            text-align: center;
            font-weight: bold;
            font-size: 18px;
        }
        .status-playing {
            background: #e8f5e8;
            color: #2e7d32;
        }
        .status-gameover {
            background: #ffebee;
            color: #c62828;
        }
        .status-win {
            background: #e8f5e8;
            color: #2e7d32;
        }
        .mines-counter {
            font-family: 'Digital', monospace;
            background: #000;
            color: #f00;
            padding: 5px 10px;
            border-radius: 3px;
            font-size: 18px;
            font-weight: bold;
        }
        .game-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
            padding: 10px;
            background: #bdbdbd;
            border: 3px outset #bdbdbd;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>扫雷游戏</h1>
        
        <div class="game-header">
            <div class="mines-counter">
                地雷: <span id="remainingMines">10</span>
            </div>
            <button onclick="startNewGame()" style="font-size: 20px;">😊</button>
            <div class="timer">
                时间: <span id="gameTimer">0</span>
            </div>
        </div>

        <div class="game-status status-playing" id="gameStatus">
            游戏进行中
        </div>
        
        <div class="controls">
            <div class="control-group">
                <label for="width">宽度:</label>
                <input type="number" id="width" value="9" min="5" max="20">
                
                <label for="height">高度:</label>
                <input type="number" id="height" value="9" min="5" max="20">
            </div>
            
            <div class="control-group">
                <label for="mines">地雷数目:</label>
                <input type="number" id="mines" value="10" min="1" max="50">
            </div>

            <div class="control-group">
                <label>游戏模式:</label>
                <button class="mode-btn active" onclick="setMode('reveal')">点击模式</button>
                <button class="mode-btn" onclick="setMode('flag')">标记模式</button>
            </div>
            
            <div class="control-group">
                <button onclick="startNewGame()">新游戏</button>
                <button onclick="generateWithSeed()">使用种子</button>
                <input type="text" id="customSeed" placeholder="输入种子">
            </div>
        </div>

        <div id="gameField"></div>
        
        <div class="game-info">
            <strong>游戏统计:</strong>
            <div>已翻开: <span id="revealedCells">0</span> / <span id="totalCells">81</span></div>
            <div>种子: <span class="seed-info" id="currentSeed">-</span></div>
        </div>
    </div>

    <script>
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
                this.currentMode = 'reveal'; // 'reveal' or 'flag'
                this.cellStates = {}; // Track each cell's display state
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
                    
                    // 确保第一个点击的格子不是地雷
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
                alert('请输入种子');
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
            const gameStatus = document.getElementById('gameStatus');
            const gameTimer = document.getElementById('gameTimer');
            
            currentSeed.textContent = gameData.seed;
            totalCells.textContent = game.width * game.height;
            remainingMines.textContent = game.getRemainingMines();
            revealedCells.textContent = game.revealedCount;
            gameTimer.textContent = game.getElapsedTime();
            
            // 更新游戏状态显示
            const statusElement = document.getElementById('gameStatus');
            if (game.gameState === 'playing') {
                statusElement.className = 'game-status status-playing';
                statusElement.textContent = '游戏进行中';
            } else if (game.gameState === 'gameover') {
                statusElement.className = 'game-status status-gameover';
                statusElement.textContent = '游戏结束！你踩到地雷了！';
            } else if (game.gameState === 'win') {
                statusElement.className = 'game-status status-win';
                statusElement.textContent = '恭喜！你赢了！';
            }
            
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
                    
                    // 设置单元格内容
                    if (cellState === 'revealed') {
                        cell.classList.add('revealed');
                        if (value === -1) {
                            cell.classList.add('mine');
                            cell.textContent = '💣';
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
                    
                    // 游戏结束时显示所有地雷
                    if (game.gameState === 'gameover' && value === -1 && !game.flaggedPositions.has(key)) {
                        cell.classList.add('revealed', 'mine');
                        cell.textContent = '💣';
                    }
                    
                    // 添加点击事件
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
