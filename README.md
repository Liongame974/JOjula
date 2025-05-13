<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Évite les blocs !</title>
  <link rel="manifest" href="manifest.json" />
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: white;
      font-family: sans-serif;
    }

    canvas {
      display: block;
      background: #f0f0f0;
    }

    #touchControls {
      position: absolute;
      bottom: 20px;
      width: 100%;
      text-align: center;
    }

    .touch-btn {
      background-color: #3498db;
      color: white;
      font-size: 30px;
      border: none;
      padding: 20px 30px;
      margin: 0 20px;
      border-radius: 10px;
      opacity: 0.8;
    }

    .touch-btn:active {
      background-color: #2980b9;
    }

    #scoreBoard {
      position: absolute;
      top: 10px;
      left: 10px;
      color: #333;
      font-size: 18px;
      background: rgba(255, 255, 255, 0.7);
      padding: 5px 10px;
      border-radius: 10px;
    }

    #rejouerBtn {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 24px;
      padding: 15px 30px;
      background-color: #2ecc71;
      color: white;
      border: none;
      border-radius: 10px;
    }

    @media (min-width: 600px) {
      #touchControls {
        display: none;
      }
    }
  </style>
</head>
<body>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<div id="scoreBoard">Score : 0 | Meilleur : 0</div>
<div id="touchControls">
  <button class="touch-btn" id="leftBtn">⬅️</button>
  <button class="touch-btn" id="rightBtn">➡️</button>
</div>
<button id="rejouerBtn">Rejouer</button>

<audio id="musique" src="intrépide.mp3" loop autoplay></audio>

<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");
  const musique = document.getElementById("musique");
  const rejouerBtn = document.getElementById("rejouerBtn");

  const largeur = canvas.width;
  const hauteur = canvas.height;

  let joueur, blocs, frame, perdu, score, meilleur;

  function initialiser() {
    joueur = {
      x: 175,
      y: 500,
      largeur: 50,
      hauteur: 50,
      vitesse: 5,
      couleur: "blue",
      direction: 0
    };
    blocs = [];
    frame = 0;
    perdu = false;
    score = 0;
    meilleur = localStorage.getItem("meilleurScore") || 0;
    document.getElementById("scoreBoard").textContent = `Score : 0 | Meilleur : ${meilleur}`;
    musique.currentTime = 0;
    musique.play();
    rejouerBtn.style.display = "none";
    boucleJeu();
  }

  const leftBtn = document.getElementById("leftBtn");
  const rightBtn = document.getElementById("rightBtn");

  leftBtn.addEventListener("touchstart", () => joueur.direction = -1);
  leftBtn.addEventListener("touchend", () => joueur.direction = 0);

  rightBtn.addEventListener("touchstart", () => joueur.direction = 1);
  rightBtn.addEventListener("touchend", () => joueur.direction = 0);

  document.addEventListener("keydown", (e) => {
    if (e.key === "ArrowLeft") joueur.direction = -1;
    if (e.key === "ArrowRight") joueur.direction = 1;
  });
  document.addEventListener("keyup", (e) => {
    if (["ArrowLeft", "ArrowRight"].includes(e.key)) joueur.direction = 0;
  });

  function genererBloc() {
    const x = Math.random() * (largeur - 50);
    blocs.push({ x, y: 0, largeur: 50, hauteur: 50 });
  }

  function afficherTexte(message) {
    ctx.fillStyle = "red";
    ctx.font = "40px Arial";
    ctx.textAlign = "center";
    ctx.fillText(message, largeur / 2, hauteur / 2);
  }

  function boucleJeu() {
    if (perdu) return;

    ctx.clearRect(0, 0, largeur, hauteur);

    joueur.x += joueur.direction * joueur.vitesse;
    joueur.x = Math.max(0, Math.min(joueur.x, largeur - joueur.largeur));

    ctx.fillStyle = joueur.couleur;
    ctx.fillRect(joueur.x, joueur.y, joueur.largeur, joueur.hauteur);

    if (frame % 25 === 0) genererBloc();

    for (let i = blocs.length - 1; i >= 0; i--) {
      const bloc = blocs[i];
      bloc.y += 5;
      ctx.fillStyle = "red";
      ctx.fillRect(bloc.x, bloc.y, bloc.largeur, bloc.hauteur);

      if (
        bloc.x < joueur.x + joueur.largeur &&
        bloc.x + bloc.largeur > joueur.x &&
        bloc.y < joueur.y + joueur.hauteur &&
        bloc.y + bloc.hauteur > joueur.y
      ) {
        perdu = true;
        afficherTexte("Perdu !");
        musique.pause();
        if (score > meilleur) {
          localStorage.setItem("meilleurScore", score);
        }
        rejouerBtn.style.display = "block";
        return;
      }

      if (bloc.y > hauteur) {
        blocs.splice(i, 1);
        score++;
      }
    }

    document.getElementById("scoreBoard").textContent = `Score : ${score} | Meilleur : ${Math.max(score, meilleur)}`;
    frame++;
    requestAnimationFrame(boucleJeu);
  }

  // Démarrer musique sur interaction (obligatoire sur mobile)
  document.body.addEventListener("click", () => musique.play().catch(() => {}), { once: true });

  rejouerBtn.addEventListener("click", () => initialiser());

  initialiser();
</script>
</body>
</html>

<script>
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('service-worker.js')
        .then(registration => {
          console.log('Service Worker enregistré avec succès:', registration);
        })
        .catch(error => {
          console.log('Échec de l\'enregistrement du Service Worker:', error);
        });
    });
  }
</script>
