<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Block Blast Stable</title>

<style>
body {
    margin:0;
    background:#0f172a;
    color:white;
    text-align:center;
    font-family:sans-serif;
}

#board {
    display:grid;
    grid-template-columns:repeat(8,1fr);
    gap:5px;
    width:90vw;
    max-width:400px;
    margin:auto;
}

.cell {
    aspect-ratio:1;
    background:#1e293b;
    border-radius:6px;
}

#pieces {
    display:flex;
    justify-content:center;
    gap:10px;
    margin-top:20px;
}

.piece {
    padding:10px;
    background:#334155;
    border-radius:10px;
    position:absolute;
}

.block {
    width:18px;
    height:18px;
    margin:2px;
    display:inline-block;
    border-radius:4px;
}
</style>
</head>
<body>

<h2>🧩 Block Blast (Stable)</h2>
Score: <span id="score">0</span>

<div id="board"></div>
<div id="pieces" style="position:relative;height:120px;"></div>

<script>
const size=8;
let board=Array(size).fill().map(()=>Array(size).fill(null));
let score=0;

const shapes=[
 [[1,1],[1,1]],
 [[1,1,1]],
 [[1],[1],[1]],
 [[1,0],[1,1]]
];

function randColor(){
    return `hsl(${Math.random()*360},80%,60%)`;
}

function drawBoard(){
    const b=document.getElementById("board");
    b.innerHTML="";
    for(let y=0;y<size;y++){
        for(let x=0;x<size;x++){
            let d=document.createElement("div");
            d.className="cell";
            if(board[y][x]) d.style.background=board[y][x];
            b.appendChild(d);
        }
    }
}

function spawnPieces(){
    const container=document.getElementById("pieces");
    container.innerHTML="";

    for(let i=0;i<3;i++){
        let shape=shapes[Math.floor(Math.random()*shapes.length)];
        let color=randColor();

        let div=document.createElement("div");
        div.className="piece";

        shape.forEach(row=>{
            let r=document.createElement("div");
            row.forEach(c=>{
                let b=document.createElement("div");
                b.className="block";
                b.style.background=color;
                b.style.visibility=c?"visible":"hidden";
                r.appendChild(b);
            });
            div.appendChild(r);
        });

        container.appendChild(div);
        enableDrag(div,shape,color);
    }
}

// 🔥 POINTER EVENT (FIX BUG)
function enableDrag(el,shape,color){
    let offsetX, offsetY;

    el.onpointerdown = (e)=>{
        el.setPointerCapture(e.pointerId);

        offsetX = e.offsetX;
        offsetY = e.offsetY;

        function move(e){
            el.style.left = (e.pageX - offsetX) + "px";
            el.style.top  = (e.pageY - offsetY) + "px";
        }

        function up(e){
            el.releasePointerCapture(e.pointerId);

            document.removeEventListener("pointermove",move);
            document.removeEventListener("pointerup",up);

            let rect=document.getElementById("board").getBoundingClientRect();
            let cellSize=rect.width/8;

            let x=Math.floor((e.clientX-rect.left)/cellSize);
            let y=Math.floor((e.clientY-rect.top)/cellSize);

            if(canPlace(shape,x,y)){
                place(shape,x,y,color);
                checkLines();
                score+=10;
                document.getElementById("score").innerText=score;
                drawBoard();
                spawnPieces();
            }

            el.remove();
        }

        document.addEventListener("pointermove",move);
        document.addEventListener("pointerup",up);
    };
}

function canPlace(shape,x,y){
    for(let i=0;i<shape.length;i++){
        for(let j=0;j<shape[i].length;j++){
            if(shape[i][j]){
                let ny=y+i,nx=x+j;
                if(ny<0||nx<0||ny>=size||nx>=size||board[ny][nx]) return false;
            }
        }
    }
    return true;
}

function place(shape,x,y,color){
    shape.forEach((r,i)=>{
        r.forEach((c,j)=>{
            if(c) board[y+i][x+j]=color;
        });
    });
}

function checkLines(){
    for(let y=0;y<size;y++){
        if(board[y].every(c=>c)){
            board[y]=Array(size).fill(null);
            score+=100;
        }
    }

    for(let x=0;x<size;x++){
        let full=true;
        for(let y=0;y<size;y++){
            if(!board[y][x]) full=false;
        }
        if(full){
            for(let y=0;y<size;y++) board[y][x]=null;
            score+=100;
        }
    }
}

// INIT
drawBoard();
spawnPieces();
</script>

</body>
</html>
