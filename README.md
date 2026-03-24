<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hand Control Pro</title>
  <style>
    :root {
      --bg1: #050816;
      --bg2: #0b1026;
      --card: rgba(255,255,255,0.08);
      --border: rgba(255,255,255,0.14);
      --text: #eaf2ff;
      --muted: #a9b7d0;
      --accent: #22d3ee;
      --accent2: #3b82f6;
      --danger: #fb7185;
      --success: #34d399;
    }

    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    html, body {
      margin: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      font-family: "Segoe UI", system-ui, -apple-system, Arial, sans-serif;
      background:
        radial-gradient(circle at top, #13204a 0%, transparent 32%),
        radial-gradient(circle at bottom right, #0f766e33 0%, transparent 28%),
        linear-gradient(160deg, var(--bg2), var(--bg1));
      color: var(--text);
    }

    .app {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 20px;
      gap: 18px;
      text-align: center;
    }

    .title {
      font-size: clamp(28px, 6vw, 46px);
      font-weight: 800;
      letter-spacing: 0.5px;
      margin: 0;
      text-shadow: 0 0 24px rgba(34,211,238,0.22);
    }

    .subtitle {
      margin: 0;
      color: var(--muted);
      max-width: 640px;
      line-height: 1.5;
      font-size: clamp(14px, 3.5vw, 18px);
    }

    .card {
      width: min(92vw, 560px);
      background: var(--card);
      border: 1px solid var(--border);
      border-radius: 24px;
      padding: 22px;
      box-shadow: 0 18px 60px rgba(0,0,0,0.35);
      backdrop-filter: blur(14px);
    }

    .status {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      padding: 10px 14px;
      border-radius: 999px;
      background: rgba(255,255,255,0.07);
      border: 1px solid rgba(255,255,255,0.12);
      color: var(--muted);
      margin-bottom: 18px;
      font-size: 14px;
      min-height: 42px;
    }

    .dot {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background: var(--muted);
      box-shadow: 0 0 0 0 rgba(169,183,208,0.5);
    }

    .dot.active {
      background: var(--accent);
      box-shadow: 0 0 0 10px rgba(34,211,238,0.08);
    }

    .dot.ok {
      background: var(--success);
      box-shadow: 0 0 0 10px rgba(52,211,153,0.08);
    }

    .dot.bad {
      background: var(--danger);
      box-shadow: 0 0 0 10px rgba(251,113,133,0.08);
    }

    .buttons {
      display: flex;
      gap: 12px;
      justify-content: center;
      flex-wrap: wrap;
      margin-top: 14px;
    }

    button {
      border: none;
      border-radius: 16px;
      padding: 14px 22px;
      font-size: 16px;
      font-weight: 700;
      cursor: pointer;
      transition: transform 0.18s ease, box-shadow 0.18s ease, opacity 0.18s ease;
      min-width: 150px;
    }

    button:active {
      transform: scale(0.98);
    }

    #startBtn {
      color: white;
      background: linear-gradient(135deg, var(--accent2), var(--accent));
      box-shadow: 0 14px 30px rgba(59,130,246,0.3);
    }

    #stopBtn {
      color: white;
      background: linear-gradient(135deg, #ef4444, #fb7185);
      box-shadow: 0 14px 30px rgba(251,113,133,0.22);
      display: none;
    }

    .hint {
      margin-top: 14px;
      color: var(--muted);
      font-size: 13px;
      line-height: 1.5;
    }

    .cameraWrap {
      position: fixed;
      right: 16px;
      bottom: 16px;
      width: min(40vw, 240px);
      aspect-ratio: 4 / 3;
      border-radius: 18px;
      overflow: hidden;
      border: 1px solid rgba(255,255,255,0.16);
      background: rgba(0,0,0,0.3);
      box-shadow: 0 16px 40px rgba(0,0,0,0.45);
      display: none;
    }

    video {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transform: scaleX(-1);
      display: block;
    }

    .badge {
      position: fixed;
      left: 16px;
      bottom: 16px;
      padding: 10px 14px;
      border-radius: 999px;
      background: rgba(255,255,255,0.08);
      border: 1px solid rgba(255,255,255,0.14);
      color: var(--muted);
      font-size: 12px;
      backdrop-filter: blur(10px);
    }

    .error {
      color: #fecaca;
    }

    .hidden {
      display: none !important;
    }
  </style>
</head>
<body>
  <main class="app">
    <h1 class="title">Hand Control Pro</h1>
    <p class="subtitle">
      Camera start karke live preview lo. Agar permission sahi hogi aur page localhost/HTTPS par hoga, to ye smoothly chalega.
    </p>

    <section class="card">
      <div class="status" id="statusBox">
        <span class="dot" id="statusDot"></span>
        <span id="statusText">Ready to start camera</span>
      </div>

      <div class="buttons">
        <button id="startBtn">START CAMERA</button>
        <button id="stopBtn">STOP CAMERA</button>
      </div>

      <div class="hint">
        Note: camera access ke liye page ko <b>localhost</b> ya <b>HTTPS</b> par run karna hota hai.
      </div>
    </section>
  </main>

  <div class="cameraWrap" id="cameraWrap">
    <video id="cameraFeed" autoplay playsinline muted></video>
  </div>

  <div class="badge">Mobile-friendly camera demo</div>

  <script>
    const startBtn = document.getElementById("startBtn");
    const stopBtn = document.getElementById("stopBtn");
    const cameraFeed = document.getElementById("cameraFeed");
    const cameraWrap = document.getElementById("cameraWrap");
    const statusBox = document.getElementById("statusBox");
    const statusDot = document.getElementById("statusDot");
    const statusText = document.getElementById("statusText");

    let stream = null;

    function setStatus(text, type = "idle") {
      statusText.textContent = text;
      statusDot.className = "dot";

      if (type === "active") statusDot.classList.add("active");
      if (type === "ok") statusDot.classList.add("ok");
      if (type === "bad") statusDot.classList.add("bad");
      if (type === "bad") statusBox.classList.add("error");
      else statusBox.classList.remove("error");
    }

    async function startCamera() {
      try {
        setStatus("Requesting camera permission...", "active");

        if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
          throw new Error("getUserMedia is not supported in this browser.");
        }

        stream = await navigator.mediaDevices.getUserMedia({
          video: {
            facingMode: "user",
            width: { ideal: 1280 },
            height: { ideal: 720 }
          },
          audio: false
        });

        cameraFeed.srcObject = stream;
        cameraWrap.style.display = "block";
        startBtn.style.display = "none";
        stopBtn.style.display = "inline-block";
        setStatus("Camera started successfully", "ok");
      } catch (err) {
        console.error(err);
        cameraWrap.style.display = "none";
        setStatus("Camera blocked or unavailable", "bad");
        alert("Camera allow karo aur page ko localhost/HTTPS par run karo.");
      }
    }

    function stopCamera() {
      if (stream) {
        stream.getTracks().forEach(track => track.stop());
        stream = null;
      }

      cameraFeed.srcObject = null;
      cameraWrap.style.display = "none";
      startBtn.style.display = "inline-block";
      stopBtn.style.display = "none";
      setStatus("Camera stopped", "idle");
    }

    startBtn.addEventListener("click", startCamera);
    stopBtn.addEventListener("click", stopCamera);
  </script>
</body>
</html>
