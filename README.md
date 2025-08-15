<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Jogo da Memória - Multiplayer</title>
<style>
    * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
    body {
        background-color: #121212;
        color: #fff;
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
    }
    h1 { color: #00e5ff; margin: 0.5rem 0; }
    .menu {
        margin-bottom: 1rem;
        display: flex;
        flex-wrap: wrap;
        gap: 0.5rem;
        justify-content: center;
    }
    select, button {
        background-color: #00e5ff;
        color: #121212;
        padding: 0.4rem 0.8rem;
        border: none;
        border-radius: 6px;
        font-weight: bold;
        cursor: pointer;
    }
    button:hover, select:hover { background-color: #00bcd4; }
    .status {
        font-size: 1rem;
        margin: 0.5rem 0;
        text-align: center;
    }
    .scoreboard {
        margin-bottom: 1rem;
        font-size: 1.1rem;
        display: flex;
        gap: 2rem;
    }
    .game-board {
        display: grid;
        gap: 10px;
        margin-bottom: 1rem;
    }
    .card {
        width: 80px;
        height: 80px;
        background-color: #1e1e1e;
        border-radius: 10px;
        cursor: pointer;
        perspective: 1000px;
        position: relative;
        transform-style: preserve-3d;
    }
    .card.flip .card-inner { transform: rotateY(180deg); }
    .card-inner {
        position: relative;
        width: 100%;
        height: 100%;
        transition: transform 0.5s;
        transform-style: preserve-3d;
    }
    .card-front, .card-back {
        position: absolute;
        width: 100%;
        height: 100%;
        border-radius: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 2rem;
        backface-visibility: hidden;
    }
    .card-front { background-color: #2c2c2c; }
    .card-back {
        background-color: #00e5ff;
        color: #121212;
        transform: rotateY(180deg);
    }
</style>
</head>
<body>

<h1>MEMORY GAME 2.0</h1>

<div class="menu">
    <select id="nivel">
        <option value="facil">EASY (4x4)</option>
        <option value="medio">MIDDLE (4x5)</option>
        <option value="dificil">HARD (5x6)</option>
        <option value="extremo">EXTREME (6x6)</option>
        <option value="deus">GOD (10X10)</option>
    </select>
    <button onclick="iniciarJogo()">Iniciar Jogo</button>
</div>

<div class="scoreboard">
    <div id="scoreP1">Jogador 1: 0</div>
    <div id="scoreP2">Jogador 2: 0</div>
</div>

<div id="status" class="status">Selecione um nível e clique em Iniciar</div>

<div id="game-board" class="game-board"></div>

<audio id="flipSound" src="https://cdn.pixabay.com/audio/2022/03/01/audio_d2b96f3b84.mp3"></audio>
<audio id="matchSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_c0b7f8aa5b.mp3"></audio>

<script>
    const listaEmojisBase = ['🍕','🎮','🐱','🎧','🚀','🌈','📚','🧠','⚽','🍔','🎲','🏆','🐶','🍩','🎹','🚗','🌍','🦋','🎤','🍓','🛸','🥇','🎯','🐠','🍿','📀','🎨','🛹','🦄','🎁','🎬','🏖️','🌋','🪐','🌙','🧩','🎃','🎄','🎆','🪙','🗿','🚓','💎','🥑','🍇','🪵','🐢','🐍','🦖','🦕','🐙','🐳','🐬','🦈'];
    
    const niveis = {
        facil: { linhas: 4, colunas: 4, jogadas: 20 },
        medio: { linhas: 4, colunas: 5, jogadas: 30 },
        dificil: { linhas: 5, colunas: 6, jogadas: 40 },
        extremo: { linhas: 6, colunas: 6, jogadas: 50 },
        deus: { linhas: 10, colunas: 10, jogadas: 80}
    };

    let board, statusDiv, flipSound, matchSound;
    let firstCard = null, secondCard = null, lockBoard = false;
    let matchedPairs = 0, maxPairs = 0;
    let jogadorAtual = 1, pontosP1 = 0, pontosP2 = 0;
    let jogadasRestantes = 0;

    document.addEventListener('DOMContentLoaded', () => {
        board = document.getElementById('game-board');
        statusDiv = document.getElementById('status');
        flipSound = document.getElementById('flipSound');
        matchSound = document.getElementById('matchSound');
    });

    function iniciarJogo() {
        const nivelSelecionado = document.getElementById('nivel').value;
        const { linhas, colunas, jogadas } = niveis[nivelSelecionado];
        const totalCartas = linhas * colunas;
        const emojis = embaralhar([...listaEmojisBase].sort(() => Math.random() - 0.5).slice(0, totalCartas / 2));
        const cartas = embaralhar([...emojis, ...emojis]);

        // Reset variáveis
        matchedPairs = 0;
        pontosP1 = 0;
        pontosP2 = 0;
        jogadorAtual = 1;
        jogadasRestantes = jogadas;
        maxPairs = totalCartas / 2;

        atualizarPlacar();
        statusDiv.textContent = `Vez do Jogador 1 - Jogadas restantes: ${jogadasRestantes}`;

        // Render tabuleiro
        board.innerHTML = '';
        board.style.gridTemplateColumns = `repeat(${colunas}, 80px)`;
        cartas.forEach(emoji => {
            const card = criarCarta(emoji);
            board.appendChild(card);
        });
    }

    function criarCarta(emoji) {
        const card = document.createElement('div');
        card.className = 'card';
        const inner = document.createElement('div');
        inner.className = 'card-inner';
        const front = document.createElement('div');
        front.className = 'card-front';
        front.textContent = '?';
        const back = document.createElement('div');
        back.className = 'card-back';
        back.textContent = emoji;
        inner.appendChild(front);
        inner.appendChild(back);
        card.appendChild(inner);
        card.dataset.emoji = emoji;
        card.addEventListener('click', () => virarCarta(card));
        return card;
    }

    function virarCarta(card) {
        if (lockBoard || card === firstCard || card.classList.contains('flip')) return;
        flipSound.play();
        card.classList.add('flip');
        if (!firstCard) {
            firstCard = card;
            return;
        }
        secondCard = card;
        lockBoard = true;
        if (firstCard.dataset.emoji === secondCard.dataset.emoji) {
            matchSound.play();
            matchedPairs++;
            if (jogadorAtual === 1) pontosP1++; else pontosP2++;
            resetarVirada();
            atualizarPlacar();
            verificarFim();
        } else {
            setTimeout(() => {
                firstCard.classList.remove('flip');
                secondCard.classList.remove('flip');
                alternarJogador();
                resetarVirada();
            }, 1000);
        }
    }

    function alternarJogador() {
        jogadorAtual = jogadorAtual === 1 ? 2 : 1;
        jogadasRestantes--;
        statusDiv.textContent = `Vez do Jogador ${jogadorAtual} - Jogadas restantes: ${jogadasRestantes}`;
        if (jogadasRestantes <= 0) {
            encerrarJogo();
        }
    }

    function atualizarPlacar() {
        document.getElementById('scoreP1').textContent = `Jogador 1: ${pontosP1}`;
        document.getElementById('scoreP2').textContent = `Jogador 2: ${pontosP2}`;
    }

    function verificarFim() {
        if (matchedPairs === maxPairs) {
            encerrarJogo(true);
        }
    }

    function encerrarJogo(completo = false) {
        lockBoard = true;
        if (completo) {
            if (pontosP1 > pontosP2) {
                statusDiv.textContent = `Fim de jogo! Jogador 1 venceu 🏆 (${pontosP1} x ${pontosP2})`;
            } else if (pontosP2 > pontosP1) {
                statusDiv.textContent = `Fim de jogo! Jogador 2 venceu 🏆 (${pontosP2} x ${pontosP1})`;
            } else {
                statusDiv.textContent = `Fim de jogo! Empate 🤝 (${pontosP1} x ${pontosP2})`;
            }
        } else {
            statusDiv.textContent += " | Fim das jogadas!";
        }
    }

    function resetarVirada() {
        [firstCard, secondCard] = [null, null];
        lockBoard = false;
    }

    function embaralhar(array) {
        return array.sort(() => Math.random() - 0.5);
    }
</script>
</body>
</html>
