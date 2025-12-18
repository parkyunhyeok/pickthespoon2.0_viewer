<html lang="ko" translate="no" class="notranslate">
<head>
  <meta charset="utf-8" />
  <!-- ✅ iOS 안전영역 + 모바일 최적화 -->
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  <!-- ✅ 자동번역 방지 -->
  <meta name="google" content="notranslate" />
  <meta name="robots" content="notranslate" />
  <title>RKS 팀 매칭 결과</title>

  <style>
    :root{
      --page-pad: 24px;
      --card-pad: 20px;
      --radius: 16px;
    }

    /* ✅ iOS/Android 공통: 자동 글자 확대(특히 iOS) 방지 */
    html{
      -webkit-text-size-adjust: 100%;
      text-size-adjust: 100%;
    }

    body{
      font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
      background:#f3f4f6;
      margin:0;

      /* ✅ 노치/안전영역 대응 */
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
      text-align:center;
      letter-spacing:-0.2px;
    }

    .time{
      font-size:13px;
      color:#6b7280;
      margin-bottom:14px;
      text-align:center;
    }

    pre{
      margin:0;
      background:#f9fafb;
      padding:16px;
      border-radius:12px;

      /* ✅ 가독성 좋은 줄바꿈 (안드/IOS 모두 안정적) */
      white-space:pre-wrap;     /* 원래 줄바꿈 유지 + 화면폭에 맞게 랩 */
      overflow-wrap:anywhere;   /* 긴 토큰도 강제로 끊어서 랩 */
      word-break:break-word;    /* 단어가 길면 끊어서 랩 */

      font-size:15px;
      line-height:1.7;

      /* ✅ iOS 스크롤 부드럽게(긴 결과일 때) */
      -webkit-overflow-scrolling: touch;
    }

    .hint{
      font-size:12px;
      color:#9ca3af;
      margin-top:12px;
      text-align:center;
    }

    /* ✅ 작은 모바일 최적화 */
    @media (max-width: 420px){
      :root{
        --page-pad: 14px;
        --card-pad: 16px;
        --radius: 14px;
      }

      h1{ font-size:20px; }

      pre{
        font-size:14px;
        line-height:1.8;
        padding:14px;
      }
    }

    /* ✅ 아주 작은 기기(옛 iPhone SE 등) */
    @media (max-width: 360px){
      pre{ font-size:13.5px; }
    }
  </style>
</head>

<body>
  <div class="card notranslate">
    <h1 class="notranslate">🏸 RKS 팀 매칭 결과</h1>
    <div class="time notranslate" id="time"></div>
    <pre class="notranslate" id="result">결과를 불러오는 중…</pre>
    <div class="hint notranslate">이 페이지는 자동으로 최신 결과를 누적 표시합니다 (최근 3시간)</div>
  </div>

  <script>
    document.addEventListener("DOMContentLoaded", () => {
      const API_URL =
        "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

      const HOURS = 3;
      const MAX_AGE_MS = HOURS * 60 * 60 * 1000;
      const STORAGE_KEY = "rks_team_results_v1";

      function safe(v){
        return (v === null || v === undefined) ? "" : String(v);
      }
      function normalizeText(t){
        return safe(t).replace(/\\n/g, "\n").trim();
      }
      function parsePickedAt(v){
        if (!v) return null;
        const d = new Date(v);
        return isNaN(d.getTime()) ? null : d;
      }
      function loadHistory(){
        try{
          const raw = localStorage.getItem(STORAGE_KEY);
          const arr = raw ? JSON.parse(raw) : [];
          return Array.isArray(arr) ? arr : [];
        } catch {
          return [];
        }
      }
      function saveHistory(arr){
        localStorage.setItem(STORAGE_KEY, JSON.stringify(arr));
      }
      function pruneHistory(arr){
        const now = Date.now();
        return arr.filter(item =>
          item &&
          typeof item.ts === "number" &&
          typeof item.text === "string" &&
          (now - item.ts) <= MAX_AGE_MS
        );
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

        const blocks = sorted.map(item => {
          const when = new Date(item.ts).toLocaleString();
          return `🕒 ${when}\n${item.text}`;
        });

        resultEl.innerText = blocks.join("\n\n--------------------------------\n\n");
        timeEl.innerText = `최근 ${HOURS}시간 결과 ${sorted.length}건`;
      }

      async function loadResult(){
        try{
          // ✅ 모바일 사파리 캐시/프리로드로 인한 stale 방지
          const url = API_URL + "?_=" + Date.now();
          const res = await fetch(url, { cache: "no-store" });

          const json = await res.json();
          const data = (json && json.data) ? json.data : json;

          const resultTextRaw = data.resultText || data.text || "";
          const pickedAtRaw   = data.pickedAt || data.updatedAt || "";

          const resultText = normalizeText(resultTextRaw);

          let history = pruneHistory(loadHistory());

          if (!resultText){
            saveHistory(history);
            renderHistory(history);
            return;
          }

          const pickedAtDate = parsePickedAt(pickedAtRaw);
          const ts = pickedAtDate ? pickedAtDate.getTime() : Date.now();

          const exists = history.some(h => h.ts === ts && h.text === resultText);
          if (!exists){
            history.push({ ts, text: resultText });
          }

          history = pruneHistory(history);
          saveHistory(history);
          renderHistory(history);

        } catch(e){
          const history = pruneHistory(loadHistory());
          saveHistory(history);
          renderHistory(history);
        }
      }

      loadResult();
      setInterval(loadResult, 15000);
    });
  </script>
</body>
</html>
