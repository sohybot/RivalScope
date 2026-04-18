<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>RivalScope</title>
  <style>
    body { font-family: sans-serif; background: #111; color: #fff; }
    .container { display: flex; gap: 20px; padding: 20px; }
    .left, .right { flex: 1; }
    .card { background: #1e1e1e; padding: 16px; border-radius: 10px; margin-bottom: 12px; }
  </style>
</head>

<body>

<div class="container">
  <div class="left">
    <input id="gameInput" placeholder="경쟁 게임명 입력" />
    <button onclick="handleAnalyze()">분석하기</button>
  </div>

  <div class="right">
    <div id="loading" style="display:none;">분석 중...</div>
    <div id="result"></div>
  </div>
</div>

<script>
const API_KEY = "YOUR_API_KEY";

/* =========================
   MAIN FLOW
========================= */
async function handleAnalyze() {
  const game = document.getElementById("gameInput").value;
  if (!game) return alert("게임명을 입력하세요");

  showLoading();

  try {
    const data = await fetchAnalysis(game);
    renderAll(data);
  } catch (e) {
    console.error(e);
    alert("분석 실패");
  } finally {
    hideLoading();
  }
}

/* =========================
   API CALL
========================= */
async function fetchAnalysis(game) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": API_KEY,
      "anthropic-version": "2023-06-01"
    },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1500,
      messages: [{
        role: "user",
        content: `게임 ${game} 분석해서 JSON으로 반환해`
      }]
    })
  });

  const result = await response.json();
  return parseJSON(result);
}

/* =========================
   JSON PARSE
========================= */
function parseJSON(result) {
  try {
    const text = result.content?.[0]?.text || "";
    return JSON.parse(text);
  } catch (e) {
    console.error("JSON 파싱 실패", e);
    return null;
  }
}

/* =========================
   RENDER
========================= */
function renderAll(data) {
  if (!data) return;

  const container = document.getElementById("result");
  container.innerHTML = `
    ${renderProfile(data.profile)}
    ${renderReactions(data.user_reactions)}
    ${renderStrengths(data)}
    ${renderInsights(data.insights)}
  `;
}

function renderProfile(profile) {
  return `
    <div class="card">
      <h3>${profile.name}</h3>
      <p>${profile.genre} / ${profile.publisher}</p>
    </div>
  `;
}

function renderReactions(reactions) {
  return `
    <div class="card">
      👍 ${reactions.positives.join(", ")}<br>
      👎 ${reactions.negatives.join(", ")}
    </div>
  `;
}

function renderStrengths(data) {
  return `
    <div class="card">
      <b>강점</b>: ${data.strengths.join(", ")}<br>
      <b>약점</b>: ${data.weaknesses.join(", ")}
    </div>
  `;
}

function renderInsights(insights) {
  return `
    <div class="card">
      ${insights.map(i => `<p>${i.title}</p>`).join("")}
    </div>
  `;
}

/* =========================
   UI
========================= */
function showLoading() {
  document.getElementById("loading").style.display = "block";
}

function hideLoading() {
  document.getElementById("loading").style.display = "none";
}
</script>

</body>
</html>
