# 2048-6-6
It's one of 2048'games
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2048 Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#f65e3b',
                        secondary: '#edcf72',
                        bg: '#faf8ef',
                        grid: '#bbada0',
                        cellEmpty: '#cdc1b4',
                        textDark: '#776e65',
                        textLight: '#f9f6f2',
                    },
                    fontFamily: {
                        game: ['"Clear Sans"', '"Helvetica Neue"', 'Arial', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .text-shadow {
                text-shadow: 0 2px 4px rgba(0,0,0,0.1);
            }
            .animate-appear {
                animation: appear 0.2s ease-out;
            }
            .animate-merge {
                animation: merge 0.2s ease-out;
            }
            .animate-move {
                transition: transform 0.15s ease-out;
            }
        }

        @keyframes appear {
            0% { opacity: 0; transform: scale(0.8); }
            100% { opacity: 1; transform: scale(1); }
        }

        @keyframes merge {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body class="bg-bg font-game min-h-screen flex flex-col items-center justify-center p-4">
    <div class="max-w-md w-full mx-auto">
        <!-- Header Section -->
        <header class="flex flex-col md:flex-row md:items-center justify-between mb-4">
            <div>
                <h1 class="text-[clamp(2.5rem,5vw,3.5rem)] font-bold text-textDark mb-1">2048</h1>
                <p class="text-textDark/80 text-sm md:text-base">Join the numbers and get to the <strong>2048 tile!</strong></p>
            </div>
            <div class="flex gap-2 mt-4 md:mt-0">
                <div class="bg-grid rounded-md p-2 text-center">
                    <div class="text-xs text-textLight/80 uppercase">Score</div>
                    <div id="score" class="text-xl font-bold text-textLight">0</div>
                </div>
                <div class="bg-grid rounded-md p-2 text-center">
                    <div class="text-xs text-textLight/80 uppercase">Best</div>
                    <div id="best-score" class="text-xl font-bold text-textLight">0</div>
                </div>
            </div>
        </header>

        <!-- Game Controls -->
        <div class="flex justify-between items-center mb-4">
            <p class="text-textDark/80 text-sm">
                <i class="fa fa-arrow-up mr-1"></i>
                <i class="fa fa-arrow-right mr-1"></i>
                <i class="fa fa-arrow-down mr-1"></i>
                <i class="fa fa-arrow-left mr-2"></i>
                <span class="text-primary">W</span>/<span class="text-primary">A</span>/<span class="text-primary">S</span>/<span class="text-primary">D</span> to move.
            </p>
            <button id="new-game" class="bg-primary hover:bg-primary/90 text-white font-bold py-2 px-4 rounded-md transition-all duration-200 transform hover:scale-105 active:scale-95 shadow-md hover:shadow-lg">
                <i class="fa fa-refresh mr-1"></i> New Game
            </button>
        </div>

        <!-- Game Container -->
        <div class="relative">
            <!-- Grid Background -->
            <div class="bg-grid rounded-lg p-3">
                <div id="grid-container" class="grid grid-cols-6 gap-2">
                    <!-- Grid cells will be generated here -->
                </div>
            </div>

            <!-- Game Message (Win/Lose) -->
            <div id="game-message" class="hidden absolute inset-0 bg-textDark/80 rounded-lg flex flex-col items-center justify-center z-10">
                <p id="game-message-text" class="text-[clamp(1.5rem,3vw,2rem)] font-bold text-white mb-4"></p>
                <button id="game-message-button" class="bg-primary hover:bg-primary/90 text-white font-bold py-2 px-4 rounded-md transition-all duration-200">
                    Try Again
                </button>
            </div>
        </div>

        <!-- Game Instructions -->
        <div class="mt-6 text-textDark/80 text-sm">
            <h3 class="font-bold mb-2">How to play:</h3>
            <p>Use your <strong>arrow keys</strong> or <strong>W/A/S/D</strong> to move the tiles. When two tiles with the same number touch, they <strong>merge into one!</strong></p>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // Game variables
            const gridSize = 6;
            let grid = [];
            let score = 0;
            let bestScore = localStorage.getItem('2048-best-score') || 0;
            let gameOver = false;
            let gameWon = false;
            let canMove = true;
            
            // DOM elements
            const gridContainer = document.getElementById('grid-container');
            const scoreElement = document.getElementById('score');
            const bestScoreElement = document.getElementById('best-score');
            const newGameButton = document.getElementById('new-game');
            const gameMessage = document.getElementById('game-message');
            const gameMessageText = document.getElementById('game-message-text');
            const gameMessageButton = document.getElementById('game-message-button');
            
            // Tile colors based on value
            const tileColors = {
                2: { bg: '#eee4da', text: '#776e65' },
                4: { bg: '#ede0c8', text: '#776e65' },
                8: { bg: '#f2b179', text: '#f9f6f2' },
                16: { bg: '#f59563', text: '#f9f6f2' },
                32: { bg: '#f67c5f', text: '#f9f6f2' },
                64: { bg: '#f65e3b', text: '#f9f6f2' },
                128: { bg: '#edcf72', text: '#f9f6f2' },
                256: { bg: '#edcc61', text: '#f9f6f2' },
                512: { bg: '#edc850', text: '#f9f6f2' },
                1024: { bg: '#edc53f', text: '#f9f6f2' },
                2048: { bg: '#edc22e', text: '#f9f6f2' },
                // For values greater than 2048
                default: { bg: '#3c3a32', text: '#f9f6f2' }
            };
            
            // Initialize the game
            function initGame() {
                // Reset game state
                grid = createEmptyGrid();
                score = 0;
                gameOver = false;
                gameWon = false;
                canMove = true;
                
                // Update UI
                updateScore();
                updateBestScore();
                generateRandomTile();
                generateRandomTile();
                renderGrid();
                hideGameMessage();
            }
            
            // Create an empty grid
            function createEmptyGrid() {
                return Array(gridSize).fill().map(() => Array(gridSize).fill(null));
            }
            
            // Generate a random tile (2 or 4) at an empty position
            function generateRandomTile() {
                if (!hasEmptyCell()) return false;
                
                let emptyCells = [];
                for (let y = 0; y < gridSize; y++) {
                    for (let x = 0; x < gridSize; x++) {
                        if (grid[y][x] === null) {
                            emptyCells.push({ x, y });
                        }
                    }
                }
                
                const position = emptyCells[Math.floor(Math.random() * emptyCells.length)];
                const value = Math.random() < 0.9 ? 2 : 4;
                
                grid[position.y][position.x] = {
                    value,
                    merged: false,
                    new: true
                };
                
                return true;
            }
            
            // Check if there are any empty cells
            function hasEmptyCell() {
                for (let y = 0; y < gridSize; y++) {
                    for (let x = 0; x < gridSize; x++) {
                        if (grid[y][x] === null) {
                            return true;
                        }
                    }
                }
                return false;
            }
            
            // Render the grid
            function renderGrid() {
                gridContainer.innerHTML = '';
                
                for (let y = 0; y < gridSize; y++) {
                    for (let x = 0; x < gridSize; x++) {
                        const cell = document.createElement('div');
                        cell.className = 'aspect-square bg-cellEmpty rounded-md flex items-center justify-center';
                        
                        if (grid[y][x]) {
                            const { value, new: isNew, merged } = grid[y][x];
                            const tile = document.createElement('div');
                            
                            // Get tile colors
                            const colors = tileColors[value] || tileColors.default;
                            
                            // Set tile classes
                            tile.className = `w-full h-full rounded-md flex items-center justify-center font-bold text-shadow ${
                                isNew ? 'animate-appear' : ''
                            } ${
                                merged ? 'animate-merge' : 'animate-move'
                            }`;
                            
                            // Set tile styles
                            tile.style.backgroundColor = colors.bg;
                            tile.style.color = colors.text;
                            tile.style.fontSize = getFontSize(value);
                            
                            // Set tile content
                            tile.textContent = value;
                            
                            // Add tile to cell
                            cell.appendChild(tile);
                            
                            // Reset tile states
                            if (isNew) grid[y][x].new = false;
                            if (merged) grid[y][x].merged = false;
                        }
                        
                        gridContainer.appendChild(cell);
                    }
                }
            }
            
            // Get font size based on tile value
            function getFontSize(value) {
                if (value < 100) return 'clamp(1rem,2vw,1.5rem)';
                if (value < 1000) return 'clamp(0.8rem,1.8vw,1.2rem)';
                return 'clamp(0.7rem,1.5vw,1rem)';
            }
            
            // Update the score
            function updateScore() {
                scoreElement.textContent = score;
                
                // Update best score if needed
                if (score > bestScore) {
                    bestScore = score;
                    localStorage.setItem('2048-best-score', bestScore);
                    updateBestScore();
                }
            }
            
            // Update the best score
            function updateBestScore() {
                bestScoreElement.textContent = bestScore;
            }
            
            // Move tiles up
            function moveUp() {
                let moved = false;
                
                for (let x = 0; x < gridSize; x++) {
                    for (let y = 1; y < gridSize; y++) {
                        if (grid[y][x] !== null) {
                            let newY = y;
                            
                            // Find the highest possible position
                            while (newY > 0 && grid[newY - 1][x] === null) {
                                newY--;
                            }
                            
                            // Check if can merge
                            if (newY > 0 && grid[newY - 1][x] !== null && 
                                grid[newY - 1][x].value === grid[y][x].value && 
                                !grid[newY - 1][x].merged) {
                                // Merge tiles
                                grid[newY - 1][x].value *= 2;
                                grid[newY - 1][x].merged = true;
                                score += grid[newY - 1][x].value;
                                grid[y][x] = null;
                                moved = true;
                                
                                // Check for win condition
                                if (grid[newY - 1][x].value === 2048 && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('You Win!');
                                }
                            } 
                            // Move tile
                            else if (newY !== y) {
                                grid[newY][x] = grid[y][x];
                                grid[y][x] = null;
                                moved = true;
                            }
                        }
                    }
                }
                
                return moved;
            }
            
            // Move tiles down
            function moveDown() {
                let moved = false;
                
                for (let x = 0; x < gridSize; x++) {
                    for (let y = gridSize - 2; y >= 0; y--) {
                        if (grid[y][x] !== null) {
                            let newY = y;
                            
                            // Find the lowest possible position
                            while (newY < gridSize - 1 && grid[newY + 1][x] === null) {
                                newY++;
                            }
                            
                            // Check if can merge
                            if (newY < gridSize - 1 && grid[newY + 1][x] !== null && 
                                grid[newY + 1][x].value === grid[y][x].value && 
                                !grid[newY + 1][x].merged) {
                                // Merge tiles
                                grid[newY + 1][x].value *= 2;
                                grid[newY + 1][x].merged = true;
                                score += grid[newY + 1][x].value;
                                grid[y][x] = null;
                                moved = true;
                                
                                // Check for win condition
                                if (grid[newY + 1][x].value === 2048 && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('You Win!');
                                }
                            } 
                            // Move tile
                            else if (newY !== y) {
                                grid[newY][x] = grid[y][x];
                                grid[y][x] = null;
                                moved = true;
                            }
                        }
                    }
                }
                
                return moved;
            }
            
            // Move tiles left
            function moveLeft() {
                let moved = false;
                
                for (let y = 0; y < gridSize; y++) {
                    for (let x = 1; x < gridSize; x++) {
                        if (grid[y][x] !== null) {
                            let newX = x;
                            
                            // Find the leftmost possible position
                            while (newX > 0 && grid[y][newX - 1] === null) {
                                newX--;
                            }
                            
                            // Check if can merge
                            if (newX > 0 && grid[y][newX - 1] !== null && 
                                grid[y][newX - 1].value === grid[y][x].value && 
                                !grid[y][newX - 1].merged) {
                                // Merge tiles
                                grid[y][newX - 1].value *= 2;
                                grid[y][newX - 1].merged = true;
                                score += grid[y][newX - 1].value;
                                grid[y][x] = null;
                                moved = true;
                                
                                // Check for win condition
                                if (grid[y][newX - 1].value === 2048 && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('You Win!');
                                }
                            } 
                            // Move tile
                            else if (newX !== x) {
                                grid[y][newX] = grid[y][x];
                                grid[y][x] = null;
                                moved = true;
                            }
                        }
                    }
                }
                
                return moved;
            }
            
            // Move tiles right
            function moveRight() {
                let moved = false;
                
                for (let y = 0; y < gridSize; y++) {
                    for (let x = gridSize - 2; x >= 0; x--) {
                        if (grid[y][x] !== null) {
                            let newX = x;
                            
                            // Find the rightmost possible position
                            while (newX < gridSize - 1 && grid[y][newX + 1] === null) {
                                newX++;
                            }
                            
                            // Check if can merge
                            if (newX < gridSize - 1 && grid[y][newX + 1] !== null && 
                                grid[y][newX + 1].value === grid[y][x].value && 
                                !grid[y][newX + 1].merged) {
                                // Merge tiles
                                grid[y][newX + 1].value *= 2;
                                grid[y][newX + 1].merged = true;
                                score += grid[y][newX + 1].value;
                                grid[y][x] = null;
                                moved = true;
                                
                                // Check for win condition
                                if (grid[y][newX + 1].value === 2048 && !gameWon) {
                                    gameWon = true;
                                    showGameMessage('You Win!');
                                }
                            } 
                            // Move tile
                            else if (newX !== x) {
                                grid[y][newX] = grid[y][x];
                                grid[y][x] = null;
                                moved = true;
                            }
                        }
                    }
                }
                
                return moved;
            }
            
            // Handle keyboard controls - 添加WASD支持
            function handleKeydown(e) {
                if (!canMove || gameOver) return;
                
                let moved = false;
                let key = e.key.toLowerCase();
                
                // Map WASD to arrow keys
                if (key === 'w') key = 'arrowup';
                if (key === 'a') key = 'arrowleft';
                if (key === 's') key = 'arrowdown';
                if (key === 'd') key = 'arrowright';
                
                switch (key) {
                    case 'arrowup':
                        moved = moveUp();
                        break;
                    case 'arrowdown':
                        moved = moveDown();
                        break;
                    case 'arrowleft':
                        moved = moveLeft();
                        break;
                    case 'arrowright':
                        moved = moveRight();
                        break;
                    default:
                        return; // Exit if not a recognized key
                }
                
                // Prevent page scrolling when using arrow keys or WASD
                if (['arrowup', 'arrowdown', 'arrowleft', 'arrowright', 'w', 'a', 's', 'd'].includes(e.key.toLowerCase())) {
                    e.preventDefault();
                }
                
                if (moved) {
                    canMove = false;
                    renderGrid();
                    
                    // Add a small delay before generating new tile for better animation
                    setTimeout(() => {
                        generateRandomTile();
                        renderGrid();
                        updateScore();
                        
                        // Check if game is over
                        if (!canMoveAny() && !gameOver) {
                            gameOver = true;
                            showGameMessage('Game Over!');
                        }
                        
                        canMove = true;
                    }, 150);
                }
            }
            
            // Check if any moves are possible
            function canMoveAny() {
                // Check for empty cells
                if (hasEmptyCell()) return true;
                
                // Check for possible merges
                for (let y = 0; y < gridSize; y++) {
                    for (let x = 0; x < gridSize; x++) {
                        const value = grid[y][x].value;
                        
                        // Check right
                        if (x < gridSize - 1 && grid[y][x + 1] && grid[y][x + 1].value === value) {
                            return true;
                        }
                        
                        // Check down
                        if (y < gridSize - 1 && grid[y + 1][x] && grid[y + 1][x].value === value) {
                            return true;
                        }
                    }
                }
                
                return false;
            }
            
            // Show game message
            function showGameMessage(text) {
                gameMessageText.textContent = text;
                gameMessage.classList.remove('hidden');
                gameMessage.classList.add('flex');
            }
            
            // Hide game message
            function hideGameMessage() {
                gameMessage.classList.remove('flex');
                gameMessage.classList.add('hidden');
            }
            
            // Event listeners
            document.addEventListener('keydown', handleKeydown);
            
            // Mobile touch controls
            let touchStartX = 0;
            let touchStartY = 0;
            
            document.addEventListener('touchstart', (e) => {
                if (!canMove || gameOver) return;
                touchStartX = e.touches[0].clientX;
                touchStartY = e.touches[0].clientY;
            }, { passive: true });
            
            document.addEventListener('touchend', (e) => {
                if (!canMove || gameOver) return;
                
                const touchEndX = e.changedTouches[0].clientX;
                const touchEndY = e.changedTouches[0].clientY;
                
                const diffX = touchEndX - touchStartX;
                const diffY = touchEndY - touchStartY;
                
                // Determine swipe direction based on which difference is greater
                if (Math.abs(diffX) > Math.abs(diffY)) {
                    // Horizontal swipe
                    if (diffX > 20) {
                        // Swipe right
                        handleKeydown({ key: 'ArrowRight' });
                    } else if (diffX < -20) {
                        // Swipe left
                        handleKeydown({ key: 'ArrowLeft' });
                    }
                } else {
                    // Vertical swipe
                    if (diffY > 20) {
                        // Swipe down
                        handleKeydown({ key: 'ArrowDown' });
                    } else if (diffY < -20) {
                        // Swipe up
                        handleKeydown({ key: 'ArrowUp' });
                    }
                }
            }, { passive: true });
            
            newGameButton.addEventListener('click', initGame);
            gameMessageButton.addEventListener('click', initGame);
            
            // Initialize the game
            initGame();
        });
    </script>
</body>
</html>
    
