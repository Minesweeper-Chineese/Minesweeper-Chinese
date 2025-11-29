<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>扫雷艇</title>
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
        }
        .cell.mine {
            background: #ff4444;
            color: white;
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
    </style>
</head>
<body>
    <div class="container">
        <h1>中文扫雷舰</h1>
        
        <div class="controls">
            <div class="control-group">
                <label for="width">阔度:</label>
                <input type="number" id="width" value="8" min="5" max="20">
                
                <label for="height">身高:</label>
                <input type="number" id="height" value="8" min="5" max="20">
            </div>
            
            <div class="control-group">
                <label for="mines">地雷数目:</label>
                <input type="number" id="mines" value="10" min="1" max="50">
            </div>
            
            <div class="control-group">
                <button onclick="generateNewGame()">新游戏</button>
                <button onclick="generateWithSeed()">🔑生成 seed</button>
                <input type="text" id="customSeed" placeholder="写 seed">
            </div>
        </div>

        <div class="game-info">
            <strong>Seed当前的游戏:</strong>
            <div class="seed-info" id="currentSeed">-</div>
        </div>

        <div id="gameField"></div>
        
        <div class="game-info">
            <strong>统计数字:</strong>
            <div>总细胞: <span id="totalCells">0</span></div>
          <div>地雷数目: <span id="minesCount">0</span></div>
            <div>地雷的百分比: <span id="minesPercent">0%</span></div>
        </div>
    </div>

    <script>
        class MinesweeperGenerator {
            constructor(width = 8, height = 8, minesCount = 10) {
                this.width = width;
                this.height = height;
                this.minesCount = minesCount;
                this.seed = null;
                this.usedSeeds = new Set();
            }
            
            generateGame(seed = null) {
                if (seed) {
                    this.seed = seed;
                } else {
                    this.seed = this.generateUniqueSeed();
                }
                
                this.setSeed(this.seed);
                
                const field = Array(this.height).fill().map(() => 
                    Array(this.width).fill(0)
                );
                
                const minesPositions = this.placeMinesRandomly();
                this.updateNumbers(field, minesPositions);
                
                return {
                    field: field,
                    mines: minesPositions,
                    seed: this.seed,
                    width: this.width,
                    height: this.height
                };
            }
            
            generateUniqueSeed() {
                const timestamp = Date.now();
                const randomFactor = Math.floor(Math.random() * 1000000);
                let seed = timestamp + randomFactor;
                
                let attempts = 0;
                while (this.usedSeeds.has(seed) && attempts < 1000) {
                    seed = timestamp + Math.floor(Math.random() * 1000000) + ++attempts;
                }
                
                this.usedSeeds.add(seed);
                return seed;
            }
            
            setSeed(seed) {
                let x = Math.sin(seed) * 10000;
                this.random = () => {
                    x = Math.sin(x * 10000) * 10000;
                    return x - Math.floor(x);
                };
            }
            
            placeMinesRandomly() {
                const minesPositions = [];
                const totalCells = this.width * this.height;
                
                if (this.minesCount >= totalCells) {
                    this.minesCount = totalCells - 1;
                }
                
                while (minesPositions.length < this.minesCount) {
                    const x = Math.floor(this.random() * this.width);
                    const y = Math.floor(this.random() * this.height);
                    const position = `${x},${y}`;
                    
                    if (!minesPositions.includes(position)) {
                        minesPositions.push(position);
                    }
                }
                
                return minesPositions;
            }
            
            updateNumbers(field, minesPositions) {
                minesPositions.forEach(pos => {
                    const [x, y] = pos.split(',').map(Number);
                    field[y][x] = -1;
                });
                
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
        }

        let currentGame = null;
        const generator = new MinesweeperGenerator();

        function generateNewGame() {
            const width = parseInt(document.getElementById('width').value) || 8;
            const height = parseInt(document.getElementById('height').value) || 8;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            generator.width = width;
            generator.height = height;
            generator.minesCount = mines;
            
            currentGame = generator.generateGame();
            displayGame(currentGame);
        }

        function generateWithSeed() {
            const seedInput = document.getElementById('customSeed').value;
            if (!seedInput) {
                alert('写seed生成');
                return;
            }
            
            const width = parseInt(document.getElementById('width').value) || 8;
            const height = parseInt(document.getElementById('height').value) || 8;
            const mines = parseInt(document.getElementById('mines').value) || 10;
            
            generator.width = width;
            generator.height = height;
            generator.minesCount = mines;
            
            const seed = parseInt(seedInput) || seedInput.hashCode();
            currentGame = generator.generateGame(seed);
            displayGame(currentGame);
        }

        function displayGame(game) {
            const gameField = document.getElementById('gameField');
            const currentSeed = document.getElementById('currentSeed');
            const totalCells = document.getElementById('totalCells');
            const minesCount = document.getElementById('minesCount');
            const minesPercent = document.getElementById('minesPercent');
            
            currentSeed.textContent = game.seed;
            totalCells.textContent = game.width * game.height;
            minesCount.textContent = game.mines.length;
            minesPercent.textContent = ((game.mines.length / (game.width * game.height)) * 100).toFixed(1) + '%';
            
            const fieldElement = document.createElement('div');
            fieldElement.className = 'field';
            fieldElement.style.gridTemplateColumns = `repeat(${game.width}, 32px)`;
            
            for (let y = 0; y < game.height; y++) {
                for (let x = 0; x < game.width; x++) {
                    const cell = document.createElement('div');
                    cell.className = 'cell';
                    cell.dataset.x = x;
                    cell.dataset.y = y;
                    
                    const value = game.field[y][x];
                    
                    if (value === -1) {
                        cell.className += ' mine';
                        cell.textContent = '地雷!';
                    } else if (value > 0) {
                        cell.className += ` number-${value}`;
                        cell.textContent = value;
                    }
                    
                    cell.title = `Клетка [${x}, ${y}]`;
                    fieldElement.appendChild(cell);
                }
            }
            
            gameField.innerHTML = '';
            gameField.appendChild(fieldElement);
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

        window.onload = generateNewGame;
    </script>
</body>
</html>
