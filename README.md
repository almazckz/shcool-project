<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gemini Помощник 7 Класс</title>
  <style>
    * { box-sizing: border-box; }
    body { font-family: sans-serif; background: #0f172a; color: white; display: flex; justify-content: center; padding: 20px; }
    .container { background: #1e293b; padding: 20px; border-radius: 15px; width: 100%; max-width: 450px; }
    input, textarea { width: 100%; margin-bottom: 10px; padding: 10px; border-radius: 8px; border: none; }
    textarea { height: 100px; }
    .btn { width: 100%; padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; margin-bottom: 5px; }
    .btn-blue { background: #3b82f6; color: white; }
    .btn-gray { background: #64748b; color: white; font-size: 12px; }
    #output { background: #020617; padding: 15px; border-radius: 10px; min-height: 50px; margin-top: 10px; white-space: pre-wrap; font-size: 14px; border: 1px solid #334155; }
    .loader { color: #60a5fa; font-size: 12px; display: none; margin-bottom: 10px; }
  </style>
</head>
<body>

<div class="container">
  <h2 style="text-align:center; color:#60a5fa">Gemini 1.5 Flash</h2>
  
  <input type="password" id="apiKey" placeholder="Вставь API-ключ сюда">
  <textarea id="userInput" placeholder="Напиши вопрос (например, по физике или геометрии)"></textarea>
  
  <div id="loader" class="loader">Думаю...</div>
  
  <button onclick="askGemini()" class="btn btn-blue">🚀 Спросить нейросеть</button>
  <button onclick="showMyCode()" class="btn btn-gray">🖥️ Посмотреть код страницы</button>
  
  <div id="output">Ответ будет здесь...</div>
</div>

<script>
  async function askGemini() {
    const key = document.getElementById('apiKey').value.trim();
    const text = document.getElementById('userInput').value.trim();
    const output = document.getElementById('output');
    const loader = document.getElementById('loader');

    if (!key || !text) return alert("Заполни ключ и вопрос!");

    loader.style.display = 'block';
    output.innerText = "";

    try {
      // МАКСИМАЛЬНО ПРОСТОЙ URL
      const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${key}`;

      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: "Ты помощник ученика. Ответ дай кратко. Вопрос: " + text }] }]
        })
      });

      const data = await response.json();

      if (data.error) {
        // Если опять NOT_FOUND, значит ключ реально не видит модель 1.5 Flash
        output.innerText = "❌ Ошибка Google: " + data.error.message + "\n\nКод: " + data.error.status;
      } else if (data.candidates && data.candidates[0].content) {
        output.innerText = data.candidates[0].content.parts[0].text;
      } else {
        output.innerText = "⚠️ Что-то пошло не так. Проверь консоль (F12).";
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
    win.document.write("<pre>" + code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;") + "</pre>");
    win.document.close();
  }
</script>

</body>
</html>
