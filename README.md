<!Bosh #2>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Помощник для учебы</title>
  <style>
    /* Глобальные настройки для аккуратного дизайна */
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
      padding: 24px;
      border-radius: 16px;
      width: 100%;
      max-width: 480px;
      box-shadow: 0 20px 50px rgba(0,0,0,0.3);
    }

    h1 { 
      text-align: center; 
      font-size: 24px; 
      margin-bottom: 20px; 
      color: #38bdf8;
    }

    /* Секция ключа */
    .key-section {
      background: #0f172a;
      padding: 12px;
      border-radius: 10px;
      margin-bottom: 20px;
      border: 1px solid #334155;
    }

    .key-section label {
      display: block;
      font-size: 11px;
      color: #94a3b8;
      margin-bottom: 5px;
      text-transform: uppercase;
    }

    input[type="password"] {
      width: 100%;
      padding: 8px;
      border-radius: 6px;
      border: 1px solid #1e293b;
      background: #1e293b;
      color: #38bdf8;
      outline: none;
    }

    /* Поле ввода вопроса */
    textarea {
      width: 100%;
      height: 120px;
      background: #f8fafc;
      color: #0f172a;
      border: none;
      border-radius: 10px;
      padding: 15px;
      font-size: 16px;
      margin-bottom: 15px;
      resize: none;
      outline: none;
    }

    /* Кнопки управления */
    .button-group {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 20px;
    }

    button {
      padding: 12px;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      transition: all 0.2s;
      font-size: 14px;
    }

    .btn-main { background: #38bdf8; color: #0f172a; }
    .btn-alt { background: #fbbf24; color: #0f172a; grid-column: span 2; }
    
    button:hover { opacity: 0.9; transform: translateY(-1px); }
    button:active { transform: translateY(0); }
    button:disabled { background: #475569; cursor: wait; opacity: 0.7; }

    /* Окно ответа */
    .output-box {
      background: #020617;
      padding: 15px;
      border-radius: 10px;
      min-height: 100px;
      border: 1px solid #334155;
      font-size: 15px;
      line-height: 1.6;
      white-space: pre-wrap;
    }

    .loader { color: #38bdf8; font-style: italic; display: none; }
  </style>
</head>
<body>

<div class="container">
  <h1>AI Помошник</h1>

  <div class="key-section">
  <label>OpenAI API Key</label>
  <input type="password" id="apiKey" placeholder="sk-..." />
  <div style="margin-top: 8px; display: flex; align-items: center; gap: 8px;">
    <input type="checkbox" id="saveKey" style="cursor: pointer;">
    <label for="saveKey" style="font-size: 11px; color: #94a3b8; cursor: pointer; text-transform: none;">
      Запомнить ключ на этом устройстве
    </label>
  </div>
</div>
    <label>OpenAI API Key</label>
    <input type="password" id="apiKey" placeholder="sk-..." />
  </div>

  <textarea id="userInput" placeholder="Напиши тему или задачу..."></textarea>

  <div class="button-group">
    <button id="solveBtn" onclick="runAI('solve')" class="btn-main">Решить задачу</button>
    <button id="explainBtn" onclick="runAI('explain')" class="btn-main">Объяснить тему</button>
    <button id="summaryBtn" onclick="runAI('summary')" class="btn-alt">Сделать краткий конспект</button>
  </div>

  <div id="status" class="loader">Ищу ответ в базе знаний...</div>
  <div class="output-box" id="resultDisplay">Ваш ответ будет здесь...</div>
</div>

<script>
  // Загружаем ключ при старте страницы
  const keyInput = document.getElementById('apiKey');
  keyInput.value = localStorage.getItem('_helper_key') || '';

async function runAI(mode) {
    const key = keyInput.value.trim();
    const prompt = document.getElementById('userInput').value.trim();
    const status = document.getElementById('status');
    const buttons = document.querySelectorAll('button');

    if (!key) return alert("Нужен API-ключ!");
    if (!prompt) return alert("Напиши вопрос!");

    // Управление сохранением: если галочка стоит — сохраняем, если нет — удаляем
    if (saveCheckbox.checked) {
      localStorage.setItem('_helper_key', key);
    } else {
      localStorage.removeItem('_helper_key');
    }

    status.style.display = 'block';
    resultDisplay.innerText = '';
    
    // Блокируем кнопки на время запроса
    buttons.forEach(b => b.disabled = true);

    let systemMsg = "Ты — помощник для ученика 7 класса.";
    if (mode === 'solve') systemMsg += " Реши задачу по шагам.";
    if (mode === 'summary') systemMsg += " Сделай краткий конспект.";

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
            { role: "system", content: systemMsg },
            { role: "user", content: prompt }
          ]
        })
      });

      const data = await response.json();
      
      if (!response.ok) {
        // Если API вернул ошибку (например, кончились деньги)
        throw new Error(data.error ? data.error.message : "Ошибка API");
      }

      // Выводим результат
      resultDisplay.innerText = data.choices[0].message.content;

    } catch (err) {
      // Обработка сетевых ошибок (включая Failed to fetch)
      resultDisplay.innerText = "⚠️ Ошибка:\n" + err.message + 
                                 "\n\nПроверь VPN или баланс ключа.";
    } finally {
      // В любом случае убираем надпись "Думаю" и включаем кнопки
      status.style.display = 'none';
      buttons.forEach(b => b.disabled = false);
    }
  }
    const key = keyInput.value.trim();
    const prompt = document.getElementById('userInput').value.trim();
    const status = document.getElementById('status');
    const buttons = document.querySelectorAll('button');

    if (!key) return alert("Нужен API-ключ!");
    if (!prompt) return alert("Напиши вопрос!");

    // Управление сохранением: если галочка стоит — сохраняем, если нет — удаляем
    if (saveCheckbox.checked) {
      localStorage.setItem('_helper_key', key);
    } else {
      localStorage.removeItem('_helper_key');
    }

    status.style.display = 'block';
    resultDisplay.innerText = '';
    
    // Блокируем кнопки на время запроса
    buttons.forEach(b => b.disabled = true);

    let systemMsg = "Ты — помощник для ученика 7 класса.";
    if (mode === 'solve') systemMsg += " Реши задачу по шагам.";
    if (mode === 'summary') systemMsg += " Сделай краткий конспект.";

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
            { role: "system", content: systemMsg },
            { role: "user", content: prompt }
          ]
        })
      });

      const data = await response.json();
      
      if (!response.ok) {
        // Если API вернул ошибку (например, кончились деньги)
        throw new Error(data.error ? data.error.message : "Ошибка API");
      }

      // Выводим результат
      resultDisplay.innerText = data.choices[0].message.content;

    } catch (err) {
      // Обработка сетевых ошибок (включая Failed to fetch)
      resultDisplay.innerText = "⚠️ Ошибка:\n" + err.message + 
                                 "\n\nПроверь VPN или баланс ключа.";
    } finally {
      // В любом случае убираем надпись "Думаю" и включаем кнопки
      status.style.display = 'none';
      buttons.forEach(b => b.disabled = false);
    }
  }
    const key = keyInput.value.trim();
    const prompt = document.getElementById('userInput').value.trim();
    const status = document.getElementById('status');
    const result = document.getElementById('resultDisplay');
    const buttons = document.querySelectorAll('button');

    if (!key) return alert("Вставь API-ключ!");
    if (!prompt) return alert("Сначала напиши вопрос.");

    // Сохраняем ключ
    localStorage.setItem('_helper_key', key);

    // Подготовка интерфейса
    status.style.display = 'block';
    result.innerText = '';
    buttons.forEach(b => b.disabled = true);

    let systemMsg = "Ты — помощник для ученика 7 класса. Ответ должен быть простым и понятным.";
    if (mode === 'solve') systemMsg = "Реши задачу по физике, математике или химии. Распиши решение по действиям.";
    if (mode === 'summary') systemMsg = "Сделай краткую выжимку по теме в виде списка из 5 пунктов.";

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
            { role: "system", content: systemMsg },
            { role: "user", content: prompt }
          ],
          temperature: 0.7
        })
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error ? data.error.message : "Ошибка API");
      }

      result.innerText = data.choices[0].message.content;

    } catch (err) {
      console.error(err);
      result.innerText = "⚠️ Ошибка:\n" + err.message + 
                         "\n\nЕсли написано 'Failed to fetch', проверь VPN!";
    } finally {
      status.style.display = 'none';
      buttons.forEach(b => b.disabled = false);
    }
  }
</script>

</body>
</html>
