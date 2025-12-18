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
    <div class="hint">이 페이지는 자동으로 최신 결과를 표시합니다</div>
  </div>

  <script>
    /*********************************************************
     * 🔽 Apps Script Web App URL
     *********************************************************/
    const API_URL =
      "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

    function safe(v){
      return (v === null || v === undefined) ? "" : String(v);
    }

    async function loadResult(){
      const resultEl = document.getElementById("result");
      const timeEl   = document.getElementById("time");

      try{
        // 캐시 방지용 타임스탬프
        const res = await fetch(API_URL + "?_=" + Date.now());
        const json = await res.json();

        // Apps Script 응답 형태가 달라도 대응
        const data = json.data ?? json;

        // ✅ 우리가 원하는 것
        const resultText =
          data.resultText || data.text || "";

        const pickedAt =
          data.pickedAt || data.updatedAt || "";

        if (!resultText) {
          resultEl.innerText = "아직 팀 매칭 결과가 없습니다.";
        } else {
          // \n이 문자열로 올 경우 실제 줄바꿈 처리
          resultEl.innerText =
            safe(resultText).replace(/\\n/g, "\n").trim();
        }

        if (pickedAt) {
          timeEl.innerText =
            "뽑은 시간: " + new Date(pickedAt).toLocaleString();
        } else {
          timeEl.innerText = "";
        }

      } catch(e){
        resultEl.innerText = "결과를 불러오지 못했습니다.";
        timeEl.innerText = "";
      }
    }

    loadResult();
    setInterval(loadResult, 15000); // 15초마다 자동 갱신
  </script>
</body>
</html>
