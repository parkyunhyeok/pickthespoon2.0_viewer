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
  const API_URL = "https://script.google.com/macros/s/AKfycbzL71rMBzJvauXeFyFz2AuHXILUbQxO3IosQkMDySF3LB8LIXp3OGPc7r88Zw6zSdmh/exec";

  function safeText(v){
    return (v === null || v === undefined) ? "" : String(v);
  }

  async function loadResult(){
    const timeEl = document.getElementById("time");
    const resultEl = document.getElementById("result");

    try{
      const res = await fetch(API_URL + "?t=" + Date.now()); // 캐시 방지
      const json = await res.json();

      if (!json || json.ok === false) {
        resultEl.innerText = "결과를 불러오지 못했습니다.";
        timeEl.innerText = "";
        return;
      }

      // ✅ Apps Script가 2가지 형태 중 무엇을 주든 대응
      // 1) { ok:true, data:{ pickedAt, resultText } }  (텍스트만 저장하도록 바꾼 경우)
      // 2) { updatedAt, ... , text:"A조...\n휴식자..." } 또는 { ok:true, data:{..., text:"..."} }
      const data = json.data ?? json;

      const pickedAt = data.pickedAt || data.updatedAt || "";
      const resultText = data.resultText || data.text || "";

      if (!resultText) {
        resultEl.innerText = "아직 뽑기 결과가 없습니다.";
      } else {
        // 혹시 \n이 문자로 들어오면 실제 줄바꿈으로 변환
        resultEl.innerText = safeText(resultText).replace(/\\n/g, "\n").trim();
      }

      if (pickedAt) {
        timeEl.innerText = "뽑은 시간: " + new Date(pickedAt).toLocaleString();
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
