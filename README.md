<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Block Animator v2</title>
<style>
  body {
    margin: 0;
    background: #1e1e1e;
    color: white;
    font-family: sans-serif;
    text-align: center;
  }

  canvas {
    background: white;
    margin-top: 10px;
    cursor: crosshair;
  }

  button {
    margin: 3px;
    padding: 6px;
  }

  .bar {
    margin-top: 10px;
  }
</style>
</head>
<body>

<h3>🎬 Block Animator v2</h3>

<div class="bar">
  <button onclick="prevFrame()">⬅</button>
  <button onclick="nextFrame()">➡</button>
  <button onclick="togglePlay()">▶/⏸</button>
  <button onclick="clearFrame()">🧹</button>
</div>

<div class="bar">
  <button onclick="setColor('#000000')">⚫</button>
  <button onclick="setColor('#ff0000')">🔴</button>
  <button onclick="setColor('#00ff00')">🟢</button>
  <button onclick="setColor('#0000ff')">🔵</button>
  <button onclick="setEraser()">🧽</button>
</div>

<div class="bar">
  <label>Speed:</label>
  <input type="range" min="50" max="600" value="200" id="speed">
  <span id="speedVal">200ms</span>
  <span id="info"></span>
</div>

<canvas id="c"></canvas>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");

const size = 20;
const cols = 25;
const rows = 15;

canvas.width = cols * size;
canvas.height = rows * size;

// ---------- FRAMES ----------
let frames = [];
let current = 0;

function emptyFrame() {
  return Array.from({ length: rows }, () =>
    Array(cols).fill(null)
  );
}

frames.push(emptyFrame());
let frame = frames[current];

// ---------- COLOR / ERASER ----------
let currentColor = "#000000";
let eraserMode = false;

function setColor(color) {
  currentColor = color;
  eraserMode = false;
}

function setEraser() {
  eraserMode = true;
}

// ---------- SPEED ----------
let speed = 200;

document.getElementById("speed").addEventListener("input", (e) => {
  speed = parseInt(e.target.value);
  document.getElementById("speedVal").innerText = speed + "ms";

  if (playing) {
    clearInterval(interval);
    startPlay();
  }
});

// ---------- DRAW ----------
function drawGrid() {
  ctx.strokeStyle = "#ddd";

  for (let x = 0; x <= cols; x++) {
    ctx.beginPath();
    ctx.moveTo(x * size, 0);
    ctx.lineTo(x * size, canvas.height);
    ctx.stroke();
  }

  for (let y = 0; y <= rows; y++) {
    ctx.beginPath();
    ctx.moveTo(0, y * size);
    ctx.lineTo(canvas.width, y * size);
    ctx.stroke();
  }
}

function drawFrame(f) {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // 👻 onion skin (previous frame)
  if (current > 0) {
    let prev = frames[current - 1];

    for (let y = 0; y < rows; y++) {
      for (let x = 0; x < cols; x++) {
        if (prev[y][x]) {
          ctx.fillStyle = "rgba(0,0,0,0.15)";
          ctx.fillRect(x * size, y * size, size, size);
        }
      }
    }
  }

  // current frame
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < cols; x++) {
      if (f[y][x]) {
        ctx.fillStyle = f[y][x];
        ctx.fillRect(x * size, y * size, size, size);
      }
    }
  }

  drawGrid();
}

// ---------- DRAWING ----------
let drawing = false;

canvas.addEventListener("mousedown", () => drawing = true);
canvas.addEventListener("mouseup", () => drawing = false);
canvas.addEventListener("mouseleave", () => drawing = false);

canvas.addEventListener("mousemove", (e) => {
  if (!drawing) return;

  const r = canvas.getBoundingClientRect();
  const x = Math.floor((e.clientX - r.left) / size);
  const y = Math.floor((e.clientY - r.top) / size);

  if (x >= 0 && y >= 0 && x < cols && y < rows) {
    if (eraserMode) {
      frame[y][x] = null;
    } else {
      frame[y][x] = currentColor;
    }

    drawFrame(frame);
  }
});

// ---------- FRAMES ----------
function saveFrame() {
  frames[current] = JSON.parse(JSON.stringify(frame));
}

function loadFrame() {
  frame = frames[current];
  drawFrame(frame);
  updateInfo();
}

function nextFrame() {
  saveFrame();

  if (current === frames.length - 1) {
    frames.push(emptyFrame());
  }

  current++;
  loadFrame();
}

function prevFrame() {
  saveFrame();
  if (current > 0) current--;
  loadFrame();
}

function clearFrame() {
  frame = emptyFrame();
  frames[current] = frame;
  drawFrame(frame);
}

// ---------- PLAY ----------
let playing = false;
let interval = null;
let i = 0;

function startPlay() {
  saveFrame();
  playing = true;
  i = 0;

  interval = setInterval(() => {
    drawFrame(frames[i]);
    updateInfo(i);

    i++;
    if (i >= frames.length) i = 0;
  }, speed);
}

function togglePlay() {
  if (playing) {
    playing = false;
    clearInterval(interval);
    loadFrame();
  } else {
    startPlay();
  }
}

// ---------- INFO ----------
function updateInfo(index = current) {
  document.getElementById("info").innerText =
    ` Frame ${index + 1} / ${frames.length}`;
}

// ---------- INIT ----------
updateInfo();
drawFrame(frame);
</script>

</body>
</html>
