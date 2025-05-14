# 1win.xaker

<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>3-в-1 Игры</title>
  <style>
    body {
      font-family: sans-serif;
      background: linear-gradient(to bottom right, #0f0f0f, #1e1e1e);
      color: #fff;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 30px;
    }

    h1 {
      margin-bottom: 20px;
    }

    .game-selection {
      margin-bottom: 20px;
      display: flex;
      gap: 12px;
      flex-wrap: wrap;
      justify-content: center;
    }

    .game-selection button,
    .buttons button {
      margin: 0 10px;
      padding: 10px 20px;
      font-size: 16px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      background: linear-gradient(145deg, #444, #222);
      color: white;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
      transition: all 0.3s ease;
    }

    .game-selection button:hover,
    .buttons button:hover {
      background: linear-gradient(145deg, #666, #333);
      transform: scale(1.05);
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(5, 50px);
      gap: 8px;
      margin-bottom: 20px;
    }

    .cell {
      width: 50px;
      height: 50px;
      background-color: #2b2b2b;
      border: 2px solid #000000;
      border-radius: 6px;
      cursor: pointer;
      font-size: 24px;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.6s ease, box-shadow 0.6s ease, transform 0.6s ease;
      opacity: 0;
    }

    .cell.visible {
      opacity: 1;
      transform: scale(1.1);
    }

    .cell.open {
      background-color: #444;
    }

    .cell.bomb {
      background-color: #8b0000;
    }

    .cell.open.glow {
      box-shadow: 0 0 10px 3px #f0f000;
      background-color: #555;
    }

    .mine-settings {
      margin-bottom: 10px;
      color: #ccc;
      display: flex;
      gap: 10px;
      align-items: center;
      justify-content: center;
    }

    .mine-settings label {
      margin-right: 5px;
    }

    .back-button {
      display: flex;
      justify-content: center;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <h1 id="mainTitle">🎮 Выберите игру</h1>

  <div class="game-selection" id="gameSelection">
    <button onclick="showGame('mines')">🧨 Mines</button>
    <button onclick="showGame('penalty')">⚽ Penalty</button>
    <button onclick="showGame('bomb')">💣 Bomb</button>
  </div>

  <div id="gameContainer"></div>

  <script>
    function showGame(game) {
      document.getElementById('mainTitle').style.display = 'none';
      document.getElementById('gameSelection').style.display = 'none';

      if (game === 'mines') {
        showMinesGame();
      } else if (game === 'penalty') {
        showPenaltyGame();
      } else if (game === 'bomb') {
        showBombGame();
      }
    }

    let gameStarted = false;
    const gridSize = 5;
    let minePositions = [];
    let cells = [];
    let selectedMineCount = 5;

    function showBackButton() {
      const backBtn = document.createElement('div');
      backBtn.classList.add('back-button');
      backBtn.innerHTML = '<button onclick="backToMenu()">🔙 Назад</button>';
      document.getElementById('gameContainer').prepend(backBtn);
    }

    function backToMenu() {
      document.getElementById('mainTitle').style.display = 'block';
      document.getElementById('gameSelection').style.display = 'flex';
      document.getElementById('gameContainer').innerHTML = '';
    }

    function showMinesGame() {
      document.getElementById('gameContainer').innerHTML = `
        <h2>🧨 Игра Mines</h2>
        <div class="mine-settings">
          <label for="mineCount">Мин:</label>
          <input type="number" id="mineCount" value="5" min="1" max="24" style="width: 50px; background: #333; color: #fff; border: 1px solid #555;">
        </div>
        <div class="grid" id="grid"></div>
        <div class="buttons">
          <button onclick="resetCells()">📶 Получить сигнал</button>
          <button onclick="startGame()">🔁 Старт</button>
        </div>
      `;
      gameStarted = false;
      renderGrid();
      showBackButton();
    }

    function renderGrid() {
      const grid = document.getElementById('grid');
      grid.innerHTML = '';
      cells = [];
      for (let i = 0; i < gridSize * gridSize; i++) {
        const cell = document.createElement('div');
        cell.classList.add('cell');
        cell.dataset.index = i;
        cell.addEventListener('click', handleCellClick);
        grid.appendChild(cell);
        cells.push(cell);
        setTimeout(() => {
          cell.classList.add('visible');
        }, i * 20);
      }
    }

    async function startGame() {
      selectedMineCount = parseInt(document.getElementById('mineCount').value);
      minePositions = [];
      gameStarted = true;
      renderGrid();

      while (minePositions.length < selectedMineCount) {
        const index = Math.floor(Math.random() * gridSize * gridSize);
        if (!minePositions.includes(index)) {
          minePositions.push(index);
        }
      }

      let opened = 0;
      while (opened < 4) {
        const index = Math.floor(Math.random() * gridSize * gridSize);
        if (!minePositions.includes(index) && !cells[index].classList.contains('open')) {
          await openCellSmooth(index);
          opened++;
        }
      }
    }

    async function openCellSmooth(index) {
      return new Promise(resolve => {
        setTimeout(() => {
          cells[index].classList.add('open', 'glow');
          cells[index].textContent = '⭐';
          resolve();
        }, 400);
      });
    }

    function handleCellClick(e) {
      if (!gameStarted) return;
    }

    function resetCells() {
      cells.forEach(cell => {
        cell.classList.remove('open', 'bomb', 'glow', 'visible');
        cell.textContent = '';
      });
      gameStarted = false;
    }

    function showPenaltyGame() {
      const goalie = ["left", "center", "right"][Math.floor(Math.random() * 3)];
      document.getElementById('gameContainer').innerHTML = `
        <h2>⚽ Игра Penalty</h2>
        <p>Куда будете бить?</p>
        <div class="buttons">
          <button onclick="checkPenalty('left', '${goalie}')">⬅️ Влево</button>
          <button onclick="checkPenalty('center', '${goalie}')">⬆️ В центр</button>
          <button onclick="checkPenalty('right', '${goalie}')">➡️ Вправо</button>
        </div>
      `;
      showBackButton();
    }

    function checkPenalty(choice, goalie) {
      const result = choice == goalie ? "❌ Вратарь поймал!" : "✅ ГОЛ!";
      document.getElementById("gameContainer").innerHTML += `<p>${result}</p>`;
    }

    function showBombGame() {
      const bomb = Math.floor(Math.random() * 3) + 1;
      document.getElementById('gameContainer').innerHTML = `
        <h2>💣 Игра Bomb</h2>
        <p>Один из ящиков содержит бомбу. Какой выберете?</p>
        <div class="buttons">
          <button onclick="checkBomb(1, ${bomb})">📦 Ящик 1</button>
          <button onclick="checkBomb(2, ${bomb})">📦 Ящик 2</button>
          <button onclick="checkBomb(3, ${bomb})">📦 Ящик 3</button>
        </div>
      `;
      showBackButton();
    }

    function checkBomb(choice, bomb) {
      const result = choice == bomb ? "💣 Бомба!" : "✅ Безопасно!";
      document.getElementById("gameContainer").innerHTML += `<p>${result}</p>`;
    }
  </script>
</body>
</html>

