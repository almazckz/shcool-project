<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Помощник 7 Класс</title>
  <style>
    /* Основные стили */
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
    h1 { text-align: center; color: #38bdf8; margin: 0 0 20px 0; font-size: 24px; }

    /* Блок управления ключом */
    .key-section {
      background: #0f172a;
      padding: 15px;
      border-radius: 12px;
      margin-bottom: 20px;
      border: 1px solid #334155;
    }
    .input-wrapper {
      display: flex;
      gap: 8px;
      margin-bottom: 10px;
    }
    input[type="password"] {
      flex: 1;
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #1e293b;
      background: #1e293b;
      color: #38bdf8;
      outline: none;
    }
    .btn-clear-key {
      background: #ef4444;
      color: white;
      padding: 0 12px;
      border-radius: 8px;
      border: none;
      cursor: pointer;
      transition: background 0.2s;
    }
    .btn-clear-key:hover { background: #dc2626; }
    .save-option {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 11px;
      color: #94a3b8;
    }

    /* Поле ввода вопроса */
    textarea {
      width: 100%;
      height: 120px;
      background: #f8fafc;
      border-radius: 10px;
      padding: 15px;
      font-size: 16px;
      margin-bottom: 8px;
      resize: none;
      border: none;
      color: #0f172a;
      outline: none;
    }
    .btn-clear-text {
      background: #475569;
      color: white;
      border: none;
      padding: 6px 12px;
      border-radius: 6px;
      font-size: 12px;
      cursor: pointer;
      margin-bottom: 15px;
      transition: background 0.2s;
    }
    .btn-clear-text:hover { background: #334155; }

    /* Кнопки действий */
    .button-group {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 20px;
    }
    .btn-action {
      padding: 12px;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      transition: 0.2s;
    }
    .btn-main { background: #38bdf8; color: #0f172a; }
    .btn-alt { background: #fbbf24; color: #0f172a; grid-column: span 2; }
    .btn-action:hover { opacity: 0.9; transform: translateY(-1px); }
    button:disabled { opacity: 0.5; cursor: wait; transform: none; }

    /* Блок вывода ответа */
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
    .btn-copy {
      display: none; /* Скрыта по умолчанию */
      width: 100%;
      margin-top: 10px;
      background: #10b981;
      color: white;
      padding: 12px;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s;
    }
    .btn-copy:hover { background: #059669; }
    .loader { color: #38bdf8; font-style: italic; display: none; margin-bottom: 10px; font-size: 14px; }
  </style>
</head>
<body>

<div class="container">
  <h1>AI Учеба 7 Класс</h1>

  <div class="key-section">
    <div class="input-wrapper">
      <input type="password" id="apiKey" placeholder="Вставь OpenAI API Key (sk-...)" />
      <button class="btn-clear-key" onclick="deleteKeyFromMemory()" title="Стереть ключ">❌</button>
    </div>
    <div class="save-option">
      <input type="checkbox" id="saveKeyCheckbox">
      <label for="saveKeyCheckbox" style="cursor: pointer;">Запомнить ключ на этом устройстве</label>
    </div>
  </div>

  <textarea id="userInput" placeholder="Напиши тему или задачу..."></textarea>
  <button class="btn-clear-text" onclick="document.getElementById('userInput').value = ''">🧹 Очистить вопрос</button>

  <div class="button-group">
    <button onclick="runAI('solve')" class="btn-action btn-main">🔢 Решить задачу</button>
    <button onclick="runAI('explain')" class="btn-action btn-main">📖 Объяснить тему</button>
    <button onclick="runAI('summary')" class="btn-action btn-alt">📝 Сделать конспект</button>
  </div>

  <div id="status" class="loader">Думаю над ответом...</div>
  
  <div class="output-box" id="resultDisplay">Ответ появится здесь...</div>
  <button id="copyBtn" class="btn-copy" onclick="copyResultToClipboard()">📋 Скопировать решение</button>
</div>

<script>
  const keyInputField = document.getElementById('apiKey');
  const saveKeyCheck = document.getElementById('saveKeyCheckbox');
  const displayArea = document.getElementById('resultDisplay');
  const copyButton = document.getElementById('copyBtn');

  // 1. Проверяем наличие ключа в памяти при открытии сайта
  const alreadySavedKey = localStorage.getItem('_helper_key');
  if (alreadySavedKey) {
    keyInputField.value = alreadySavedKey;
    saveKeyCheck.checked = true;
  }

  // 2. Функция для принудительного удаления ключа
  function deleteKeyFromMemory() {
    if (confirm("Вы действительно хотите стереть ключ из памяти браузера?")) {
      localStorage.removeItem('_helper_key');
      keyInputField.value = '';
      saveKeyCheck.checked = false;
      alert("Ключ удален.");
    }
  }

  // 3. Основная функция логики AI
  async function runAI(mode) {
    const key = keyInputField.value.trim();
    const userText = document.getElementById('userInput').value.trim();
    const statusLabel = document.getElementById('status');
    const allButtons = document.querySelectorAll('.btn-action');

    if (!key) return alert("Пожалуйста, введите API-ключ!");
    if (!userText) return alert("Введите ваш вопрос или условие задачи.");

    // Обработка сохранения/удаления ключа на лету
    if (saveKeyCheck.checked) {
      localStorage.setItem('_helper_key', key);
    } else {
      localStorage.removeItem('_helper_key');
    }

    // Подготовка UI к запросу
    statusLabel.style.display = 'block';
    displayArea.innerText = '';
    copyButton.style.display = 'none';
    allButtons.forEach(btn => btn.disabled = true);

    let systemContext = "Ты — экспертный помощник для ученика 7 класса. Объясняй доступно.";
    if (mode === 'solve') systemContext += " Реши задачу поэтапно, в конце выдели краткий ответ.";
    if (mode === 'summary') systemContext += " Составь краткий и структурированный конспект (5-7 пунктов).";

    try {
      const apiResponse = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${key}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [
            { role: "system", content: systemContext },
            { role: "user", content: userText }
          ],
          temperature: 0.7
        })
      });

      const responseData = await apiResponse.json();

      if (!apiResponse.ok) {
        throw new Error(responseData.error ? responseData.error.message : "Ошибка при запросе к серверу");
      }

      // Вывод результата и показ кнопки копирования
      displayArea.innerText = responseData.choices[0].message.content;
      copyButton.style.display = 'block';

    } catch (err) {
      console.error(err);
      displayArea.innerText = "⚠️ Произошла ошибка:\n" + err.message + 
                              "\n\nЕсли это ошибка 'Failed to fetch', убедитесь, что ваш VPN включен.";
    } finally {
      statusLabel.style.display = 'none';
      allButtons.forEach(btn => btn.disabled = false);
    }
  }

  // 4. Функция копирования текста
  function copyResultToClipboard() {
    const textToCopy = displayArea.innerText;
    navigator.clipboard.writeText(textToCopy).then(() => {
      alert("Текст скопирован в буфер обмена!");
    }).catch(err => {
      alert("Не удалось скопировать текст: " + err);
    });
  }
</script>

</body>
</html>
