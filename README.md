<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>RKS 팀 매칭 결과</title>

  <style>
    body{
      font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
      background:#f3f4f6;
      margin:0;
      padding:24px;
    }
    .card{
      max-width:720px;
      margin:0 auto;
      background:#ffffff;
      border-radius:16px;
      padding:20px;
      box-shadow:0 12px 30px rgba(0,0,0,.12);
    }
    h1{
      margin:0 0 12px;
      font-size:22px;
      text-align:center;
    }
    .time{
      font-size:13px;
      color:#6b7280;
      margin-bottom:14px;
      text-align:center;
    }
    pre{
      white-space:pre-wrap;
      word-break:keep-all;
      font-size:15px;
      line-height:1.6;
      background:#f9fafb;
      padding:16px;
      border-radius:12px;
      margin:0;
    }
    .hint{
      font-size:12px;
      color:#9ca3af;
      margin-top:12px;
      text-align:center;
    }
  </style>
</head>

<body>
  <div class="card">
    <h1>🏸 RKS 팀 매칭 결과</h1>
    <div class="time" id="time"></div>
    <pre id="result">결과를 불러오는 중…</pre>
    <div class="hint">이 페이지는 자동으로 최신 결과를 누적 표시합니다 (최근 3시간)</div>
  </div>

  <script>
    /*********************************************************
     * 🔽 Apps Script Web App URL
     *********************************************************/
    const API_URL =
      "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

    // ✅ 최근 N시간 누적 표시 설정
    const HOURS = 3;
    const MAX_AGE_MS = HOURS * 60 * 60 * 1000;
    const STORAGE_KEY = "rks_team_results_v1";

    function safe(v){
      return (v === null || v === undefined) ? "" : String(v);
    }

    function normalizeText(t){
      // \n이 문자열로 올 경우 실제 줄바꿈 처리
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

      // 최신순 정렬
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
        // 캐시 방지용 타임스탬프
        const res = await fetch(API_URL + "?_=" + Date.now());
        const json = await res.json();

        // Apps Script 응답 형태가 달라도 대응
        const data = json.data ?? json;

        const resultTextRaw = data.resultText || data.text || "";
        const pickedAtRaw   = data.pickedAt || data.updatedAt || "";

        const resultText = normalizeText(resultTextRaw);

        // 기존 기록 불러오고 오래된 것 삭제
        let history = pruneHistory(loadHistory());

        if (!resultText){
          // 결과가 없으면 누적만 유지해서 보여줌
          saveHistory(history);
          renderHistory(history);
          return;
        }

        // pickedAt이 없거나 파싱 실패하면 "지금"으로 기록
        const pickedAtDate = parsePickedAt(pickedAtRaw);
        const ts = pickedAtDate ? pickedAtDate.getTime() : Date.now();

        // ✅ 중복 방지 (같은 ts + 같은 text면 추가 안 함)
        const exists = history.some(h => h.ts === ts && h.text === resultText);
        if (!exists){
          history.push({ ts, text: resultText });
        }

        // 저장 + 렌더
        history = pruneHistory(history);
        saveHistory(history);
        renderHistory(history);

      } catch(e){
        // 에러 시에도 기존 누적은 보여주기
        const history = pruneHistory(loadHistory());
        saveHistory(history);
        renderHistory(history);
      }
    }

    loadResult();
    setInterval(loadResult, 15000); // 15초마다 자동 갱신
  </script>
</body>
</html>
