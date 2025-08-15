<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>JOGO DA MEMÓRIA 2.0</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --primary: #00e5ff;
      --primary-dark: #00bcd4;
      --bg: #121212;
      --card-front: rgba(255, 255, 255, 0.08);
      --card-back: var(--primary);
      --glass: rgba(255, 255, 255, 0.12);
      --text: #ffffff;
    }
    * {
      box-sizing: border-box;
      font-family: 'Poppins', sans-serif;
      margin: 0;
      padding: 0;
    }
    body {
      background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      padding: 2rem;
      color: var(--text);
    }
    h1 {
      font-size: 2.5rem;
      margin-bottom: 1rem;
      text-transform: uppercase;
      letter-spacing: 2px;
      color: var(--primary);
      text-shadow: 0 0 10px var(--primary);
    }
    .controls {
      display: flex;
      gap: 1rem;
      margin-bottom: 2rem;
      flex-wrap: wrap;
      justify-content: center;
    }
    select, button {
      background: var(--glass);
      border: 1px solid var(--primary-dark);
      color: var(--text);
      padding: 0.5rem 1rem;
      border-radius: 10px;
      font-weight: 600;
      cursor: pointer;
      transition: 0.3s;
      backdrop-filter: blur(8px);
    }
    select:hover, button:hover {
      background: var(--primary-dark);
      color: #fff;
    }
    .scoreboard {
      background: var(--glass);
      border-radius: 15px;
      padding: 1rem 2rem;
      margin-bottom: 2rem;
      display: flex;
      gap: 2rem;
      flex-wrap: wrap;
      justify-content: center;
      backdrop-filter: blur(10px);
      box-shadow: 0 8px 25px rgba(0,0,0,0.3);
    }
    .scoreboard div {
      text-align: center;
      font-weight: 600;
    }
    .game-board {
      display: grid;
      gap: 12px;
      justify-content: center;
    }
    .card {
      width: 90px;
      height: 90px;
      background: var(--card-front);
      border-radius: 15px;
      cursor: pointer;
      perspective: 1000px;
      position: relative;
      transform-style: preserve-3d;
      transition: transform 0.5s;
      box-shadow: 0 5px 15px rgba(0,0,0,0.4);
    }
    .card.flip .card-inner {
      transform: rotateY(180deg);
    }
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
      border-radius: 15px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 2rem;
      backface-visibility: hidden;
    }
    .card-front {
      background: var(--card-front);
      border: 2px solid var(--glass);
    }
    .card-back {
      background: var(--card-back);
      color: #121212;
      transform: rotateY(180deg);
      border: 2px solid var(--primary-dark);
      font-size: 2.2rem;
    }
    .status {
      font-size: 1.2rem;
      margin-top: 1.5rem;
      font-weight: 600;
    }
    @media (max-width: 600px) {
      .card {
        width: 70px;
        height: 70px;
        font-size: 1.5rem;
      }
    }
  </style>
</head>
<body>
  <h1>Jogo da Memória</h1>
  <div class="controls">
    <select id="nivel">
      <option value="facil">Fácil</option>
      <option value="medio">Médio</option>
      <option value="dificil">Difícil</option>
      <option value="extremo">Extremo</option>
      <option value="deus">God</option>
    </select>
    <button onclick="iniciarJogo()">Iniciar Jogo</button>
    <button onclick="reiniciarJogo()">Reiniciar</button>
  </div>
  <div class="scoreboard">
    <div>👤 Jogador 1: <span id="score1">0</span></div>
    <div>👤 Jogador 2: <span id="score2">0</span></div>
    <div>🎮 Turno: <span id="turno">Jogador 1</span></div>
    <div>🎯 Jogadas: <span id="jogadas">0</span></div>
  </div>
  <div id="game-board" class="game-board"></div>
  <div id="status" class="status"></div>

  <audio id="flipSound" src="https://cdn.pixabay.com/audio/2022/03/01/audio_d2b96f3b84.mp3"></audio>
  <audio id="matchSound" src="https://cdn.pixabay.com/audio/2022/03/15/audio_c0b7f8aa5b.mp3"></audio>

  <script>
    const allEmojis = ['🍕','🎮','🐱','🎧','🚀','🌈','📚','🧠','⚡','🍔','🐶','🍩','🎲','🎵','📱','🏀','🎬','🥑','🐼','🌍','🦄','🎁','🍉','🌙','🔥','⭐','⚽','🐸','🍒','🕹️','🎤','✈️','📸'];
    
    const niveis = {
      facil: { pares: 6, jogadas: 20, colunas: 4 },
      medio: { pares: 8, jogadas: 30, colunas: 4 },
      dificil: { pares: 12, jogadas: 40, colunas: 6 },
      extremo: { pares: 18, jogadas: 50, colunas: 6 },
      deus: { pares: 22, jogadas: 70, colunas: 8 }
    };
    
    let board, status, flipSound, matchSound;
    let firstCard = null, secondCard = null, lockBoard = false;
    let matchedPairs = 0, score1 = 0, score2 = 0, turno = 1, jogadas = 0, limiteJogadas = 0;
    let paresTotais = 0;

    document.addEventListener('DOMContentLoaded', () => {
      board = document.getElementById('game-board');
      status = document.getElementById('status');
      flipSound = document.getElementById('flipSound');
      matchSound = document.getElementById('matchSound');
    });

    function iniciarJogo() {
      const nivel = document.getElementById('nivel').value;
      const config = niveis[nivel];
      paresTotais = config.pares;
      limiteJogadas = config.jogadas;
      board.style.gridTemplateColumns = `repeat(${config.colunas}, 1fr)`;

      const selectedEmojis = embaralhar(allEmojis).slice(0, config.pares);
      const cards = embaralhar([...selectedEmojis, ...selectedEmojis]);

      board.innerHTML = '';
      matchedPairs = 0;
      jogadas = 0;
      score1 = 0;
      score2 = 0;
      turno = 1;
      updateHUD();
      status.textContent = '';

      firstCard = null;
      secondCard = null;
      lockBoard = false;

      cards.forEach(emoji => {
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
      jogadas++;
      updateHUD();

      if (firstCard.dataset.emoji === secondCard.dataset.emoji) {
        matchSound.play();
        matchedPairs++;
        if (turno === 1) score1++; else score2++;
        resetarVirada();
        if (matchedPairs === paresTotais) {
          fimDeJogo();
        }
      } else {
        setTimeout(() => {
          firstCard.classList.remove('flip');
          secondCard.classList.remove('flip');
          resetarVirada();
          turno = turno === 1 ? 2 : 1;
          updateHUD();
        }, 1000);
      }

      if (jogadas >= limiteJogadas && matchedPairs < paresTotais) {
        fimDeJogo();
      }
    }

    function resetarVirada() {
      [firstCard, secondCard] = [null, null];
      lockBoard = false;
    }

    function embaralhar(array) {
      return array.sort(() => Math.random() - 0.5);
    }

    function reiniciarJogo() {
      iniciarJogo();
    }

    function updateHUD() {
      document.getElementById('score1').textContent = score1;
      document.getElementById('score2').textContent = score2;
      document.getElementById('turno').textContent = turno === 1 ? 'Jogador 1' : 'Jogador 2';
      document.getElementById('jogadas').textContent = `${jogadas}/${limiteJogadas}`;
    }

    function fimDeJogo() {
      if (score1 > score2) status.textContent = "🏆 Jogador 1 venceu!";
      else if (score2 > score1) status.textContent = "🏆 Jogador 2 venceu!";
      else status.textContent = "🤝 Empate!";
    }
  </script>
</body>
</html>
