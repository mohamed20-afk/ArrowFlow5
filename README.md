<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Arrows - Puzzle Escape</title>

<style>

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    -webkit-tap-highlight-color:transparent;
}

:root{
    --bg:#0b0b16;
    --panel:#131322;
    --cell:#1b1b2d;
    --line:#2b2b46;

    --cyan:#00f3ff;
    --coral:#ff3860;
    --purple:#7b61ff;

    --text:#f2f5ff;
}

body{
    font-family:Inter,Segoe UI,sans-serif;
    background:
    radial-gradient(circle at top,
    rgba(0,243,255,.08),
    transparent 30%),

    radial-gradient(circle at bottom,
    rgba(255,56,96,.08),
    transparent 40%),

    var(--bg);

    color:var(--text);

    min-height:100vh;

    display:flex;
    justify-content:center;
    align-items:center;

    overflow:hidden;
    padding:20px;
}

.app{
    width:min(95vw,520px);

    display:flex;
    flex-direction:column;
    align-items:center;

    gap:22px;
}

.topbar{
    width:100%;
    display:flex;
    justify-content:space-between;
    align-items:center;
}

.title{
    display:flex;
    flex-direction:column;
}

.title h1{
    font-size:2rem;
    letter-spacing:2px;
    font-weight:900;
}

.title span{
    color:var(--cyan);
    opacity:.8;
}

.level{
    padding:12px 18px;
    border-radius:16px;

    background:rgba(255,255,255,.04);

    border:1px solid rgba(255,255,255,.08);

    box-shadow:
    0 0 25px rgba(0,243,255,.08);

    font-weight:700;
}

.board{
    position:relative;

    width:min(90vw,420px);
    height:min(90vw,420px);

    display:grid;

    grid-template-columns:repeat(4,1fr);
    grid-template-rows:repeat(4,1fr);

    gap:8px;

    padding:8px;

    border-radius:28px;

    background:rgba(255,255,255,.03);

    border:1px solid rgba(255,255,255,.08);

    box-shadow:
    0 0 40px rgba(0,243,255,.08),
    inset 0 0 30px rgba(255,255,255,.03);

    overflow:hidden;
}

.cell{
    background:var(--cell);
    border-radius:18px;
    border:1px solid rgba(255,255,255,.03);
}

.block{
    position:absolute;

    display:flex;
    justify-content:center;
    align-items:center;

    font-size:2rem;
    font-weight:900;

    border-radius:18px;

    color:white;

    cursor:pointer;
    user-select:none;

    transition:
    left .25s cubic-bezier(.2,.9,.2,1),
    top .25s cubic-bezier(.2,.9,.2,1),
    opacity .35s ease,
    transform .12s ease;

    box-shadow:
    0 0 25px rgba(0,243,255,.25),
    inset 0 0 12px rgba(255,255,255,.08);
}

.block:hover{
    transform:scale(1.03);
}

