<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Arcade Mini-Games</title>

<style>
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
}

body {
    background: #101426;
    color: white;
    text-align: center;
    padding: 25px;
}

h1 {
    color: #00e5ff;
    margin-bottom: 20px;
}

.container {
    max-width: 700px;
    margin: auto;
}

.games {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 12px;
    margin-bottom: 25px;
}

button {
    background: #273454;
    color: white;
    border: none;
    padding: 14px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 15px;
}

button:hover {
    background: #007bff;
}

.game-box {
    background: #1c2541;
    padding: 25px;
    border-radius: 15px;
    min-height: 250px;
}

.score {
    color: #00ffae;
    font-size: 22px;
    margin: 15px;
}

#gameArea {
    margin-top: 20px;
}

#gameArea button {
    margin: 5px;
}

input {
    padding: 12px;
    width: 100px;
    border-radius: 8px;
    border: none;
}

.hidden {
    display: none;
}

#result {
    margin-top: 20px;
    color: #ffd700;
    font-size: 20px;
}
</style>
</head>

<body>

<div class="container">

<h1>🎮 ARCADE MINI-GAMES</h1>

<div class="games">
    <button onclick="startGame('number')">🔢 Number Hunter</button>
    <button onclick="startGame('rps')">✊ Rock Paper Scissors</button>
    <button onclick="startGame('memory')">🧠 Memory Match</button>
    <button onclick="startGame('tic')">❌ Tic-Tac-Toe</button>
    <button onclick="startGame('reaction')">⚡ Reaction Rush</button>
</div>

<div class="game-box">
    <h2 id="gameTitle">Welcome to Arcade!</h2>

    <p id="instructions">
        Select a mini-game to begin playing.
    </p>

    <div class="score">
        Score: <span id="score">0</span>
    </div>

    <div id="gameArea"></div>

    <div id="result"></div>

    <button onclick="restartGame()">🔄 Restart</button>
    <button onclick="showMenu()">🏠 Main Menu</button>
</div>

</div>

<script>

let score = 0;
let currentGame = "";
let secretNumber;
let attempts;
let board = ["","","","","","","","",""];
let currentPlayer = "X";
let gameOver = false;
let reactionStart;
let reactionTimer;
let reactionTimeout;

function updateScore(points) {
    score += points;
    document.getElementById("score").textContent = score;
}

function startGame(game) {
    currentGame = game;
    score = 0;

    document.getElementById("score").textContent = 0;
    document.getElementById("result").textContent = "";

    const area = document.getElementById("gameArea");

    if(game === "number") {
        document.getElementById("gameTitle").textContent = "🔢 Number Hunter";
        document.getElementById("instructions").textContent =
            "Guess the secret number between 1 and 100. You have 7 attempts.";

        secretNumber = Math.floor(Math.random() * 100) + 1;
        attempts = 7;

        area.innerHTML = `
            <input type="number" id="guess" placeholder="Number">
            <button onclick="checkNumber()">Guess</button>
            <p id="hint">Attempts remaining: 7</p>
        `;
    }

    else if(game === "rps") {
        document.getElementById("gameTitle").textContent = "✊ Rock Paper Scissors";
        document.getElementById("instructions").textContent =
            "Choose your move and beat the computer.";

        area.innerHTML = `
            <button onclick="playRPS('Rock')">✊ Rock</button>
            <button onclick="playRPS('Paper')">✋ Paper</button>
            <button onclick="playRPS('Scissors')">✌️ Scissors</button>
        `;
    }

    else if(game === "memory") {
        document.getElementById("gameTitle").textContent = "🧠 Memory Match";
        document.getElementById("instructions").textContent =
            "Find all matching pairs of cards.";

        let cards = ["🍎","🍌","🍇","🍒","🍎","🍌","🍇","🍒"];
        cards.sort(() => Math.random() - 0.5);

        let first = null;
        let locked = false;
        let matched = 0;

        area.innerHTML = `
            <div style="display:grid;grid-template-columns:repeat(4,1fr);gap:10px">
            ${cards.map((card,i) =>
                `<button id="card${i}" onclick="flipCard(${i})">❓</button>`
            ).join("")}
            </div>
        `;

        window.memoryCards = cards;
        window.firstCard = null;
        window.lockedCards = false;
        window.matchedCards = [];
    }

    else if(game === "tic") {
        document.getElementById("gameTitle").textContent = "❌ Tic-Tac-Toe";
        document.getElementById("instructions").textContent =
            "Take turns with another player. Get three in a row to win.";

        board = ["","","","","","","","",""];
        currentPlayer = "X";
        gameOver = false;

        renderTic();
    }

    else if(game === "reaction") {
        document.getElementById("gameTitle").textContent = "⚡ Reaction Rush";
        document.getElementById("instructions").textContent =
            "Wait for the green signal, then click as quickly as possible.";

        area.innerHTML = `
            <button id="reactionButton" onclick="reactionClick()">
                START
            </button>
        `;

        document.getElementById("reactionButton").style.background = "#273454";
    }
}

function checkNumber() {
    let guess = Number(document.getElementById("guess").value);

    if(!guess || guess < 1 || guess > 100) {
        alert("Enter a number between 1 and 100.");
        return;
    }

    attempts--;

    if(guess === secretNumber) {
        updateScore(100);
        endGame("Correct! You found the number!");
    }
    else if(attempts === 0) {
        endGame("Game Over! The number was " + secretNumber);
    }
    else {
        document.getElementById("hint").textContent =
            (guess > secretNumber ? "Too high!" : "Too low!") +
            " Attempts remaining: " + attempts;
    }
}

