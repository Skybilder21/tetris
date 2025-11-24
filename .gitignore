<!doctype html>
<html lang="de">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Tetris — Einfaches Spiel</title>
  <style>
    :root{--cell:24px;--cols:10;--rows:20}
    body{font-family:Inter,Segoe UI,Arial;margin:0;display:flex;gap:20px;align-items:flex-start;padding:20px;background:#0b1220;color:#e6eef8}
    .board{width:calc(var(--cell) * var(--cols));height:calc(var(--cell) * var(--rows));background:#071022;border-radius:8px;padding:6px;box-shadow:0 6px 30px rgba(2,6,23,.6);position:relative}
    canvas{display:block;background:linear-gradient(180deg,#071022,#05101b);width:100%;height:100%;image-rendering:pixelated;border-radius:6px}
    .sidebar{display:flex;flex-direction:column;gap:12px}
    .panel{background:rgba(255,255,255,0.04);padding:12px;border-radius:8px;width:220px}
    h1{font-size:18px;margin:0 0 6px}
    .small{font-size:12px;color:#a9bbd6}
    .big{font-size:22px;font-weight:700}
    button{background:#15304b;color:#e6eef8;border:0;padding:8px;border-radius:8px;cursor:pointer}
    .controls{display:flex;flex-direction:column;gap:6px;font-size:13px}
    .next{display:grid;grid-template-columns:repeat(4,var(--cell));grid-auto-rows:var(--cell);gap:2px}
    .center{display:flex;align-items:center;justify-content:center}
    footer{font-size:12px;color:#88a0c0;margin-top:6px}
  </style>
</head>
<body>
  <div class="board" id="board">
    <canvas id="playfield" width="240" height="480"></canvas>
  </div>

  <div class="sidebar">
    <div class="panel">
      <h1>Tetris</h1>
      <div class="small">Punkte</div>
      <div class="big" id="score">0</div>
      <div style="height:8px"></div>
      <div class="small">Level</div>
      <div class="big" id="level">1</div>
      <div style="height:8px"></div>
      <div class="small">Zeilen</div>
      <div class="big" id="lines">0</div>
      <footer>Steuerung: ← → = bewegen, ↑ = rotieren, ↓ = soft drop, Space = Hard Drop, P = Pause</footer>
    </div>

    <div class="panel">
      <div class="small">Nächstes</div>
      <canvas id="next" width="96" height="96" style="background:transparent;border-radius:6px"></canvas>
    </div>

    <div class="panel controls">
      <button id="btn-start">Start / Neu</button>
      <button id="btn-pause">Pause (P)</button>
      <div class="small">Tipps</div>
      <div style="font-size:13px;color:#bcd0ea">Baue möglichst flache, gleichmäßige Stapel. Nutze Rotationen früh. Hard Drop spart Zeit und bringt Punkte.</div>
    </div>
  </div>

<script>
(() => {
  const COLS = 10, ROWS = 20, CELL = 24;
  const canvas = document.getElementById('playfield');
  const ctx = canvas.getContext('2d');
  const nextCanvas = document.getElementById('next');
  const nctx = nextCanvas.getContext('2d');

  canvas.width = COLS * CELL; canvas.height = ROWS * CELL;
  nextCanvas.width = 4 * CELL; nextCanvas.height = 4 * CELL;

  const colors = {
    I: '#31d6f6', J: '#2f6fe6', L: '#f19c2f', O: '#f6d633', S: '#4fe65a', T: '#c84fe6', Z: '#e64f4f'
  };

  const SHAPES = {
    I: [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]],
    J: [[1,0,0],[1,1,1],[0,0,0]],
    L: [[0,0,1],[1,1,1],[0,0,0]],
    O: [[1,1],[1,1]],
    S: [[0,1,1],[1,1,0],[0,0,0]],
    T: [[0,1,0],[1,1,1],[0,0,0]],
    Z: [[1,1,0],[0,1,1],[0,0,0]]
  };

  function createEmpty(){
    const g=[]; for(let r=0;r<ROWS;r++){ g.push(new Array(COLS).fill(0)); } return g;
  }

  let grid = createEmpty();
  let score=0, level=1, lines=0;
  let dropInterval = 800; // ms
  let lastDrop = 0;
  let current = null;
  let next = null;
  let gameOver=false; let paused=false;

  function randPiece(){
    const keys = Object.keys(SHAPES);
    const k = keys[Math.floor(Math.random()*keys.length)];
    return {type:k, matrix:SHAPES[k].map(r=>r.slice()), x: Math.floor((COLS - SHAPES[k][0].length)/2), y: 0 };
  }

  function rotate(matrix){
    const N = matrix.length; const res = Array.from({length:N},()=>Array(N).fill(0));
    for(let r=0;r<N;r++) for(let c=0;c<N;c++) res[c][N-1-r]=matrix[r][c];
    return res;
  }

  function collide(grid, piece){
    const m = piece.matrix; for(let r=0;r<m.length;r++){
      for(let c=0;c<m[r].length;c++){
        if(m[r][c]){
          const x = piece.x + c; const y = piece.y + r;
          if(x<0 || x>=COLS || y>=ROWS) return true;
          if(y>=0 && grid[y][x]) return true;
        }
      }
    }
    return false;
  }

  function merge(grid, piece){
    const m = piece.matrix; for(let r=0;r<m.length;r++){
      for(let c=0;c<m[r].length;c++){
        if(m[r][c]){
          const x = piece.x + c; const y = piece.y + r;
          if(y>=0) grid[y][x] = piece.type;
        }
      }
    }
  }

  function clearLines(){
    let cleared = 0;
    outer: for(let r=ROWS-1;r>=0;r--){
      for(let c=0;c<COLS;c++) if(!grid[r][c]) continue outer;
      // full line
      grid.splice(r,1); grid.unshift(new Array(COLS).fill(0)); cleared++; r++; // re-check same row index
    }
    if(cleared>0){
      lines += cleared;
      score += [0,100,300,700,1500][cleared];
      level = Math.floor(lines/10) + 1;
      dropInterval = Math.max(100, 800 - (level-1)*60);
    }
  }

  function spawn(){
    current = next || randPiece(); next = randPiece(); current.x = Math.floor((COLS - current.matrix[0].length)/2); current.y = -1;
    if(collide(grid,current)){
      gameOver=true; paused=true;
    }
  }

  function hardDrop(){
    while(!collide(grid, {...current, y: current.y+1})) current.y++;
    lockPiece(); score += 2 * (ROWS - current.y); // reward
  }

  function lockPiece(){
    merge(grid,current); clearLines(); spawn();
  }

  function step(time){
    if(gameOver){ draw(); return; }
    if(!paused){
      if(!lastDrop) lastDrop = time;
      const delta = time - lastDrop;
      if(delta > dropInterval){
        lastDrop = time;
        current.y++;
        if(collide(grid,current)){
          current.y--;
          lockPiece();
        }
      }
    }
    draw();
    requestAnimationFrame(step);
  }

  function drawCell(ctx,x,y,fill){
    const pad = 1;
    ctx.fillStyle = '#03111b';
    ctx.fillRect(x*CELL+pad,y*CELL+pad,CELL-2*pad,CELL-2*pad);
    if(!fill) return;
    ctx.fillStyle = colors[fill] || '#888';
    ctx.fillRect(x*CELL+3,y*CELL+3,CELL-6,CELL-6);
    // highlight
    ctx.strokeStyle = 'rgba(255,255,255,0.06)';
    ctx.strokeRect(x*CELL+3,y*CELL+3,CELL-6,CELL-6);
  }

  function draw(){
    // background
    ctx.clearRect(0,0,canvas.width,canvas.height);
    // grid
    for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++) drawCell(ctx,c,r,grid[r][c]);
    // current piece
    if(current){
      const m = current.matrix; for(let r=0;r<m.length;r++) for(let c=0;c<m[r].length;c++){
        if(m[r][c]){
          const x = current.x + c, y = current.y + r;
          if(y>=0) drawCell(ctx,x,y,current.type);
        }
      }
    }
    // update HUD
    document.getElementById('score').textContent = score;
    document.getElementById('level').textContent = level;
    document.getElementById('lines').textContent = lines;

    // next
    nctx.clearRect(0,0,nextCanvas.width,nextCanvas.height);
    nctx.fillStyle = 'transparent'; nctx.fillRect(0,0,nextCanvas.width,nextCanvas.height);
    const nm = next.matrix; const offsetX = Math.floor((4 - nm[0].length)/2);
    const offsetY = Math.floor((4 - nm.length)/2);
    for(let r=0;r<nm.length;r++) for(let c=0;c<nm[r].length;c++){
      if(nm[r][c]){
        const x = offsetX + c, y = offsetY + r;
        // draw tiny cell
        nctx.fillStyle = colors[next.type] || '#aaa';
        nctx.fillRect(x*CELL+3,y*CELL+3,CELL-6,CELL-6);
        nctx.strokeStyle='rgba(0,0,0,0.1)'; nctx.strokeRect(x*CELL+3,y*CELL+3,CELL-6,CELL-6);
      }
    }

    if(paused && !gameOver){
      ctx.fillStyle = 'rgba(2,8,20,0.6)'; ctx.fillRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = '#dbeafe'; ctx.font='22px Inter,Arial'; ctx.textAlign='center';
      ctx.fillText('PAUSE', canvas.width/2, canvas.height/2);
    }
    if(gameOver){
      ctx.fillStyle = 'rgba(0,0,0,0.6)'; ctx.fillRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = '#ffdede'; ctx.font='20px Inter,Arial'; ctx.textAlign='center';
      ctx.fillText('GAME OVER', canvas.width/2, canvas.height/2-6);
      ctx.fillStyle = '#e6eef8'; ctx.font='14px Inter,Arial';
      ctx.fillText('Drücke Start / Neu', canvas.width/2, canvas.height/2 + 18);
    }
  }

  // input
  document.addEventListener('keydown', e => {
    if(gameOver) return;
    if(e.key === 'ArrowLeft'){ current.x--; if(collide(grid,current)) current.x++; }
    else if(e.key === 'ArrowRight'){ current.x++; if(collide(grid,current)) current.x--; }
    else if(e.key === 'ArrowDown'){ current.y++; if(collide(grid,current)){ current.y--; lockPiece(); lastDrop = performance.now(); } }
    else if(e.key === 'ArrowUp'){ const orig = current.matrix; current.matrix = rotate(current.matrix); if(collide(grid,current)) current.matrix = orig; }
    else if(e.code === 'Space'){ hardDrop(); lastDrop = performance.now(); }
    else if(e.key.toLowerCase() === 'p'){ paused = !paused; }
  });

  document.getElementById('btn-start').addEventListener('click', ()=>{
    grid = createEmpty(); score=0; level=1; lines=0; dropInterval=800; lastDrop=0; gameOver=false; paused=false; next = randPiece(); spawn(); requestAnimationFrame(step);
  });
  document.getElementById('btn-pause').addEventListener('click', ()=>{ paused = !paused; });

  // start automatically
  next = randPiece(); spawn(); requestAnimationFrame(step);
})();
</script>
</body>
</html>
