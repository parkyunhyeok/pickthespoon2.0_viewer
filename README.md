<!doctype html>
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
      margin:0 0 10px;
      font-size:22px;
      text-align:center;
    }
    .meta{
      display:flex;
      flex-wrap:wrap;
      gap:8px 12px;
      justify-content:center;
      align-items:center;
      margin-bottom:14px;
      color:#6b7280;
      font-size:12px;
    }
    .pill{
      background:#f9fafb;
      border:1px solid #e5e7eb;
      padding:6px 10px;
      border-radius:999px;
    }
    pre{
      white-space:pre-wrap;
      word-break:keep-all;
      font-size:15px;
      line-height:1.6;
      background:#f9fafb;
      padding:16px;
      border-radius:12px;
      border:1px solid #eef2f7;
      margin:0;
    }
    details{
      margin-top:12px;
    }
    summary{
      cursor:pointer;
      font-size:12px;
      color:#6b7280;
    }
    .raw{
      margin-top:8px;
      font-size:12px;
      color:#374151;
      background:#fff;
      border:1px solid #e5e7eb;
      border-radius:12px;
      padding:12px;
      white-space:pre-wrap;
      word-break:break-word;
    }
  </style>
</head>

<body>
  <div class="card">
    <h1>🏸 RKS 팀 매칭 결과</h1>

    <div class="meta">
      <span class="pill" id="pickedAt">뽑은 시간: -</span>
      <span class="pill" id="fetchedAt">불러온 시간: -</span>
      <span class="pill" id="status">상태: 대기</span>
    </div>

    <pre id="result">결과를 불러오는 중…</pre>

    <details>
      <summary>문제 해결용: 원본 응답 보기</summary>
      <div class="raw" id="raw">(원본 응답이 여기에 표시됩니다)</div>
    </details>
  </div>

  <script>
    // ✅ 너의 Apps Script Web App URL
    const API_URL =
      "https://script.google.com/macros/s/AKfycbwRcIc-LvIumOnpsmthxObSYgVgqq2obWS69VVPt9k2gBBfLHLHeQZeGB3r6rpuyVE/exec";

    const $ = (id) => document.getElementById(id);

    function s(v){ return (v === null || v === undefined) ? "" : String(v); }

    // 어떤 응답이 오든 "조편성 텍스트"를 최대한 찾아내기
    function extractTextAndTime(parsedJson){
      // 1) { ok:true, data:{...} } 형태면 data 우선
      const data = (parsedJson && typeof parsedJson === "object" && "data" in parsedJson)
        ? parsedJson.data
        : parsedJson;

      // 2) 텍스트 후보들(가능한 모든 키 대응)
      // - text (업로드 payload에 들어있던 결과 텍스트)
      // - resultText / payloadText (Code.gs에서 텍스트만 저장하도록 바꾼 경우)
      // - payload (혹시 payload에 JSON문자열/객체가 들어오는 경우)
      let text =
        s(data?.text) ||
        s(data?.resultText) ||
        s(data?.payloadText);

      // payload가 문자열(JSON)일 수도 있어서 추가 처리
      if (!text && data?.payload) {
        if (typeof data.payload === "string") {
          // payload가 JSON 문자열이면 파싱 시도 후 text 꺼내기
          try {
            const maybeObj = JSON.parse(data.payload);
            text = s(maybeObj?.text) || s(maybeObj?.resultText) || s(maybeObj?.payloadText) || "";
          } catch {
            // 그냥 문자열이면 그대로 표시(그래도 JSON 전체면 보기 싫으니 text 없으면 빈 처리)
            text = "";
          }
        } else if (typeof data.payload === "object") {
          text = s(data.payload.text) || s(data.payload.resultText) || s(data.payload.payloadText) || "";
        }
      }

      // 3) 시간 후보들
      const pickedAt =
        data?.pickedAt ||
        data?.updatedAt ||
        (data?.data && (data.data.pickedAt || data.data.updatedAt)) ||
        "";

      // \n이 "문자 두개"로 들어오면 실제 줄바꿈으로
      text = s(text).replace(/\\n/g, "\n").trim();

      return { text, pickedAt };
    }

    async function loadResult(){
      $("status").innerText = "상태: 불러오는 중…";
      $("fetchedAt").innerText = "불러온 시간: " + new Date().toLocaleString();

      try{
        const res = await fetch(API_URL + "?_=" + Date.now()); // 캐시 방지
        const rawText = await res.text();
        $("raw").innerText = rawText;

        // JSON 파싱 시도
        let parsed = null;
        try { parsed = JSON.parse(rawText); } catch { parsed = null; }

        if (!parsed) {
          $("status").innerText = "상태: 응답이 JSON이 아님";
          $("result").innerText = "결과를 불러오지 못했습니다. (응답 형식 오류)";
          $("pickedAt").innerText = "뽑은 시간: -";
          return;
        }

        // ok:false면 에러 표시
        if (parsed.ok === false) {
          $("status").innerText = "상태: 오류(" + s(parsed.error) + ")";
          $("result").innerText = "결과를 불러오지 못했습니다.";
          $("pickedAt").innerText = "뽑은 시간: -";
          return;
        }

        const { text, pickedAt } = extractTextAndTime(parsed);

        if (!text) {
          $("status").innerText = "상태: 결과 없음(또는 키 불일치)";
          $("result").innerText = "아직 팀 매칭 결과가 없거나, 결과 형식이 바뀌었습니다.";
          $("pickedAt").innerText = "뽑은 시간: -";
          return;
        }

        $("status").innerText = "상태: 정상";
        $("result").innerText = text;

        if (pickedAt) {
          $("pickedAt").innerText = "뽑은 시간: " + new Date(pickedAt).toLocaleString();
        } else {
          $("pickedAt").innerText = "뽑은 시간: -";
        }

      } catch(e){
        $("status").innerText = "상태: 네트워크/권한 오류";
        $("result").innerText =
          "결과를 불러오지 못했습니다.\n\n(가능한 원인)\n- Apps Script 웹앱 권한이 '모든 사용자(Anyone)'가 아님\n- 배포 버전 업데이트가 안 됨\n- 일시적인 네트워크 문제";
        $("pickedAt").innerText = "뽑은 시간: -";
      }
    }

    loadResult();
    setInterval(loadResult, 15000);
  </script>
</body>
</html>
