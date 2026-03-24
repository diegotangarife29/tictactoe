<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tic Tac Toe Pro</title>

<!-- ICONOS -->
<link href="https://unpkg.com/boxicons@2.1.4/css/boxicons.min.css" rel="stylesheet">

<style>
:root {
  --color-bg: #111;
  --color-cell: #222;
  --color-border: #555;
  --color-x: crimson;
  --color-o: royalblue;
  --color-win: #4caf50;
  --color-text: white;
  --color-btn: #333;
  --color-selected: #e6c34a;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

body {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: var(--color-bg);
  color: var(--color-text);
  font-family: Arial, sans-serif;
}

h1 { margin-bottom: 10px; }

/* TABLERO */
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 5px;
  width: min(90vw, 400px);
  height: min(90vw, 400px);
  transition: transform 0.2s;
}

.cell {
  background: var(--color-cell);
  border: 2px solid var(--color-border);
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 3rem;
  cursor: pointer;
  aspect-ratio: 1;
  transition: 0.2s;
}

.cell:hover { filter: brightness(1.2); }

.cell.previewX { background: var(--color-x); }
.cell.previewO { background: var(--color-o); }

.cell.taken { cursor: not-allowed; }

.winner {
  animation: glow 0.7s infinite alternate;
}

@keyframes glow {
  from { box-shadow: 0 0 10px #4caf50; }
  to { box-shadow: 0 0 25px #4caf50; }
}

/* MODOS */
.modeRow {
  display: grid;
  grid-template-columns: 50px 1fr 50px;
  gap: 30px;
  margin-bottom: 15px;
}

.modeIcon {
  display: flex;
  justify-content: center;
  align-items: center;
  background: #444;
  border: 2px solid black;
  cursor: pointer;
}

.modeIcon.active { background: var(--color-selected); }

/* TURNO */
#turnBox {
  display: flex;
  border: 2px solid black;
}

.turnSide {
  flex: 1;
  text-align: center;
  padding: 5px;
  background: #444;
}

.activeX { background: var(--color-x); }
.activeO { background: var(--color-o); }

/* MENSAJE */
.winnerJ {
  margin-top: 15px;
  font-size: 20px;
}

.hidden { display: none; }

/* BOTÓN */
.restartBtn {
  margin-top: 10px;
  padding: 10px;
  border: none;
  background: var(--color-btn);
  color: white;
  cursor: pointer;
}
</style>
</head>

<body>

<h1>Tic Tac Toe</h1>

<div class="modeRow">
  <div id="playerIcon" class="modeIcon"><i class="bx bxs-user"></i></div>

  <div id="turnBox">
    <div id="turnX" class="turnSide">X</div>
    <div id="turnO" class="turnSide">O</div>
  </div>

  <div id="botIcon" class="modeIcon"><i class="bx bxs-invader"></i></div>
</div>

<div class="container">
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
  <div class="cell"></div>
</div>

<p id="winnerMessage" class="winnerJ hidden"></p>
<button class="restartBtn" id="restartBtn">Reiniciar</button>

<script>
const cells = document.querySelectorAll(".cell");
let board = ["","","","","","","","",""];
let currentPlayer = "X";
let gameOver = false;
let gameMode = "bot";
let botThinking = false;

const winCombos = [
[0,1,2],[3,4,5],[6,7,8],
[0,3,6],[1,4,7],[2,5,8],
[0,4,8],[2,4,6]
];

function checkWinner(){
 for(let combo of winCombos){
   let [a,b,c]=combo;
   if(board[a] && board[a]===board[b] && board[a]===board[c]){
     gameOver=true;
     [a,b,c].forEach(i=>cells[i].classList.add("winner"));
     return board[a];
   }
 }
 if(!board.includes("")) return "Empate";
 return null;
}

cells.forEach((cell,i)=>{
 cell.addEventListener("mouseenter",()=>{
   if(board[i]==="" && !gameOver && !botThinking){
     cell.classList.add(currentPlayer==="X"?"previewX":"previewO");
   }
 });

 cell.addEventListener("mouseleave",()=>{
   cell.classList.remove("previewX","previewO");
 });

 cell.addEventListener("click",()=>{
   if(gameOver || board[i]!=="" || botThinking) return;

   playMove(i);
 });
});

function playMove(i){
 board[i]=currentPlayer;
 cells[i].textContent=currentPlayer;
 cells[i].classList.add("taken");
 cells[i].classList.remove("previewX","previewO");

 let winner=checkWinner();
 if(winner){ endGame(winner); return; }

 currentPlayer = currentPlayer==="X"?"O":"X";
 updateTurn();

 if(gameMode==="bot" && currentPlayer==="O"){
   botThinking=true;
   setTimeout(()=>{
     botMove();
     botThinking=false;
   },500);
 }
}

function botMove(){
 // ganar
 for(let [a,b,c] of winCombos){
   if(board[a]==="" && board[b]==="O" && board[c]==="O") return playMove(a);
   if(board[b]==="" && board[a]==="O" && board[c]==="O") return playMove(b);
   if(board[c]==="" && board[a]==="O" && board[b]==="O") return playMove(c);
 }

 // bloquear
 for(let [a,b,c] of winCombos){
   if(board[a]==="" && board[b]==="X" && board[c]==="X") return playMove(a);
   if(board[b]==="" && board[a]==="X" && board[c]==="X") return playMove(b);
   if(board[c]==="" && board[a]==="X" && board[b]==="X") return playMove(c);
 }

 // centro
 if(board[4]==="") return playMove(4);

 // esquinas
 let corners=[0,2,6,8].filter(i=>board[i]==="");
 if(corners.length) return playMove(corners[Math.floor(Math.random()*corners.length)]);

 // random
 let empty=board.map((v,i)=>v===""?i:null).filter(v=>v!==null);
 playMove(empty[Math.floor(Math.random()*empty.length)]);
}

function endGame(res){
 let msg=document.getElementById("winnerMessage");
 msg.textContent = res==="Empate"?"Empate":"Ganó "+res;
 msg.classList.remove("hidden");
}

function updateTurn(){
 document.getElementById("turnX").classList.toggle("activeX", currentPlayer==="X");
 document.getElementById("turnO").classList.toggle("activeO", currentPlayer==="O");
}

function resetGame(){
 board=["","","","","","","","",""];
 cells.forEach(c=>{
   c.textContent="";
   c.classList.remove("winner","taken","previewX","previewO");
 });
 gameOver=false;
 currentPlayer="X";
 document.getElementById("winnerMessage").classList.add("hidden");
 updateTurn();
}

document.getElementById("restartBtn").onclick=resetGame;

const playerIcon=document.getElementById("playerIcon");
const botIcon=document.getElementById("botIcon");

playerIcon.onclick=()=>{
 gameMode="player";
 playerIcon.classList.add("active");
 botIcon.classList.remove("active");
 resetGame();
};

botIcon.onclick=()=>{
 gameMode="bot";
 botIcon.classList.add("active");
 playerIcon.classList.remove("active");
 resetGame();
};

updateTurn();
botIcon.classList.add("active");
</script>

</body>
</html>
