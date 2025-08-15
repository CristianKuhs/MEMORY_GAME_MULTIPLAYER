<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Jogo da Memória - 2 Jogadores</title>
<style>
    * {
        box-sizing: border-box;
        font-family: 'Segoe UI', sans-serif;
        margin: 0;
        padding: 0;
    }
    body {
        background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
        color: #fff;
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
        min-height: 100vh;
    }
    h1 {
        color: #00e5ff;
        margin: 0.5rem 0;
        font-size: 2rem;
        text-shadow: 0 0 10px rgba(0, 229, 255, 0.8);
    }
    .menu {
        margin-bottom: 1rem;
        display: flex;
        flex-wrap: wrap;
        gap: 0.5rem;
        justify-content: center;
    }
    select, button {
        background: rgba(0, 229, 255, 0.15);
        border: 1px solid rgba(0, 229, 255, 0.4);
        backdrop-filter: blur(8px);
        color: #fff;
        padding: 0.5rem 1rem;
        border-radius: 10px;
        font-weight: bold;
        cursor: pointer;
        transition: all 0.3s ease;
    }
    button:hover, select:hover {
        background: rgba(0, 229, 255, 0.3);
        box-shadow: 0 0 10px #00e5ff;
    }
    .status {
        font-size: 1.1rem;
        margin: 0.5rem 0;
        text-align: center;
        min-height: 1.5rem;
    }
    .scoreboard {
        margin-bottom: 1rem;
        font-size: 1.1rem;
        display: flex;
        gap: 2rem;
        padding: 0.5rem 1rem;
        background: rgba(255, 255, 255, 0.05);
        backdrop-filter: blur(10px);
        border-radius: 12px;
        border: 1px solid rgba(255, 255, 255, 0.1);
        box-shadow: 0 4px 10px rgba(0,0,0,0.4);
    }
    .scoreboard div {
        padding: 0.3rem 0.6rem;
        border-radius: 8px;
        transition: background 0.3s ease;
    }
    .player-active {
        background: rgba(0, 229, 255, 0.25);
        box-shadow: 0 0 10px #00e5ff;
    }
    .game-board {
        display: grid;
        gap: 12px;
        margin-bottom: 1rem;
        justify-content: center;
    }
    .card {
        width: 80px;
        height: 80px;
        background: rgba(255, 255, 255, 0.05);
        border-radius: 12px;
        cursor: pointer;
        perspective: 1000px;
        position: relative;
        transform-style: preserve-3d;
        box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        transition: transform 0.3s ease, box-shadow 0.3s ease;
    }
    .card:hover {
        transform: scale(1.05);
        box-shadow: 0 0 15px #00e5ff;
    }
    .card.flip .card-inner {
        transform: rotateY(180deg);
    }
    .card-inner {
        position: relative;
        width: 100%;
        height: 100%;
        transition: transform 0.6s;
        transform-style: preserve-3d;
    }
    .card-front, .card-back {
        position: absolute;
        width: 100%;
        height: 100%;
        border-radius: 12px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 2rem;
        backface-visibility: hidden;
    }
    .card-front {
        background: rgba(0, 0, 0, 0.4);
        border: 1px solid rgba(255, 255, 255, 0.2);
    }
    .card-back {
        background: rgba(0, 229, 255, 0.9);
        color: #121212;
        transform: rotateY(180deg);
        font-size: 2.2rem;
        border: 1px solid rgba(255, 255, 255, 0.2);
    }
</style>
</head>
<body>

<h1>Jogo da Memória</h1>

<div class="menu">
    <select id="nivel">
        <option value="facil">Fácil (4x4)</option>
        <option value="medio">Médio (4x5)</option>
        <option value="dificil">Difícil (5x6)</option>
        <option value="extremo">Extremo (6x6)</option>
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
        extremo: { linhas: 6, colunas: 6, jogadas: 50 }
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

        matchedPairs = 0;
        pontosP1 = 0;
        pontosP2 = 0;
        jogadorAtual = 1;
        jogadasRestantes = jogadas;
        maxPairs = totalCartas / 2;

        atualizarPlacar();
        destacarJogador();
        statusDiv.textContent = `Vez do Jogador 1 - Jogadas restantes: ${jogadasRestantes}`;

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
        front.textContent = '❓';
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
        destacarJogador();
        statusDiv.textContent = `Vez do Jogador ${jogadorAtual} - Jogadas restantes: ${jogadasRestantes}`;
        if (jogadasRestantes <= 0) {
            encerrarJogo();
        }
    }

    function destacarJogador() {
        document.getElementById('scoreP1').classList.remove('player-active');
        document.getElementById('scoreP2').classList.remove('player-active');
        if (jogadorAtual === 1) {
            document.getElementById('scoreP1').classList.add('player-active');
        } else {
            document.getElementById('scoreP2').classList.add('player-active');
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
                statusDiv.textContent = `🏆 Jogador 1 venceu! (${pontosP1} x ${pontosP2})`;
            } else if (pontosP2 > pontosP1) {
                statusDiv.textContent = `🏆 Jogador 2 venceu! (${pontosP2} x ${pontosP1})`;
            } else {
                statusDiv.textContent = `🤝 Empate! (${pontosP1} x ${pontosP2})`;
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
