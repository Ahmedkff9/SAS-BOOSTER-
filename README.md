# SAS-BOOSTER-
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SAS | Score Exact Boosté</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f0f4f8;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 600px;
      margin: auto;
      margin-top: 50px;
      background: #fff;
      padding: 25px;
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0,0,0,0.1);
    }
    h1, h2 {
      text-align: center;
      color: #003366;
    }
    label {
      font-weight: bold;
      display: block;
      margin-top: 15px;
      color: #222;
    }
    input[type="text"], input[type="submit"], button {
      width: 100%;
      padding: 12px;
      margin-top: 5px;
      border-radius: 6px;
      border: 1px solid #ccc;
      font-size: 16px;
    }
    input[type="submit"], button {
      background-color: #007bff;
      color: white;
      font-weight: bold;
      border: none;
      cursor: pointer;
    }
    input[type="submit"]:hover, button:hover {
      background-color: #0056b3;
    }
    .result {
      background: #e6ffed;
      padding: 15px;
      margin-top: 20px;
      border-radius: 6px;
      font-size: 17px;
    }
    .hidden { display: none; }
    ol {
      padding-left: 20px;
    }
    ol li {
      font-size: 18px;
      font-weight: bold;
      margin-bottom: 10px;
      color: #000;
    }
    .btn-group {
      display: flex;
      gap: 10px;
      margin-top: 20px;
    }
    @media (max-width: 500px) {
      .container { margin: 20px; padding: 15px; }
      h1, h2 { font-size: 22px; }
    }
  </style>
</head>
<body>
  <div id="page1" class="container">
    <h1>⚽ Score Exact SAS+ Boosté</h1>
    <form id="formScores">
      <label><input type="checkbox" id="useHalftime"> Utiliser le score de mi-temps</label>
      <label>Score à la mi-temps</label>
      <input type="text" id="halftime" placeholder="Ex: 1-1">
      <label>Score Passé 1 (le plus récent)</label>
      <input type="text" id="score1" required placeholder="Ex: 2-1">
      <label>Score Passé 2</label>
      <input type="text" id="score2" required placeholder="Ex: 1-0">
      <label>Score Passé 3</label>
      <input type="text" id="score3" required placeholder="Ex: 3-2">
      <label>Score Passé 4</label>
      <input type="text" id="score4" required placeholder="Ex: 0-0">
      <label>Score Passé 5 (le plus ancien)</label>
      <input type="text" id="score5" required placeholder="Ex: 2-2">
      <input type="submit" value="Générer la prédiction">
    </form>
  </div>

  <div id="page2" class="container hidden">
    <h2>📊 Prédictions Générées</h2>
    <div id="resultat" class="result"></div>
    <div class="btn-group">
      <button onclick="retourPage1()">🔙 Retour</button>
      <button onclick="location.reload()">🔄 Réinitialiser</button>
      <button onclick="copierScores()">📋 Copier</button>
    </div>
  </div>

  <script>
    function parseScore(score) {
      if (!/^\d+-\d+$/.test(score)) return null;
      return score.split('-').map(Number);
    }

    function calculerPondérée(scores) {
      let totalA = 0, totalB = 0, totalPoids = 0;
      for (let i = 0; i < scores.length; i++) {
        let poids = scores.length - i; // score1 + lourd que score5
        let [a, b] = scores[i];
        totalA += a * poids;
        totalB += b * poids;
        totalPoids += poids;
      }
      return [Math.round(totalA / totalPoids), Math.round(totalB / totalPoids)];
    }

    function predireScores(moyA, moyB, miA, miB, useMi) {
      let baseA = useMi ? miA : 0;
      let baseB = useMi ? miB : 0;
      let delta = useMi ? 0.5 : 1;

      let predictions = [];
      for (let i = -1; i <= 1; i++) {
        let scoreA = Math.round(baseA + (moyA + i * delta) / 2);
        let scoreB = Math.round(baseB + (moyB + i * delta) / 2);
        predictions.push([scoreA, scoreB]);
      }

      return predictions.filter((v, i, arr) => arr.findIndex(t => t[0] === v[0] && t[1] === v[1]) === i);
    }

    function afficherResultats(predictions) {
      const pourcentages = [50, 30, 20];
      let html = `<p><strong>⚽ 3 Scores Finaux Possibles :</strong></p><ol>`;
      predictions.forEach((p, i) => {
        html += `<li>${p[0]} - ${p[1]} (${pourcentages[i] || 10}%)</li>`;
      });
      html += `</ol>`;
      let total = predictions[0][0] + predictions[0][1];
      html += `<p><strong>📈 Total de Buts (score le plus probable) :</strong> ${total} buts</p>`;
      document.getElementById('resultat').innerHTML = html;
    }

    function copierScores() {
      const txt = document.getElementById("resultat").innerText;
      navigator.clipboard.writeText(txt).then(() => {
        alert("Résultats copiés dans le presse-papier !");
      });
    }

    document.getElementById('formScores').addEventListener('submit', function(e) {
      e.preventDefault();

      const useHalftime = document.getElementById('useHalftime').checked;
      const halftimeRaw = document.getElementById('halftime').value.trim() || "0-0";
      const halftimeParsed = parseScore(halftimeRaw);
      if (!halftimeParsed) return alert("Score de mi-temps invalide (format attendu: 1-1)");
      const [miA, miB] = halftimeParsed;

      let scores = [];
      for (let i = 1; i <= 5; i++) {
        let raw = document.getElementById('score' + i).value.trim();
        let parsed = parseScore(raw);
        if (!parsed) return alert("Le score " + i + " est invalide (ex: 2-1)");
        scores.push(parsed);
      }

      const [moyA, moyB] = calculerPondérée(scores);
      const predictions = predireScores(moyA, moyB, miA, miB, useHalftime);

      document.getElementById('page1').classList.add('hidden');
      document.getElementById('page2').classList.remove('hidden');
      afficherResultats(predictions);
    });

    function retourPage1() {
      document.getElementById('page1').classList.remove('hidden');
      document.getElementById('page2').classList.add('hidden');
    }
  </script>
</body>
</html>