# Artificial-Intelligence.Project-
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Hill Climb Racing AI - Night Mode</title>
  <style>
    body {
      font-family: sans-serif;
      margin: 0;
      background: url("C:/Users/muska/OneDrive/Pictures/screenshots/Screenshot 2025-04-15 101955.png") no-repeat center center fixed;
      background-size: cover;
      color: #09074d;
      text-align: center;
    }

    body.night {
      background: url("C:/Users/muska/OneDrive/Pictures/screenshots/Screenshot 2025-04-15 102109.png") no-repeat center center fixed;
      background-size: cover;
      color: #97cbe4;
    }

    h1 {
      margin-top: 10px;
      font-size: 2rem;
      text-shadow: 2px 2px 4px #000;
    }

    #info {
      background: rgba(0, 0, 0, 0.5);
      margin: 10px;
      padding: 10px;
      border-radius: 10px;
    }

    #game {
      display: flex;
      justify-content: center;
      align-items: flex-end;
      height: 300px;
      margin: 10px auto;
      max-width: 100%;
      overflow-x: auto;
    }

    .tile {
      width: 30px;
      position: relative;
      display: flex;
      flex-direction: column-reverse;
      margin: 0 1px;
    }

    .hill {
      background: #5d4037;
      width: 100%;
      border-radius: 4px;
    }

    body.night .hill {
      background: #2c2c2c;
    }

    .car {
      position: absolute;
      bottom: 100%;
      width: 30px;
      height: auto;
      font-size: 24px;
      z-index: 2;
    }

    body.night .car::after {
      content: '';
      position: absolute;
      left: 30px;
      top: 2px;
      width: 60px;
      height: 15px;
      background: radial-gradient(ellipse at left, rgba(255,255,160,0.8) 0%, transparent 70%);
      opacity: 0.8;
      pointer-events: none;
      animation: flicker 0.2s infinite alternate;
    }

    @keyframes flicker {
      0% { opacity: 0.6; }
      100% { opacity: 1; }
    }

    .fuel {
      position: absolute;
      bottom: 100%;
      width: 20px;
      height: 20px;
      background: gold;
      border-radius: 5px;
      left: 5px;
      z-index: 1;
    }

    .controls {
      margin-top: 15px;
    }

    .controls button {
      font-size: 24px;
      padding: 10px 15px;
      margin: 5px;
      border-radius: 10px;
      border: none;
      background-color: #333;
      color: white;
      box-shadow: 2px 2px 5px #000;
      cursor: pointer;
    }

    @media (max-width: 600px) {
      .tile {
        width: 20px;
      }
      .car {
        font-size: 18px;
      }
    }
  </style>
