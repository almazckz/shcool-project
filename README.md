<!Shcool Bosh>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Помощник</title>
  <style>
    /* ОБЩИЙ СТИЛЬ */
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

    /* КОНТЕЙНЕР ПРИЛОЖЕНИЯ */
    .container { 
      background: #1e293b; 
      padding: 25px; 
      border-radius: 20px; 
      width: 100%; 
      max-width: 500px; 
      box-shadow: 0 25px 50px rgba(0,0,0,0.5); 
      border: 1px solid #334155;
    }

    h1 { text-align: center; color: #60a5fa; margin: 0 0 20px 0; font-size: 22px; }

    /* ПОЛЯ ВВОДА */
    input, textarea { 
      width: 100%; 
      background: #0f172a; 
      border: 1px solid #334155; 
      color: #60a5fa; 
      padding: 12px; 
      border-radius: 10px; 
      margin-bottom: 15px; 
      outline: none; 
      transition: 0.3s;
    }
    
    textarea { 
      height: 120px; 
      color: #f8fafc; 
      background: #1e293b; 
      border: 1px solid #475569; 
      resize: none; 
      font-size: 15px;
    }

    input:focus, textarea:focus { border-color: #60a5fa; }

    /* КНОПКИ */
    .actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
    
    .btn { 
      padding: 12px; 
      border: none; 
      border-radius: 10px; 
      font-weight: bold; 
      cursor: pointer; 
      transition: 0.2s; 
      font-size: 14px;
    }

    .btn-blue { background: #60a5fa; color: #0f172a; width: 100%; grid-column: span 2; margin-bottom: 5px; }
    .btn-blue:hover { background: #93c5fd; transform: scale(0.98); }

    .btn-gray { background: #475569; color: white; }
    .btn-gray:hover { background: #64748b; }

    /* ВЫВОД РЕЗУЛЬТАТА */
    #output { 
      background: #020617; 
      padding: 15px; 
      border-radius: 12px; 
      border: 1px solid #334155; 
      margin-top: 20px; 
      min-height: 100px; 
      white-space: pre-wrap; 
      font-size: 15px;
      line-height: 1.6;
    }

    .loader { 
      color: #fbbf24; 
      font-size: 14px; 
      display: none; 
      margin: 10px 0; 
      text-align: center; 
      font-style: italic;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>🚀 AI Помощник</h1>
  
  <input type="password" id="apiKey" placeholder="Вставь API-ключ (AIza...)">
  
  <textarea id="userInput" placeholder="Напиши задачу или вопрос ..."></textarea>
  
  <div class="loader" id="loader"> Нейросеть обрабатывает запрос...</div>

  <div class="actions">
    <button onclick="askGemini()" class="btn btn-blue"> Отправить запрос</button>
    <button onclick="document.getElementById('userInput').value = ''" class="btn btn-gray"> Очистить</button>
    <button onclick="showMyCode()" class="btn btn-gray"> Код</button>
  </div>

  <div id="output">Ответ появится здесь...</div>
</div>

<script>
  // Автоматическая загрузка сохраненного ключа
  const keyField = document.getElementById('apiKey');
  const savedKey = localStorage.getItem('_my_gemini_key');
  if (savedKey) keyField.value = savedKey;

  async function askGemini() {
    const key = keyField.value.trim();
    const text = document.getElementById('userInput').value.trim();
    const output = document.getElementById('output');
    const loader = document.getElementById('loader');

    if (!key || !text) return alert("Нужно ввести ключ и твой вопрос!");

    // Сохраняем ключ локально, чтобы не вводить заново
    localStorage.setItem('_my_gemini_key', key);

    loader.style.display = 'block';
    output.innerText = "Мыслю...";
    output.style.opacity = "0.5";

    try {
      // ИСПОЛЬЗУЕМ СТАБИЛЬНУЮ ВЕРСИЮ V1 И МОДЕЛЬ 1.5-FLASH
      const url = `https://generativelanguage.googleapis.
