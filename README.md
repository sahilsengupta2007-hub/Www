
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>A Little Birthday Maze 💗</title>

<style>
  * { box-sizing: border-box; }

  body {
    margin: 0;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 18px;
    background: linear-gradient(135deg, #fff0f5, #fce4ec, #f3e5f5);
    font-family: "Trebuchet MS", sans-serif;
    color: #7b2948;
  }

  .game {
    width: min(100%, 600px);
    text-align: center;
    padding: 22px 14px;
    background: rgba(255,255,255,.82);
    border: 2px solid #f5bfd1;
    border-radius: 24px;
    box-shadow: 0 12px 35px #b84e7020;
  }

  h1 {
    margin: 0 0 8px;
    font-size: clamp(25px, 6vw, 36px);
  }

  .subtitle {
    margin: 0 0 18px;
    font-size: 14px;
    color: #a45170;
  }

  canvas {
    display: block;
    width: 100%;
    max-width: 504px;
    height: auto;
    margin: auto;
    border: 4px solid #f4b4cb;
    border-radius: 12px;
    background: #fff9fc;
    touch-action: none;
  }

  .controls {
    margin: 15px auto 8px;
    display: grid;
    grid-template-columns: repeat(3, 56px);
    justify-content: center;
    gap: 7px;
  }

  button, .upload {
    border: 0;
    border-radius: 12px;
    padding: 12px;
    background: #ec8eae;
    color: white;
    font-weight: bold;
    font-size: 17px;
    cursor: pointer;
    box-shadow: 0 3px 0 #c76587;
  }

  button:active {
    transform: translateY(2px);
    box-shadow: 0 1px 0 #c76587;
  }

  .empty { visibility: hidden; }

  .upload {
    display: inline-block;
    margin: 10px 0;
    font-size: 13px;
    background: #b68ad4;
    box-shadow: 0 3px 0 #9164b0;
  }

  #photoInput { display: none; }

  #status {
    min-height: 24px;
    margin: 12px 0 4px;
    font-size: 14px;
    font-weight: bold;
  }

  .hint {
    font-size: 12px;
    line-height: 1.6;
    color: #a36a81;
  }

  #win {
    display: none;
    padding: 15px;
    margin-top: 12px;
    background: #fff0f5;
    border-radius: 16px;
  }

  #win h2 { margin: 0 0 8px; }

  #win button {
    margin-top: 10px;
    font-size: 14px;
  }
</style>
</head>

<body>
<div class="game">
  <h1>💗 A Little Birthday Maze 💗</h1>
  <p class="subtitle">
    A little journey, just for you. Find the pink heart at the end! 🌸
  </p>

  <canvas id="maze" width="504" height="504"></canvas>

  <div class="controls">
    <span class="empty"></span>
    <button onclick="move(0,-1)" aria-label="Move up">▲</button>
    <span class="empty"></span>

    <button onclick="move(-1,0)" aria-label="Move left">◀</button>
    <button onclick="move(0,1)" aria-label="Move down">▼</button>
    <button onclick="move(1,0)" aria-label="Move right">▶</button>
  </div>

  <label class="upload" for="photoInput">
    📷 Choose player photo
  </label>
  <input id="photoInput" type="file" accept="image/*">

  <p id="status">✨ Find your way to the heart!</p>

  <p class="hint">
    Use arrow keys or WASD on your keyboard.<br>
    On mobile, use the arrow buttons above.
  </p>

  <div id="win">
    <h2>🎉 You made it! 💖</h2>
    <p>Your little surprise is waiting for you...</p>
    <button onclick="openBirthday()">Open your birthday surprise 💌</button>
  </div>
</div>

<script>
const canvas = document.getElementById("maze");
const ctx = canvas.getContext("2d");
const statusText = document.getElementById("status");

const COLS = 21;
const ROWS = 21;
const CELL = canvas.width / COLS;

let grid = [];
let player = { x: 1, y: 1 };
let goal = { x: COLS - 2, y: ROWS - 2 };
let playerImage = new Image();
let photoLoaded = false;
let won = false;
let moves = 0;

// Optional default image: put player.jpg beside index.html.
playerImage.onload = () => {
  photoLoaded = true;
  draw();
};
playerImage.onerror = () => {
  photoLoaded = false;
  draw();
};
playerImage.src = "player.jpg";

