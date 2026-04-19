<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Помощник 7 Класс</title>
  <style>
    /* Настройки дизайна */
    * { box-sizing: border-box; }
    
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
      box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
    }

    h1 { 
      text-align: center; 
      color: #38bdf8; 
      margin-top: 0;
      font-size: 22px;
    }

    /* Блок ключа */
    .key-box {
      background: #0f172a;
      padding: 15px;
      border-radius: 12px;
      border: 1px solid #334155;
      margin-bottom: 20px;
    }

    .input-row {
      display: flex;
      gap: 10px;
      margin-bottom: 10px;
    }

    input[type="password"] {
      flex: 1;
      background: #1e293b;
      border: 1px solid #334155;
      color: #38bdf8;
      padding: 10px;
      border-radius: 8px;
      outline: none;
    }

    .btn-delete {
      background: #ef4444;
      color: white;
      border: none;
      border-radius: 8px;
      padding: 0 15px;
      cursor: pointer;
    }

    .checkbox-row {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 12px;
      color: #94a3b8;
    }

    /* Поле ввода вопроса */
    textarea {
      width: 100%;
      height: 120px;
      background: #f1f5f9;
      color: #0f172a;
      border: none;
      border-radius: 12px;
      padding: 15px;
      font-size: 16px;
      resize: none;
      outline: none;
    }

    .btn-clear-input {
      background: #475569;
      color: white;
      border: none;
      padding: 5px 12px;
      border-radius: 6px;
      font-size: 12px;
      margin: 8px 0 20px 0;
      cursor: pointer;
    }

    /* Кнопки действий */
    .actions {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 20px;
    }

    .btn {
      padding: 12px;
      border: none;
      border-radius: 10px;
      font-weight: bold;
      cursor: pointer;
      transition: 0.2s;
    }

    .btn-blue { background: #38bdf8; color: #0f172a; }
    .btn-yellow { background: #fbbf24; color: #0f172a; grid-column: span 2; }
    
    .btn:hover { opacity: 0.9; transform: scale(0.98); }
    .btn:disabled { background: #475569; cursor: not-allowed; }

    /* Окно результата */
    .result-area {
      background: #020617;
      padding: 15px;
      border-radius: 12px;
      border: 1px solid #334155;
      min-height: 100px;
      font-size: 15px;
      line-height: 1.5;
      white-space: pre-wrap;
    }

    .btn-copy {
      display: none;
      width: 100%;
      margin-top: 10px;
      background: #10b981;
      color: white;
      padding: 12px;
      border: none;
      border-radius: 10px;
      font-weight: bold;
      cursor: pointer;
    }

    .loading-text {
      color: #38bdf8;
      font-size: 14px;
      font-style: italic;
      margin-bottom: 10px;
      display: none;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 AI Помощник 7 Класс</h1>

  <div class="key-box">
    <div class="input-row">
      <input type="password" id="apiKey" placeholder="Вставь свой API Key (sk-...)">
      <button class="btn-delete" onclick="fullResetKey()" title="Стереть всё">❌</button>
    </div>
    <div class="checkbox-row">
      <input type="checkbox" id="saveCheck">
      <label for="saveCheck" style="cursor:pointer">Запомнить ключ на этом устройстве</label>
    </div>
  </div>

  <textarea id="userInput" placeholder="Напиши задачу или тему..."></textarea>
  <button class="btn-clear-input" onclick="document.getElementById('userInput').value = ''">🧹 Очистить вопрос</button>

  <div class="actions">
    <button onclick="askAI('solve')" class="btn btn-blue">🔢 Решить задачу</button>
    <button onclick="askAI('explain')" class="btn btn-blue">📖 Объяснить тему</button>
    <button onclick="askAI('summary')" class="btn btn-yellow">📝 Сделать конспект</button>
  </div>

  <div id="loader" class="loading-text">Нейросеть думает...</div>

  <div class="result-area" id="output">Ваш ответ будет здесь...</div>
  <button id="copyBtn" class="btn-copy" onclick="copyToClipboard()">📋 Скопировать решение</button>
</div>

<script>
  const keyField = document.getElementById('apiKey');
  const saveCheck = document.getElementById('saveCheck');
  const outputDiv = document.getElementById('output');
  const copyBtn = document.getElementById('copyBtn');
  const loader = document.getElementById('loader');

  // 1. При загрузке страницы проверяем память
  const saved = localStorage.getItem('_user_api_key');
  if (saved) {
    keyField.value = saved;
    saveCheck.checked = true;
  }

  // 2. Функция полной очистки ключа
  function fullResetKey() {
    if (confirm("Удалить ключ из памяти навсегда?")) {
      localStorage.removeItem('_user_api_key');
      keyField.value = '';
      saveCheck.checked = false;
    }
  }

  // 3. Основная логика работы с AI
  async function askAI(type) {
    const key = keyField.value.trim();
    const text = document.getElementById('userInput').value.trim();
    const buttons = document.querySelectorAll('.btn');

    if (!key) return alert("Сначала вставь API-ключ!");
    if (!text) return alert("Напиши что-нибудь в поле ввода!");

    // Управление сохранением
    if (saveCheck.checked) {
      localStorage.setItem('_user_api_key', key);
    } else {
      localStorage.removeItem('_user_api_key');
    }

    // Интерфейс в режим ожидания
    loader.style.display = 'block';
    outputDiv.innerText = '';
    copyBtn.style.display = 'none';
    buttons.forEach(b => b.disabled = true);

    let systemRole = "Ты — крутой помощник для ученика 7 класса. ";
    if (type === 'solve') systemRole += "Реши задачу по шагам. В конце напиши ответ.";
    if (type === 'summary') systemRole += "Сделай краткий конспект темы по пунктам.";
    if (type === 'explain') systemRole += "Объясни тему максимально просто, как другу.";

    try {
      const response = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${key}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [
            { role: "system", content: systemRole },
            { role: "user", content: text }
          ],
          temperature: 0.7
        })
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error ? data.error.message : "Ошибка API");
      }

      // Выводим ответ
      outputDiv.innerText = data.choices[0].message.content;
      copyBtn.style.display = 'block';

    } catch (err) {
      outputDiv.innerText = "⚠️ Проблема:\n" + err.message + 
                            "\n\nЕсли это 'Failed to fetch' — проверь свой VPN!";
    } finally {
      loader.style.display = 'none';
      buttons.forEach(b => b.disabled = false);
    }
  }

  // 4. Функция копирования
  function copyToClipboard() {
    const text = outputDiv.innerText;
    navigator.clipboard.writeText(text).then(() => {
      alert("Готово! Текст скопирован.");
    });
  }
</script>

</body>
</html>
