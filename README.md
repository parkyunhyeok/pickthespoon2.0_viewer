<html lang="ko" translate="no" class="notranslate">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  <meta name="google" content="notranslate" />
  <meta name="robots" content="notranslate" />
  <title>RKS 팀 매칭 결과</title>

  <!-- ✅ 가독성 좋은 한글 폰트 -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Pretendard:wght@400;500;600;700&display=swap" rel="stylesheet">

  <style>
    :root{
      --page-pad: 24px;
      --card-pad: 20px;
      --radius: 16px;
    }

    html{
      -webkit-text-size-adjust: 100%;
      text-size-adjust: 100%;
    }

    body{
      font-family:
        "Pretendard",
        system-ui,
        -apple-system,
        BlinkMacSystemFont,
        "Segoe UI",
        Roboto,
        sans-serif;

      background:#f3f4f6;
      margin:0;
      padding:
        calc(var(--page-pad) + env(safe-area-inset-top))
        calc(var(--page-pad) + env(safe-area-inset-right))
        calc(var(--page-pad) + env(safe-area-inset-bottom))
        calc(var(--page-pad) + env(safe-area-inset-left));
    }

    .card{
      max-width:720px;
      margin:0 auto;
      background:#ffffff;
      border-radius:var(--radius);
      padding:var(--card-pad);
      box-shadow:0 12px 30px rgba(0,0,0,.12);
    }

    h1{
      margin:0 0 12px;
      font-size:22px;
      font-weight:700;
      text-align:center;
      letter-spacing:-0.3px;
    }

    .time{
      font-size:13px;
      font-weight:500;
      color:#6b7280;
      margin-bottom:14px;
      text-align:center;
    }

    pre{
      margin:0;
      background:#f9fafb;
      padding:16px;
      border-radius:12px;

      /* ✅ 텍스트 문서처럼 보이게 */
      font-family: inherit;
      font-weight:500;

      white-space:pre-wrap;
      overflow-wrap:anywhere;
      word-break:break-word;

      font-size:15px;
      line-height:1.8;
      color:#111827;

      -webkit-overflow-scrolling: touch;
    }

    .hint{
      font-size:12px;
      font-weight:500;
      color:#9ca3af;
      margin-top:12px;
      text-align:center;
    }

    @media (max-width: 420px){
      :root{
        --page-pad: 14px;
        --card-pad: 16px;
        --radius: 14px;
      }

      h1{ font-size:20px; }

      pre{
        font-size:14px;
        line-height:1.85;
        padding:14px;
      }
    }
  </style>
</head>

<body>
  <div class="card notranslate">
    <h1>🏸팀 매칭 결과</h1>
    <div class="time" id="time"></div>
    <pre id="result">결과를 불러오는 중…</pre>
    <div class="hint">최근 3시간 동안의 팀 매칭 결과가 누적 표시됩니다</div>
  </div>

  <script>
    document.addEventListener("DOMContentLoaded", () => {
      const API_URL =
        "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

      const HOURS = 3;
      const MAX_AGE_MS = HOURS * 60 * 60 * 1000;
      const STORAGE_KEY = "rks_team_results_v1";

      function normalizeText(t){
        return (t ?? "").toString().replace(/\\n/g, "\n").trim();
      }

      function formatTime(ts){
        const d = new Date(ts);
        const mm = String(d.getMonth() + 1).padStart(2, "0");
        const dd = String(d.getDate()).padStart(2, "0");
        const hh = String(d.getHours()).padStart(2, "0");
        const mi = String(d.getMinutes()).padStart(2, "0");
        return `${mm}.${dd} ${hh}:${mi}`;
      }

      function loadHistory(){
        try{
          return JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
        } catch {
          return [];
        }
      }

      function saveHistory(arr){
        localStorage.setItem(STORAGE_KEY, JSON.stringify(arr));
      }

      function pruneHistory(arr){
        const now = Date.now();
        return arr.filter(v => now - v.ts <= MAX_AGE_MS);
      }

      function renderHistory(arr){
        const resultEl = document.getElementById("result");
        const timeEl   = document.getElementById("time");

        if (!arr.length){
          resultEl.innerText = "최근 3시간 내 팀 매칭 결과가 없습니다.";
          timeEl.innerText = "";
          return;
        }

        const sorted = [...arr].sort((a,b) => b.ts - a.ts);

        const blocks = sorted.map(v =>
          `🕒 ${formatTime(v.ts)}\n${v.text}`
        );

        resultEl.innerText = blocks.join("\n\n────────────────────────\n\n");
        timeEl.innerText = `최근 ${HOURS}시간 · ${sorted.length}건`;
      }

      async function loadResult(){
        try{
          const res = await fetch(API_URL + "?_=" + Date.now(), { cache:"no-store" });
          const json = await res.json();
          const data = json.data ?? json;

          const text = normalizeText(data.resultText || data.text || "");
          const ts = data.pickedAt ? new Date(data.pickedAt).getTime() : Date.now();

          let history = pruneHistory(loadHistory());

          if (text && !history.some(h => h.ts === ts && h.text === text)){
            history.push({ ts, text });
          }

          history = pruneHistory(history);
          saveHistory(history);
          renderHistory(history);

        } catch {
          renderHistory(pruneHistory(loadHistory()));
        }
      }

      loadResult();
      setInterval(loadResult, 15000);
    });
  </script>
</body>
</html>