// Generate a connected, single-level maze using depth-first search.
function generateMaze() {
  grid = Array.from(
    { length: ROWS },
    () => Array(COLS).fill(1)
  );

  const stack = [{ x: 1, y: 1 }];
  grid[1][1] = 0;

  const directions = [
    { x: 0, y: -2 },
    { x: 0, y: 2 },
    { x: -2, y: 0 },
    { x: 2, y: 0 }
  ];

  while (stack.length) {
    const current = stack[stack.length - 1];

    const choices = directions
      .map(d => ({
        x: current.x + d.x,
        y: current.y + d.y,
        dx: d.x,
        dy: d.y
      }))
      .filter(p =>
        p.x > 0 && p.x < COLS - 1 &&
        p.y > 0 && p.y < ROWS - 1 &&
        grid[p.y][p.x] === 1
      );

    if (!choices.length) {
      stack.pop();
      continue;
    }

    const next = choices[
      Math.floor(Math.random() * choices.length)
    ];

    grid[current.y + next.dy / 2]
        [current.x + next.dx / 2] = 0;

    grid[next.y][next.x] = 0;
    stack.push({ x: next.x, y: next.y });
  }

  // Start and finish are connected to the carved maze.
  grid[1][1] = 0;
  grid[goal.y][goal.x] = 0;

  // Create a small entrance and exit.
  player = { x: 1, y: 1 };
  won = false;
  moves = 0;

  document.getElementById("win").style.display = "none";
  statusText.textContent = "✨ Find your way to the heart!";

  draw();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Maze tiles.
  for (let y = 0; y < ROWS; y++) {
    for (let x = 0; x < COLS; x++) {
      const px = x * CELL;
      const py = y * CELL;

      if (grid[y][x] === 1) {
        ctx.fillStyle = "#e8a0ba";
        ctx.fillRect(px, py, CELL, CELL);

        ctx.fillStyle = "#f4c3d3";
        ctx.fillRect(px + 2, py + 2, CELL - 4, CELL - 4);
      } else {
        ctx.fillStyle = "#fff9fc";
        ctx.fillRect(px, py, CELL, CELL);
      }
    }
  }

  // Start marker.
  ctx.fillStyle = "#d5b0e8";
  ctx.beginPath();
  ctx.arc(
    (1 + 0.5) * CELL,
    (1 + 0.5) * CELL,
    CELL * 0.25,
    0, Math.PI * 2
  );
  ctx.fill();

  // Finish marker.
  const gx = (goal.x + 0.5) * CELL;
  const gy = (goal.y + 0.5) * CELL;

  ctx.font = `${CELL * 0.85}px sans-serif`;
  ctx.textAlign = "center";
  ctx.textBaseline = "middle";
  ctx.fillText("💗", gx, gy);

  // Circular player photo.
  const cx = (player.x + 0.5) * CELL;
  const cy = (player.y + 0.5) * CELL;
  const radius = CELL * 0.43;

  ctx.save();

  // Soft pink outline and shadow.
  ctx.shadowColor = "#d75b89";
  ctx.shadowBlur = 7;
  ctx.fillStyle = "#fff";
  ctx.beginPath();
  ctx.arc(cx, cy, radius + 2, 0, Math.PI * 2);
  ctx.fill();

  ctx.shadowBlur = 0;
  ctx.beginPath();
  ctx.arc(cx, cy, radius, 0, Math.PI * 2);
  ctx.clip();

  if (photoLoaded && playerImage.complete) {
    // Cover-crop the photo to keep the face centered.
    const iw = playerImage.naturalWidth;
    const ih = playerImage.naturalHeight;
    const side = Math.min(iw, ih);
    const sx = (iw - side) / 2;
    const sy = (ih - side) / 2;

    ctx.drawImage(
      playerImage,
      sx, sy, side, side,
      cx - radius, cy - radius,
      radius * 2, radius * 2
    );
  } else {
    // Fallback character if no photo has been selected.
    ctx.fillStyle = "#ffd6e5";
    ctx.fillRect(cx - radius, cy - radius, radius * 2, radius * 2);
    ctx.font = `${CELL * 0.8}px sans-serif`;
    ctx.fillStyle = "#a53c67";
    ctx.fillText("☺", cx, cy);
  }

  ctx.restore();

  // Pink border around the circular portrait.
  ctx.strokeStyle = "#d85f8b";
  ctx.lineWidth = 2;
  ctx.beginPath();
  ctx.arc(cx, cy, radius, 0, Math.PI * 2);
  ctx.stroke();
}

function move(dx, dy) {
  if (won) return;

  const nx = player.x + dx;
  const ny = player.y + dy;

  // Prevent movement through walls and outside the maze.
  if (
    nx < 0 || nx >= COLS ||
    ny < 0 || ny >= ROWS ||
    grid[ny][nx] === 1
  ) {
    return;
  }

  player.x = nx;
  player.y = ny;
  moves++;

  statusText.textContent = `💗 Steps taken: ${moves}`;
  draw();

  if (player.x === goal.x && player.y === goal.y) {
    won = true;
    statusText.textContent = "You found the birthday surprise! 🎉";
    document.getElementById("win").style.display = "block";

    // Brief pause so the player sees the finish before navigating.
    setTimeout(openBirthday, 1200);
  }
}

function openBirthday() {
  if (won) {
    window.location.href =
      "https://happybirthdaytohridi.carrd.co/";
  }
}

// Keyboard controls.
document.addEventListener("keydown", event => {
  const keys = {
    ArrowUp: [0, -1],
    w: [0, -1],
    W: [0, -1],
    ArrowDown: [0, 1],
    s: [0, 1],
    S: [0, 1],
    ArrowLeft: [-1, 0],
    a: [-1, 0],
    A: [-1, 0],
    ArrowRight: [1, 0],
    d: [1, 0],
    D: [1, 0]
  };

  if (keys[event.key]) {
    event.preventDefault();
    move(...keys[event.key]);
  }
});

// Load the chosen photo as the circular player portrait.
document.getElementById("photoInput").addEventListener(
  "change",
  event => {
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();

    reader.onload = e => {
      playerImage.onload = () => {
        photoLoaded = true;
        draw();
      };

      playerImage.src = e.target.result;
    };

    reader.readAsDataURL(file);
  }
);

generateMaze();
</script>
</body>
</html>
