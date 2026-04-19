<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gemini AI Помощник 7 Класс</title>
  <style>
    /* Основное оформление */
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

    /* Кнопки управления */
    .actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
    .btn { padding: 12px; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    .btn-blue { background: #60a5fa; color: #0f172a; }
    .btn-yellow { background: #fbbf24; color: #0f172a; grid-column: span 2; }
    .btn-gray { background: #64748b; color: white; grid-column: span 2; margin-top: 5px; font-size: 13px; }
    
    .btn:hover { opacity: 0.9; transform: scale(0.98); }
    .btn:disabled { opacity: 0.5; cursor: not-allowed; }

    /* Вывод результата */
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
  <h1>🚀 Gemini Помощник (v1)</h1>

  <div class="key-box">
    <div class="input-row">
      <input type="password" id="apiKey" placeholder="Вставь Google API Key (AIza...)">
      <button class="btn-del" onclick="resetKey()" title="Стереть ключ">❌</button>
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

  <div id="loader" class="loader">Связываюсь с Google Gemini...</div>
  <div class="result-area" id="output">Ответ появится здесь...</div>
  <button id="copyBtn" class="btn-copy" onclick="copyToClipboard()">📋 Скопировать ответ</button>
</div>

<script>
  // Элементы
  const keyField = document.getElementById('apiKey');
  const saveCheck = document.getElementById('saveCheck');
  const outputDiv = document.getElementById('output');
  const copyBtn = document.getElementById('copyBtn');
  const loader = document.getElementById('loader');

  // Загрузка ключа из памяти
  const savedKey = localStorage.getItem('_gemini_api_key');
  if (savedKey) keyField.value = savedKey;

  // Очистка ключа
  function resetKey() {
    if(confirm("Удалить сохраненный ключ?")) {
      localStorage.removeItem('_gemini_api_key');
      keyField.value = '';
    }
  }

  // Главная функция
async function askGemini(mode) {
    const key = keyField.value.trim();
    const text = document.getElementById('userInput').value.trim();
    
    if (!key || !text) return alert("Заполни ключ и напиши вопрос!");

    if (saveCheck.checked) {
      localStorage.setItem('_gemini_api_key', key);
    } else {
      localStorage.removeItem('_gemini_api_key');
    }

    loader.style.display = 'block';
    outputDiv.innerText = 'Связываюсь с сервером...';
    copyBtn.style.display = 'none';

    let instruction = "Ты — помощник ученика 7 класса. ";
    if (mode === 'solve') instruction += "Реши задачу пошагово.";
    if (mode === 'summary') instruction += "Сделай краткий конспект.";
    if (mode === 'explain') instruction += "Объясни тему максимально просто.";
    
    try {
      // ИСПОЛЬЗУЕМ v1beta И gemini-1.5-flash-latest
      const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${key}`;
      
      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: instruction + "\n\nВопрос: " + text }] }]
        })
      });

      const data = await response.json();
      
      if (data.error) {
        // Если ошибка 404 (модель не найдена), пробуем запасной вариант
        if (data.error.status === "NOT_FOUND") {
            throw new Error("Модель Flash не найдена. Попробуй в AI Studio проверить, создавал ли ты API Key для 'Gemini 1.5 Flash'.");
        }
        throw new Error(`[${data.error.status}]: ${data.error.message}`);
      }

      if (data.candidates && data.candidates[0].content) {
        const aiText = data.candidates[0].content.parts[0].text;
        outputDiv.innerText = aiText;
        copyBtn.style.display = 'block';
      } else {
        throw new Error("Ответ пустой. Возможно, запрос заблокирован фильтрами.");
      }

    } catch (err) {
      outputDiv.innerText = "⚠️ Ошибка: " + err.message + "\n\nСовет: Убедись, что при создании ключа в AI Studio ты нажал 'Create API key in new project'.";
    } finally {
      loader.style.display = 'none';
    }
  }
    // Сохранение
    if (saveCheck.checked) {
      localStorage.setItem('_gemini_api_key', key);
    } else {
      localStorage.removeItem('_gemini_api_key');
    }

    loader.style.display = 'block';
    outputDiv.innerText = 'Думаю...';
    copyBtn.style.display = 'none';

    let instruction = "Ты — помощник ученика 7 класса. ";
    if (mode === 'solve') instruction += "Реши задачу пошагово.";
    if (mode === 'summary') instruction += "Сделай краткий конспект.";
    if (mode === 'explain') instruction += "Объясни тему максимально просто.";
    
    try {
      // URL стабильной версии v1
      const url = `https://generativelanguage.googleapis.com/v1/models/gemini-1.5-flash:generateContent?key=${key}`;
      
      const response = await fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: instruction + "\n\nВопрос: " + text }] }]
        })
      });

      const data = await response.json();
      
      if (data.error) throw new Error(`[${data.error.status}]: ${data.error.message}`);

      if (data.candidates && data.candidates[0].content) {
        const aiText = data.candidates[0].content.parts[0].text;
        outputDiv.innerText = aiText;
        copyBtn.style.display = 'block';
      } else {
        throw new Error("Ответ пустой. Возможно, сработали фильтры безопасности.");
      }

    } catch (err) {
      outputDiv.innerText = "⚠️ Ошибка: " + err.message + "\n\nПроверь свой API Key!";
    } finally {
      loader.style.display = 'none';
    }
  }

  // Копирование
  function copyToClipboard() {
    navigator.clipboard.writeText(outputDiv.innerText).then(() => {
      alert("Текст скопирован!");
    });
  }

  // Просмотр кода
  function showMyCode() {
    const code = document.documentElement.outerHTML;
    const codeWindow = window.open("", "_blank");
    codeWindow.document.write("<html><head><title>Source Code</title></head><body style='background: #1e293b; color: #f8fafc; padding: 20px; font-family: monospace;'>");
    codeWindow.document.write("<h2>Твой чистый код приложения:</h2>");
    codeWindow.document.write("<pre style='white-space: pre-wrap; background: #0f172a; padding: 15px; border-radius: 10px; border: 1px solid #334155;'>" + 
      code.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;") + 
      "</pre></body></html>");
    codeWindow.document.close();
  }
</script>

</body>
</html>
