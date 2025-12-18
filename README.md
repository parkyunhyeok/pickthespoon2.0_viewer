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
      background:#fff;
      border-radius:16px;
      padding:20px;
      box-shadow:0 12px 30px rgba(0,0,0,.12);
    }
    h1{
      margin:0 0 12px;
      font-size:22px;
    }
    .time{
      font-size:13px;
      color:#6b7280;
      margin-bottom:12px;
    }
    pre{
      white-space:pre-wrap;
      word-break:keep-all;
      font-size:15px;
      line-height:1.6;
      background:#f9fafb;
      padding:14px;
      border-radius:12px;
    }
    .hint{
      font-size:12px;
      color:#9ca3af;
      margin-top:10px;
      text-align:center;
    }
  </style>
</head>

<body>
  <div class="card">
    <h1>🏸 RKS 팀 매칭 결과</h1>
    <div class="time" id="time">불러오는 중…</div>
    <pre id="result">잠시만 기다려주세요…</pre>
    <div class="hint">이 페이지는 자동으로 최신 결과를 불러옵니다</div>
  </div>

  <script>
    // 🔽 너의 Apps Script Web App URL 넣기
    const API_URL = "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

    async function loadResult(){
      try{
        const res = await fetch(API_URL);
        const json = await res.json();

        if (!json.ok || !json.data) {
          document.getElementById("result").innerText = "아직 뽑기 결과가 없습니다.";
          return;
        }

        const { pickedAt, resultText } = json.data;

        document.getElementById("time").innerText =
          "뽑은 시간: " + new Date(pickedAt).toLocaleString();

        document.getElementById("result").innerText =
          resultText || "결과가 비어있습니다.";
      } catch(e){
        document.getElementById("result").innerText =
          "결과를 불러오지 못했습니다.";
      }
    }

    loadResult();
    setInterval(loadResult, 15000); // 15초마다 자동 갱신
  </script>
</body>
</html>