function playRPS(player) {
    let choices = ["Rock","Paper","Scissors"];
    let computer = choices[Math.floor(Math.random()*3)];

    let message = "Computer chose " + computer + ". ";

    if(player === computer) {
        message += "It's a draw!";
        updateScore(10);
    }
    else if(
        (player === "Rock" && computer === "Scissors") ||
        (player === "Paper" && computer === "Rock") ||
        (player === "Scissors" && computer === "Paper")
    ) {
        message += "You win!";
        updateScore(20);
    }
    else {
        message += "You lose!";
    }

    document.getElementById("result").textContent = message;
}

function flipCard(index) {
    if(currentGame !== "memory" ||
       window.lockedCards ||
       window.matchedCards.includes(index)) return;

    let button = document.getElementById("card" + index);
    button.textContent = window.memoryCards[index];

    if(window.firstCard === null) {
        window.firstCard = index;
        return;
    }

    let first = window.firstCard;
    window.lockedCards = true;

    if(window.memoryCards[first] === window.memoryCards[index]) {
        window.matchedCards.push(first,index);
        updateScore(20);
        window.firstCard = null;
        window.lockedCards = false;

        if(window.matchedCards.length === window.memoryCards.length) {
            endGame("Congratulations! All pairs matched!");
        }
    } else {
        setTimeout(() => {
            document.getElementById("card" + first).textContent = "❓";
            document.getElementById("card" + index).textContent = "❓";
            window.firstCard = null;
            window.lockedCards = false;
        },800);
    }
}

function renderTic() {
    document.getElementById("gameArea").innerHTML = `
        <div style="display:grid;grid-template-columns:repeat(3,70px);justify-content:center;gap:5px">
        ${board.map((cell,i) =>
            `<button style="height:70px;font-size:25px"
            onclick="ticMove(${i})">${cell}</button>`
        ).join("")}
        </div>
        <p>Current Player: ${currentPlayer}</p>
    `;
}

function ticMove(index) {
    if(board[index] !== "" || gameOver) return;

    board[index] = currentPlayer;

    let wins = [
        [0,1,2],[3,4,5],[6,7,8],
        [0,3,6],[1,4,7],[2,5,8],
        [0,4,8],[2,4,6]
    ];

    if(wins.some(combo => combo.every(i => board[i] === currentPlayer))) {
        gameOver = true;
        updateScore(50);
        endGame(currentPlayer + " wins!");
        renderTic();
        return;
    }

    if(board.every(cell => cell !== "")) {
        gameOver = true;
        endGame("It's a draw!");
        renderTic();
        return;
    }

    currentPlayer = currentPlayer === "X" ? "O" : "X";
    renderTic();
}

function reactionClick() {
    let button = document.getElementById("reactionButton");

    if(!button) return;

    if(button.dataset.state === "waiting") {
        clearTimeout(reactionTimeout);
        button.dataset.state = "finished";
        button.textContent = "Too early! Restart.";
        endGame("You clicked too early!");
        return;
    }

    if(button.dataset.state === "ready") {
        let reaction = Date.now() - reactionStart;

        score = Math.max(0, Math.round(1000 - reaction));
        document.getElementById("score").textContent = score;

        button.dataset.state = "finished";
        button.textContent = reaction + " ms";

        endGame("Your reaction time: " + reaction + " ms");
    }
}

function startReaction() {
    let button = document.getElementById("reactionButton");
    if(!button) return;

    button.dataset.state = "waiting";
    button.textContent = "WAIT...";
    button.style.background = "#ff3333";

    reactionTimeout = setTimeout(() => {
        if(button.dataset.state !== "waiting") return;

        button.dataset.state = "ready";
        button.textContent = "CLICK NOW!";
        button.style.background = "#00cc66";
        reactionStart = Date.now();
    }, Math.random()*3000 + 1000);
}

function endGame(message) {
    document.getElementById("result").textContent = message;
    submitGameResult();
}

function restartGame() {
    if(currentGame) startGame(currentGame);
}

function showMenu() {
    clearTimeout(reactionTimeout);
    currentGame = "";

    document.getElementById("gameTitle").textContent = "🎮 Welcome to Arcade!";
    document.getElementById("instructions").textContent =
        "Select a mini-game to begin playing.";
    document.getElementById("gameArea").innerHTML = "";
    document.getElementById("result").textContent = "";
    score = 0;
    document.getElementById("score").textContent = 0;
}

async function submitGameResult() {
    const gameResult = {
        game: currentGame,
        score: score,
        completed: true,
        timestamp: new Date().toISOString()
    };

    console.log("Game result ready for backend:", gameResult);

    try {
        const response = await fetch("/api/game-results", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(gameResult)
        });

        if (!response.ok) throw new Error("Submission failed");

        const data = await response.json();
        console.log("Validated reward:", data);
    } catch(error) {
        console.error(error);
    }
    */
}

document.addEventListener("click", function(e) {
    if(e.target.id === "reactionButton" &&
       e.target.dataset.state !== "waiting" &&
       e.target.dataset.state !== "ready" &&
       currentGame === "reaction") {
        startReaction();
    }
});

</script>
</body>
</html>