</head>
<body>
  <h1>🏔 Hill Climb Racing AI</h1>
  <div id="info"></div>
  <div id="game"></div>
  <div class="controls">
    <button onclick="moveLeft()">⬅️</button>
    <button onclick="moveRight()">➡️</button>
    <button onclick="restartGame()">🔄</button>
    <button onclick="toggleNightMode()">🌙 Night Mode</button>
  </div>

  <script>
    const WIDTH = 20;
    const MAX_HEIGHT = 6;
    const initialEnergy = 20;
    const GOAL = WIDTH - 1;

    let heights, fuelPoints, carPos, energy, manualMode, gameOver;

    const gameDiv = document.getElementById("game");
    const infoDiv = document.getElementById("info");

    function generateFuelPoints() {
      const points = new Set();
      while (points.size < 3) {
        const pos = Math.floor(Math.random() * (WIDTH - 6)) + 3;
        points.add(pos);
      }
      return Array.from(points);
    }

    function drawTerrain(carPosition = -1) {
      gameDiv.innerHTML = '';
      for (let i = 0; i < WIDTH; i++) {
        const col = document.createElement('div');
        col.className = 'tile';
        col.style.height = (heights[i] * 30) + 'px';

        const hill = document.createElement('div');
        hill.className = 'hill';
        hill.style.height = (heights[i] * 30) + 'px';

        if (fuelPoints.includes(i)) {
          const fuel = document.createElement('div');
          fuel.className = 'fuel';
          col.appendChild(fuel);
        }

        if (i === carPosition) {
          const car = document.createElement('div');
          car.className = 'car';
          car.textContent = '🚙';
          col.appendChild(car);
        }

        col.appendChild(hill);
        gameDiv.appendChild(col);
      }
    }

    function heuristic(pos, goal) {
      return Math.abs(goal - pos);
    }

    function terrainCost(current, next) {
      const rise = heights[next] - heights[current];
      return 1 + Math.max(0, rise);
    }

    function bestFirstSearch(start, goal, energy) {
      const frontier = [{ pos: start, energy, path: [start], cost: heuristic(start, goal) }];
      const visited = new Set();

      while (frontier.length) {
        frontier.sort((a, b) => a.cost - b.cost);
        const current = frontier.shift();
        const key = `${current.pos}-${current.energy}`;
        if (visited.has(key)) continue;
        visited.add(key);

        if (current.pos === goal) return current.path;

        for (let move of [1, 2]) {
          const nextPos = current.pos + move;
          if (nextPos > goal) continue;

          let cost = terrainCost(current.pos, nextPos);
          let newEnergy = current.energy - cost;

          if (newEnergy >= 0) {
            if (fuelPoints.includes(nextPos)) newEnergy += 3;
            frontier.push({
              pos: nextPos,
              energy: newEnergy,
              path: [...current.path, nextPos],
              cost: heuristic(nextPos, goal)
            });
          }
        }
      }
      return null;
    }

    async function animatePath(path) {
      for (const pos of path) {
        if (manualMode) return;
        carPos = pos;
        drawTerrain(carPos);
        updateInfo();
        await new Promise(r => setTimeout(r, 400));
      }
      winGame();
    }

    function updateInfo() {
      infoDiv.innerHTML = `
        <p>⛽ Fuel Pickups at: ${fuelPoints.join(', ')}</p>
        <p>⚡ Energy: ${energy}</p>
        <p>🚗 Position: ${carPos}</p>
        <p>🎮 Press ←/→ or use buttons, R to restart</p>
      `;
    }

    function moveCar(toPos) {
      if (toPos < 0 || toPos > GOAL || gameOver) return;

      const cost = terrainCost(carPos, toPos);
      if (energy < cost) {
        loseGame();
        return;
      }

      energy -= cost;
      if (fuelPoints.includes(toPos)) {
        energy += 3;
        fuelPoints = fuelPoints.filter(f => f !== toPos);
      }

      carPos = toPos;
      drawTerrain(carPos);
      updateInfo();

      if (carPos === GOAL) winGame();
    }

    function moveLeft() {
      manualMode = true;
      moveCar(carPos - 1);
    }

    function moveRight() {
      manualMode = true;
      moveCar(carPos + 1);
    }

    function winGame() {
      gameOver = true;
      infoDiv.innerHTML += `<p>🎉 Reached the Top!</p>`;
    }

    function loseGame() {
      gameOver = true;
      infoDiv.innerHTML += `<p>💀 Out of energy!</p>`;
    }

    function toggleNightMode() {
      document.body.classList.toggle('night');
    }

    function restartGame() {
      heights = Array.from({ length: WIDTH }, () => Math.floor(Math.random() * MAX_HEIGHT + 1));
      fuelPoints = generateFuelPoints();
      carPos = 0;
      energy = initialEnergy;
      manualMode = false;
      gameOver = false;

      drawTerrain(carPos);
      updateInfo();

      const path = bestFirstSearch(carPos, GOAL, energy);
      if (path) {
        infoDiv.innerHTML += `<p>✅ AI Path: ${path.join(' → ')}</p>`;
        animatePath(path);
      } else {
        infoDiv.innerHTML += `<p>❌ No path found.</p>`;
      }
    }

    document.addEventListener('keydown', (e) => {
      if (gameOver) return;

      if (e.key === 'ArrowRight') moveRight();
      else if (e.key === 'ArrowLeft') moveLeft();
      else if (e.key.toLowerCase() === 'r') restartGame();
    });

    restartGame();
  </script>
</body>
</html>