.block.up{
    background:linear-gradient(135deg,#00f3ff,#498eff);
}

.block.down{
    background:linear-gradient(135deg,#ff3860,#ff6b6b);
}

.block.left{
    background:linear-gradient(135deg,#00f3ff,#00c6b2);
}

.block.right{
    background:linear-gradient(135deg,#7b61ff,#9f7cff);
}

.controls{
    display:flex;
    gap:12px;
    flex-wrap:wrap;
    justify-content:center;
}

button{
    border:none;
    cursor:pointer;

    padding:14px 22px;

    border-radius:16px;

    color:white;
    font-weight:700;
    font-size:1rem;

    background:
    linear-gradient(135deg,
    var(--coral),
    #ff6b6b);

    transition:.25s;

    box-shadow:
    0 0 25px rgba(255,56,96,.18);
}

button:hover{
    transform:translateY(-2px);
}

.overlay{
    position:fixed;
    inset:0;

    background:rgba(0,0,0,.6);

    display:flex;
    justify-content:center;
    align-items:center;

    opacity:0;
    pointer-events:none;

    transition:.35s ease;

    z-index:100;
}

.overlay.show{
    opacity:1;
}

.victory{
    padding:40px 60px;

    border-radius:24px;

    background:rgba(20,20,35,.95);

    border:1px solid rgba(0,243,255,.2);

    box-shadow:
    0 0 50px rgba(0,243,255,.2);

    text-align:center;

    transform:scale(.85);

    transition:.35s ease;
}

.overlay.show .victory{
    transform:scale(1);
}

.victory h2{
    font-size:2rem;
    color:var(--cyan);
    margin-bottom:10px;
}

.victory p{
    opacity:.8;
}

@media(max-width:500px){

    .title h1{
        font-size:1.5rem;
    }

    .block{
        font-size:1.5rem;
    }

}

</style>
</head>

<body>

<div class="app">

    <div class="topbar">

        <div class="title">
            <h1>ARROWS</h1>
            <span>Puzzle Escape</span>
        </div>

        <div class="level" id="levelText">
            Level 1 / 5000
        </div>

    </div>

    <div class="board" id="board"></div>

    <div class="controls">
        <button id="prevBtn">Previous</button>
        <button id="resetBtn">Reset</button>
        <button id="nextBtn">Next</button>
    </div>

</div>

<div class="overlay" id="overlay">
    <div class="victory">
        <h2>LEVEL CLEARED</h2>
        <p>Preparing next challenge...</p>
    </div>
</div>

<script>

// ======================================================
// CONFIG
// ======================================================

const GRID = 4;
const MAX_LEVEL = 5000;

const board = document.getElementById("board");
const levelText = document.getElementById("levelText");
const overlay = document.getElementById("overlay");

let currentLevel =
Number(localStorage.getItem("arrowsLevel")) || 1;

let blocks = [];
let originalGrid = null;


// ======================================================
// SEEDED RNG
// ======================================================

function mulberry32(a){

    return function(){

        let t = a += 0x6D2B79F5;

        t = Math.imul(t ^ t >>> 15, t | 1);

        t ^= t + Math.imul(t ^ t >>> 7, t | 61);

        return ((t ^ t >>> 14) >>> 0) / 4294967296;
    }
}


// ======================================================
// UTILITIES
// ======================================================

function cloneGrid(grid){

    return grid.map(r=>[...r]);

}

function emptyGrid(){

    return Array.from(
        {length:GRID},
        ()=>Array(GRID).fill('E')
    );

}

function boardSize(){

    return (board.clientWidth - 40) / 4;

}

function pos(row,col){

    const size = boardSize();
    const gap = 8;

    return {
        x:8 + col*(size+gap),
        y:8 + row*(size+gap)
    };
}

function symbol(d){

    return {
        U:'↑',
        D:'↓',
        L:'←',
        R:'→'
    }[d];
}


// ======================================================
// BOARD
// ======================================================

function createBoard(){

    board.innerHTML = "";

    for(let i=0;i<16;i++){

        const c = document.createElement("div");
        c.className = "cell";

        board.appendChild(c);
    }
}


// ======================================================
// MOVEMENT
// ======================================================

function simulateMove(grid,row,col){

    const dir = grid[row][col];

    if(dir === 'E') return null;

    let dr=0;
    let dc=0;

    if(dir==='U') dr=-1;
    if(dir==='D') dr=1;
    if(dir==='L') dc=-1;
    if(dir==='R') dc=1;

    let r=row;
    let c=col;

    while(true){

        const nr = r + dr;
        const nc = c + dc;

        // ESCAPE
        if(
            nr<0 || nr>=GRID ||
            nc<0 || nc>=GRID
        ){

            const newGrid = cloneGrid(grid);

            newGrid[row][col]='E';

            return newGrid;
        }

        // HIT BLOCK
        if(grid[nr][nc] !== 'E'){

            if(r===row && c===col){
                return null;
            }

            const newGrid = cloneGrid(grid);

            newGrid[row][col]='E';
            newGrid[r][c]=dir;

            return newGrid;
        }

        r=nr;
        c=nc;
    }
}


// ======================================================
// SOLVER (DFS)
// GUARANTEES SOLVABILITY
// ======================================================

function serialize(grid){

    return grid.flat().join('');

}

function isSolved(grid){

    for(let r=0;r<GRID;r++){

        for(let c=0;c<GRID;c++){

            if(grid[r][c] !== 'E') return false;
        }
    }

    return true;
}

function solve(grid,visited=new Set()){

    const key = serialize(grid);

    if(visited.has(key)) return false;

    visited.add(key);

    if(isSolved(grid)) return true;

    for(let r=0;r<GRID;r++){

        for(let c=0;c<GRID;c++){

            if(grid[r][c] === 'E') continue;

            const next = simulateMove(grid,r,c);

            if(next && solve(next,visited)){
                return true;
            }
        }
    }

    return false;
}


// ======================================================
// LEVEL GENERATOR
// ======================================================

function generateLevel(level){

    const rng = mulberry32(level * 99991);

    while(true){

        const grid = emptyGrid();

        // ==================================================
        // TUTORIAL LEVELS 1-10
        // ==================================================

        if(level <= 10){

            const count = 2 + Math.floor(rng()*2);

            let placed = 0;

            while(placed < count){

                const r = Math.floor(rng()*4);
                const c = Math.floor(rng()*4);

                if(grid[r][c] !== 'E') continue;

                const easyDirs = [];

                if(r===0) easyDirs.push('U');
                if(r===3) easyDirs.push('D');
                if(c===0) easyDirs.push('L');
                if(c===3) easyDirs.push('R');

                if(!easyDirs.length) continue;

                grid[r][c] =
                easyDirs[
                    Math.floor(rng()*easyDirs.length)
                ];

                placed++;
            }
        }

        // ==================================================
        // EXTREME MODE 11-5000
        // ==================================================

        else{

            const count =
            10 + Math.floor(rng()*5);

            const cells = [];

            for(let r=0;r<4;r++){

                for(let c=0;c<4;c++){

                    cells.push([r,c]);
                }
            }

            // shuffle
            cells.sort(()=>rng()-0.5);

            const dirs = ['U','D','L','R'];

            for(let i=0;i<count;i++){

                const [r,c] = cells[i];

                // heavily interlocking logic

                let bestDir = null;
                let bestScore = -999;

                for(const d of dirs){

                    let score = 0;

                    if(d==='U' && r>0) score++;
                    if(d==='D' && r<3) score++;
                    if(d==='L' && c>0) score++;
                    if(d==='R' && c<3) score++;

                    // prefer inward arrows
                    if(r===0 && d==='D') score+=3;
                    if(r===3 && d==='U') score+=3;
                    if(c===0 && d==='R') score+=3;
                    if(c===3 && d==='L') score+=3;

                    // random variation
                    score += rng()*2;

                    if(score > bestScore){

                        bestScore = score;
                        bestDir = d;
                    }
                }

                grid[r][c] = bestDir;
            }
        }

        // ==================================================
        // VALIDATE SOLVABILITY
        // ==================================================

        if(solve(grid)){

            return grid;
        }
    }
}


// ======================================================
// CREATE BLOCKS
// ======================================================

function createBlock(row,col,dir){

    const el = document.createElement("div");

    el.className =
    `block ${
        dir==='U'?'up':
        dir==='D'?'down':
        dir==='L'?'left':'right'
    }`;

    el.innerHTML = symbol(dir);

    const size = boardSize();

    el.style.width = `${size}px`;
    el.style.height = `${size}px`;

    const p = pos(row,col);

    el.style.left = `${p.x}px`;
    el.style.top = `${p.y}px`;

    board.appendChild(el);

    const obj = {
        row,
        col,
        dir,
        el,
        moving:false
    };

    el.addEventListener("click",()=>move(obj));

    blocks.push(obj);
}


// ======================================================
// LOAD LEVEL
// ======================================================

function loadLevel(level){

    currentLevel =
    Math.max(1,Math.min(MAX_LEVEL,level));

    localStorage.setItem(
        "arrowsLevel",
        currentLevel
    );

    levelText.innerText =
    `Level ${currentLevel} / 5000`;

    createBoard();

    blocks = [];

    originalGrid =
    generateLevel(currentLevel);

    for(let r=0;r<4;r++){

        for(let c=0;c<4;c++){

            const v = originalGrid[r][c];

            if(v !== 'E'){

                createBlock(r,c,v);
            }
        }
    }
}


// ======================================================
// HELPERS
// ======================================================

function getBlock(row,col,ignore=null){

    return blocks.find(b=>

        b!==ignore &&
        b.row===row &&
        b.col===col
    );
}


// ======================================================
// GAME MOVE
// ======================================================

function move(block){

    if(block.moving) return;

    block.moving = true;

    let dr=0;
    let dc=0;

    if(block.dir==='U') dr=-1;
    if(block.dir==='D') dr=1;
    if(block.dir==='L') dc=-1;
    if(block.dir==='R') dc=1;

    let r = block.row;
    let c = block.col;

    while(true){

        const nr = r + dr;
        const nc = c + dc;

        // ESCAPE
        if(
            nr<0 || nr>=4 ||
            nc<0 || nc>=4
        ){

            animateEscape(block,dr,dc,r,c);
            return;
        }

        // HIT BLOCK
        if(getBlock(nr,nc,block)){

            if(r===block.row && c===block.col){

                block.moving = false;
                return;
            }

            animateMove(block,r,c);
            return;
        }

        r=nr;
        c=nc;
    }
}


// ======================================================
// ANIMATIONS
// ======================================================

function animateMove(block,row,col){

    block.row = row;
    block.col = col;

    const p = pos(row,col);

    block.el.style.left = `${p.x}px`;
    block.el.style.top = `${p.y}px`;

    setTimeout(()=>{

        block.moving = false;

    },250);
}

function animateEscape(block,dr,dc,row,col){

    const size = boardSize();

    const p = pos(row,col);

    block.el.style.left =
    `${p.x + dc*(size+90)}px`;

    block.el.style.top =
    `${p.y + dr*(size+90)}px`;

    block.el.style.opacity = "0";

    setTimeout(()=>{

        block.el.remove();

        blocks = blocks.filter(b=>b!==block);

        checkVictory();

    },350);
}


// ======================================================
// WIN
// ======================================================

function checkVictory(){

    if(blocks.length===0){

        overlay.classList.add("show");

        setTimeout(()=>{

            overlay.classList.remove("show");

            if(currentLevel < MAX_LEVEL){

                loadLevel(currentLevel+1);
            }

        },1000);
    }
}


// ======================================================
// RESET
// ======================================================

function resetLevel(){

    createBoard();

    blocks = [];

    for(let r=0;r<4;r++){

        for(let c=0;c<4;c++){

            const v = originalGrid[r][c];

            if(v !== 'E'){

                createBlock(r,c,v);
            }
        }
    }
}


// ======================================================
// BUTTONS
// ======================================================

document.getElementById("nextBtn")
.onclick = ()=>{

    if(currentLevel < MAX_LEVEL){

        loadLevel(currentLevel+1);
    }
};

document.getElementById("prevBtn")
.onclick = ()=>{

    if(currentLevel > 1){

        loadLevel(currentLevel-1);
    }
};

document.getElementById("resetBtn")
.onclick = ()=>{

    resetLevel();
};


// ======================================================
// RESIZE
// ======================================================

window.addEventListener("resize",()=>{

    resetLevel();

});


// ======================================================
// INIT
// ======================================================

loadLevel(currentLevel);

</script>

</body>
</html>
