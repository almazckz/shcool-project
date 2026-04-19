<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gemini AI Помощник 7 Класс</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', system-ui, sans-serif;
      background: #0f172a;
      color: #f8fafc;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      padding: 15px;
    }
    .container {
      background: #1e293b;
      padding: 25px;
      border-radius: 20px;
      width: 100%;
      max-width: 500px;
      box-shadow: 0 25px 50px rgba(0,0,0,0.5);
    }
    h1 { text-align: center; color: #60a5fa; margin: 0 0 20px 0; font-size: 22px; }

    /* Поле ключа */
    .key-box {
      background: #0f172a;
      padding: 15px;
      border-radius: 12px;
      border: 1px solid #334155;
      margin-bottom: 20px;
    }
    .input-row { display: flex; gap: 10px; margin-bottom: 10px; }
    input[type="password"] {
      flex: 1;
      background: #1e293b;
      border: 1px solid #334155;
      color: #60a5fa;
      padding: 10px;
      border-radius: 8px;
      outline: none;
    }
    .btn-del { background: #ef4444; color: white; border: none; border-radius: 8px; padding: 0 15px; cursor: pointer; }

    /* Поле ввода */
    textarea {
      width: 100%; height: 120px;
      background: #f8fafc; color: #0f172a;
      border-radius: 12px; padding: 15px;
      font-size: 16px; resize: none; border: none; outline: none;
    }
    .btn-clear { background: #475569; color: white; border: none; padding: 5px 12px; border-radius: 6px; font-size: 11px; margin: 8px 0 20px 0; cursor: pointer; }

    /* Кнопки */
    .actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
    .btn { padding: 12px; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    .btn-blue { background: #60a5fa; color: #0f172a; }
    .btn-yellow { background: #fbbf24; color: #0f172a; grid-column: span 2; }
    .btn-gray { background: #64748b; color: white; grid-column: span 2; margin-top: 5px; font-size: 13px; }
    
    .btn:hover { opacity: 0.9; transform: scale(0.98); }
    .btn:disabled { opacity: 0.5; }

    /* Результат */
    .result-area {
      background: #020617; padding: 15px; border-radius: 12px;
      border: 1px solid #334155; min-height: 100px;
      font-size: 15px; line-height: 1.6; white-space: pre-wrap;
    }
    .btn-copy { display: none; width: 100%; margin-top: 10px; background: #10b981; color: white; padding: 12px; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }
    .loader { color: #60a5fa; font-size: 14px; font-style: italic; display: none; margin-bottom: 10px; }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 Gemini Помощник (v2.0)</h1>

  <div class="key-box">
    <div class="input-row">
      <input type="password" id="apiKey" placeholder="Вставь API Key (AIza...)">
      <button class="btn-del" onclick="resetKey()">❌</button>
    </div>
    <div style="font-size: 11px; color: #94a3b8;">
      <input type="checkbox" id="saveCheck" checked> <label for="saveCheck" style="cursor:pointer">Запомнить ключ</label>
    </div>
  </div>

  <textarea id="userInput" placeholder="Напиши задачу или тему..."></textarea>
  <button class="btn-clear" onclick="document.getElementById('userInput').value = ''">🧹 Очистить вопрос</button>

  <div class="actions">
    <button onclick="askGemini('solve')" class="btn btn-blue">🔢 Решить задачу</button>
    <button onclick="askGemini('explain')" class="btn btn-blue">📖 Объяснить тему</button>
    <button onclick="askGemini('summary')" class="btn btn-yellow">📝 Сделать конспект</button>
    <button onclick="showMyCode()" class="btn btn-gray">🖥️ Посмотреть код страницы</button>
  </div>

  <div id="loader" class="loader">Связываюсь с Google...</div>
  <div class="result-area" id="output">Ответ появится здесь...</div>
  <button id="copyBtn" class="btn-copy" onclick="copyToClipboard()">📋 Скопировать ответ</button>
</div>

<script>
  const keyField = document.getElementById('apiKey');
  const saveCheck = document.getElementById('saveCheck');
  const outputDiv = document.getElementById('output');
  const copyBtn = document.getElementById('copyBtn');
  const loader = document.getElementById('loader');

  // Загрузка ключа
  const saved = localStorage.getItem('_gemini_api_key');
  if (saved) keyField.value = saved;

  function resetKey() {
    if(confirm("Удалить сохраненный ключ?")) {
      localStorage.removeItem('_gemini_api_key');
      keyField.value = '';
    }
  }

  // Главная функция с автоматическим подбором рабочего API
  async function askGemini(mode) {
    const key = keyField.value.trim();
    const text = document.getElementById('userInput').value.trim();
    
    if (!key || !text) return alert("Заполни все поля!");

    if (saveCheck.checked) {
      localStorage.setItem('_gemini_api_key', key);
    }

    loader.style.display = 'block';
    outputDiv.innerText = 'Ищу рабочий канал связи...';
    copyBtn.style.display = 'none';

    let instruction = "Ты — экспертный помощник ученика 7 класса. ";
    if (mode === 'solve') instruction += "Реши задачу пошагово.";
    if (mode === 'summary') instruction += "Сделай краткий конспект.";
    if (mode === 'explain') instruction += "Объясни тему максимально просто.";
    
    // Список адресов для обхода ошибки NOT_FOUND
    const urls = [
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${key}`,
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${key}`,
      `https://generativelanguage.googleapis.com/v1/models/gemini-1.5-flash:generateContent?key=${key}`,
      `https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${key}`
    ];

    let lastError = "";

    for (let url of urls) {
      try {
        const response = await fetch(url, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            contents: [{ parts: [{ text: instruction + "\n\nВопрос: " + text }] }]
          })
        });

        const data = await response.json();

        if (data.error) {
          lastError = data.error.message;
          continue; // Пробуем следующий URL
        }

        if (data.candidates && data.candidates[0].content) {
          const aiText = data.candidates[0].content.parts[0].text;
          outputDiv.innerText = aiText;
          copyBtn.style.display = 'block';
          loader.style.display = 'none';
          return; // Успешно выходим
        }
      } catch (err) {
        lastError = err.message;
      }
    }

    // Если всё сломалось
    outputDiv.innerText = "⚠️ Ошибка подключения.\nПричина: " + lastError + 
                          "\n\nСовет: Если ошибка NOT_FOUND, попробуй создать НОВЫЙ ключ в AI Studio (в новом проекте).";
    loader.style.display = 'none';
  }

  function copyToClipboard() {
    navigator.clipboard.writeText(outputDiv.innerText).then(() => {
      alert("Скопировано!");
    });
  }

  // Функция просмотра кода
  function showMyCode() {
    const code = document.documentElement.outerHTML;
    const codeWindow = window.open("", "_blank");
    codeWindow.document.write("<html><head><title>Source Code</title></head><body style='background: #1e293b; color: #f8fafc; padding: 20px; font-family: monospace;'>");
    codeWindow.document.write("<h2>Твой HTML/JS Код:</h2>");
    codeWindow.document.write("<pre style='white-space: pre-wrap; background: #0f172a; padding: 15px; border-radius: 10px; border: 1px solid #334155;'>" + 
      code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;") + 
      "</pre></body></html>");
    codeWindow.document.close();
  }
</script>

</body>
</html>
