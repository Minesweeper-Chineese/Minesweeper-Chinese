<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>扫雷游戏</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            background-color: #f0f0f0;
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
            width: 120px;
        }
        input, button {
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
        .field {
            display: grid;
            gap: 2px;
            margin: 20px 0;
            background: #bbb;
            padding: 5px;
            border-radius: 5px;
        }
        .cell {
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            background: #ccc;
            border: 2px outset #999;
            font-weight: bold;
            cursor: pointer;
            user-select: none;
            font-size: 14px;
        }
        .cell.revealed {
            border: 1px solid #999;
            background: #e0e0e0;
        }
        .cell.mine {
            background: #ff4444;
            color: white;
        }
        .cell.flag {
            background: #ffeb3b;
            color: #ff5722;
        }
        .cell.number-1 { color: blue; }
        .cell.number-2 { color: green; }
        .cell.number-3 { color: red; }
        .cell.number-4 { color: darkblue; }
        .cell.number-5 { color: darkred; }
        .cell.number-6 { color: teal; }
        .cell.number-7 { color: black; }
        .cell.number-8 { color: gray; }
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
    </style>
</head>
<body>
    <div class="container">
        <h1>扫雷游戏</h1>
        
        <div class="game-status status-playing" id="gameStatus">
            游戏进行中 - 左键点击翻开格子，右键标记地雷
        </div>
        
        <div class="controls">
            <div class="control-group">
                <label for="width">宽度:</label>
                <input type="number" id="width" value="8" min="5" max="20">
                
                <label for="height">高度:</label>
                <input type="number" id="height" value="8" min="5" max="20">
            </div>
            
            <div class="control-group">
                <label for="mines">地雷数目:</label>
                <input type="number" id="mines" value="10" min="1" max="50">
            </div>
            
            <div class="control-group">
                <button onclick="startNewGame()">新游戏</button>
                <button onclick="generateWithSeed()">使用种子生成</button>
                <input type="text" id="customSeed" placeholder="输入种子">
            </div>
        </div>

        <div class="game-info">
            <strong>当前游戏种子:</strong>
            <div class="seed-info" id="currentSeed">-</div>
        </div>

        <div id="gameField"></div>
        
        <div class="game-info">
            <strong>游戏统计:</strong>
            <div>剩余地雷: <span id="remainingMines">10</span></div>
            <div>已标记: <span id="flaggedCells">0</span></div>
            <div>总格子数: <span id="totalCells">0</span></div>
        </div>
    </div>

    <script>
        class MinesweeperGame {
            constructor(width = 8, height = 8, minesCount = 10) {
                this.width = width;
                this.height = height;
                this.minesCount = minesCount;
                this.seed = null;
                this.gameState = 'playing'; // playing, gameover, win
                this.revealedCount = 0;
                this.flaggedPositions = new Set();
                this.firstClick = true;
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
                this.firstClick = true;
                
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
                const timestamp = Date.now();
                const randomFactor = Math.floor(Math.random() * 1000000);
                return timestamp + randomFactor;
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
            
            revealCell(x, y, field, mines) {
                if (this.gameState !== 'playing') return false;
                if (this.flaggedPositions.has(`${x},${y}`)) return false;
                
                if (this.firstClick) {
                    this.placeMines(x, y, field);
                    this.firstClick = false;
                }
                
                if (field[y][x] === -1) {
                    this.gameState = 'gameover';
                    return true;
                }
                
                if (field[y][x] === 0) {
                    this.revealEmptyArea(x, y, field);
                } else {
                    this.revealedCount++;
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
                    this.revealedCount++;
                    
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
            
            toggleFlag(x, y) {
                if (this.gameState !== 'playing') return;
                
                const key = `${x},${y}`;
                if (this.flaggedPositions.has(key)) {
                    this.flaggedPositions.delete(key);
                } else {
                    this.flaggedPositions.add(key);
                }
            }
            
            checkWinCondition(field) {
                const totalSafeCells = this.width * this.height - this.minesCount;
                if (this.revealedCount === totalSafeCells) {
                    this.gameState = 'win';
                }
            }
            
            getRemainingMines() {
                return this.minesCount - this.flaggedPositions.size;
            }
        }

        let currentGame = null;
        let gameData = null;
        const game = new MinesweeperGame();

        function startNewGame() {
            const width = parseInt(document.getElementById('width').value) || 8;
            const height = parseInt(document.getElementById('height').value) || 8;
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
            
            const width = parseInt(document.getElementById('width').value) || 8;
            const height = parseInt(document.getElementById('height').value) || 8;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            game.width = width;
            game.height = height;
            game.minesCount = mines;
            
            const seed = parseInt(seedInput) || seedInput.hashCode();
            gameData = game.generateGame(seed);
            displayGame();
        }

        function displayGame() {
            const gameField = document.getElementById('gameField');
            const currentSeed = document.getElementById('currentSeed');
            const totalCells = document.getElementById('totalCells');
            const remainingMines = document.getElementById('remainingMines');
            const flaggedCells = document.getElementById('flaggedCells');
            const gameStatus = document.getElementById('gameStatus');
            
            currentSeed.textContent = gameData.seed;
            totalCells.textContent = game.width * game.height;
            remainingMines.textContent = game.getRemainingMines();
            flaggedCells.textContent = game.flaggedPositions.size;
            
            // 更新游戏状态显示
            if (game.gameState === 'playing') {
                gameStatus.className = 'game-status status-playing';
                gameStatus.textContent = '游戏进行中 - 左键点击翻开格子，右键标记地雷';
            } else if (game.gameState === 'gameover') {
                gameStatus.className = 'game-status status-gameover';
                gameStatus.textContent = '游戏结束！你踩到地雷了！';
            } else if (game.gameState === 'win') {
                gameStatus.className = 'game-status status-win';
                gameStatus.textContent = '恭喜！你赢了！';
            }
            
            const fieldElement = document.createElement('div');
            fieldElement.className = 'field';
            fieldElement.style.gridTemplateColumns = `repeat(${game.width}, 32px)`;
            
            for (let y = 0; y < game.height; y++) {
                for (let x = 0; x < game.width; x++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.dataset.x = x;
                    cell.dataset.y = y;
                    
                    const key = `${x},${y}`;
                    const isFlagged = game.flaggedPositions.has(key);
                    
                    if (isFlagged) {
                        cell.className += ' flag';
                        cell.textContent = '🚩';
                    } else if (game.gameState !== 'playing' && gameData.field[y][x] === -1) {
                        // 游戏结束或胜利时显示所有地雷
                        cell.className += ' mine';
                        cell.textContent = '阿！';
                    } else if (game.revealedCount > 0 && !game.firstClick) {
                        // 这里应该根据实际游戏状态显示已翻开的格子
                        // 简化显示，实际游戏中需要跟踪每个格子的状态
                    }
                    
                    // 添加点击事件
                    cell.addEventListener('click', () => handleCellClick(x, y));
                    cell.addEventListener('contextmenu', (e) => {
                        e.preventDefault();
                        handleRightClick(x, y);
                    });
                    
                    cell.title = `格子 [${x}, ${y}]`;
                    fieldElement.appendChild(cell);
                }
            }
            
            gameField.innerHTML = '';
            gameField.appendChild(fieldElement);
        }

        function handleCellClick(x, y) {
            if (game.gameState !== 'playing') return;
            
            const revealed = game.revealCell(x, y, gameData.field);
            if (revealed) {
                displayGame();
            }
        }

        function handleRightClick(x, y) {
            game.toggleFlag(x, y);
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
