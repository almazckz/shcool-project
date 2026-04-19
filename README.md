<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gemini AI Помощник</title>
  <style>
    * { box-sizing: border-box; }
    body { font-family: 'Segoe UI', sans-serif; background: #0f172a; color: white; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 15px; }
    .container { background: #1e293b; padding: 25px; border-radius: 20px; width: 100%; max-width: 500px; box-shadow: 0 25px 50px rgba(0,0,0,0.5); }
    h1 { text-align: center; color: #60a5fa; font-size: 22px; }
    input, textarea { width: 100%; background: #0f172a; border: 1px solid #334155; color: #60a5fa; padding: 12px; border-radius: 10px; margin-bottom: 15px; outline: none; }
    textarea { height: 120px; color: white; background: #1e293b; border: 1px solid #475569; }
    .actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    .btn { padding: 12px; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    .btn-blue { background: #60a5fa; color: #0f172a; }
    .btn-gray { background: #475569; color: white; grid-column: span 2; font-size: 13px; }
    #output { background: #020617; padding: 15px; border-radius: 12px; border: 1px solid #334155; margin-top: 20px; min-height: 80px; white-space: pre-wrap; font-size: 14px; }
    .loader { color: #fbbf24; font-size: 14px; display: none; margin-top: 10px; text-align: center; }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 AI Помощник 7 Класс</h1>
  <input type="password" id="apiKey" placeholder="Вставь API-ключ (AIza...)">
  <textarea id="userInput" placeholder="Напиши задачу или вопрос..."></textarea>
  
  <div class="actions">
    <button onclick="askGemini()" class="btn btn-blue">✨ Спросить нейросеть</button>
    <button onclick="showMyCode()" class="btn btn-gray">🖥️ Посмотреть мой код</button>
  </div>

  <div id="loader" class="loader">Связываюсь с Google...</div>
  <div id="output">Ответ появится здесь...</div>
</div>

<script>
  async function askGemini() {
    const key = document.getElementById('apiKey').value.trim();
    const text = document.getElementById('userInput').value.trim();
    const output = document.getElementById('output');
    const loader = document.getElementById('loader');

    if (!key || !text) return alert("Заполни ключ и вопрос!");

    loader.style.display = 'block';
    output.innerText = "Нейросеть думает...";

    try {
      // ИСПОЛЬЗУЕМ ВЕРСИЮ 1.0 PRO — ОНА САМАЯ СТАБИЛЬНАЯ ДЛЯ НОВЫХ КЛЮЧЕЙ
      const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${key}`;

      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: "Ты помощник ученика 7 класса. Отвечай кратко и понятно. Вопрос: " + text }] }]
        })
      });

      const data = await response.json();

      if (data.error) {
        output.innerText = "❌ Ошибка: " + data.error.message;
      } else if (data.candidates && data.candidates[0].content) {
        output.innerText = data.candidates[0].content.parts[0].text;
      } else {
        output.innerText = "⚠️ Ответ пустой. Попробуй другой вопрос.";
      }
    } catch (err) {
      output.innerText = "⚠️ Ошибка сети: " + err.message;
    } finally {
      loader.style.display = 'none';
    }
  }

  function showMyCode() {
    const code = document.documentElement.outerHTML;
    const win = window.open("", "_blank");
    win.document.write("<html><body style='background:#1e293b;color:white;padding:20px;'><pre>" + 
      code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;") + 
      "</pre></body></html>");
    win.document.close();
  }
</script>

</body>
</html>
