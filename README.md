<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI Помощник для учебы</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #0f172a;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      padding: 20px;
    }
    .container {
      background: #1e293b;
      padding: 25px;
      border-radius: 16px;
      width: 100%;
      max-width: 450px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.5);
    }
    h1 { text-align: center; font-size: 22px; margin-top: 0; color: #38bdf8; }
    
    .key-section {
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 1px solid #334155;
    }
    input[type="password"] {
      width: 100%;
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #334155;
      background: #020617;
      color: #38bdf8;
      font-size: 12px;
    }

    textarea {
      width: 100%;
      height: 120px;
      border-radius: 8px;
      border: none;
      padding: 12px;
      margin-bottom: 15px;
      background: #f8fafc;
      color: #0f172a;
      font-size: 16px;
      resize: vertical;
    }
    
    .buttons {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 15px;
    }
    
    button {
      padding: 12px;
      border: none;
      border-radius: 8px;
      background: #38bdf8;
      color: #020617;
      font-weight: bold;
      cursor: pointer;
      transition: opacity 0.2s;
    }
    
    button:disabled { opacity: 0.5; cursor: not-allowed; }
    
    .full-width { grid-column: span 2; background: #fbbf24; }

    .output {
      background: #020617;
      padding: 15px;
      border-radius: 8px;
      min-height: 100px;
      border: 1px solid #334155;
      line-height: 1.5;
      white-space: pre-wrap;
      font-size: 15px;
    }
  </style>
</head>
<body>

  <div class="container">
    <h1>AI Помощник 7 класс</h1>

    <div class="key-section">
      <input type="password" id="apiKey" placeholder="Вставьте ваш OpenAI API Key здесь..." />
      <p style="font-size: 10px; color: #94a3b8; margin: 5px 0 0 0;">Ключ хранится только в вашем браузере.</p>
    </div>

    <textarea id="input" placeholder="Напиши условие задачи или тему..."></textarea>

    <div class="buttons">
      <button onclick="askAI('solve')">Реши задачу</button>
      <button onclick="askAI('explain')">Объясни тему</button>
      <button onclick="askAI('summary')" class="full-width">Сделай краткий конспект</button>
    </div>

    <div class="output" id="output">Здесь появится ответ...</div>
  </div>

<script>
// Автоматически подгружаем ключ из памяти браузера, если он там есть
document.getElementById('apiKey').value = localStorage.getItem('openai_key') || '';

async function askAI(mode) {
  const key = document.getElementById('apiKey').value.trim();
  const input = document.getElementById('input').value.trim();
  const output = document.getElementById('output');

  if (!key) {
    alert("Пожалуйста, введите API ключ!");
    return;
  }

  if (!input) {
    output.innerText = "Сначала напиши вопрос.";
    return;
  }

  // Сохраняем ключ, чтобы не вводить его каждый раз
  localStorage.setItem('openai_key', key);

  output.innerText = "Нейросеть готовит ответ...";
  
  // Блокируем кнопки
  const buttons = document.querySelectorAll('button');
  buttons.forEach(b => b.disabled = true);

  let systemPrompt = "Ты добрый помощник для ученика 7 класса. Используй простые термины.";
  if (mode === 'solve') systemPrompt = "Реши школьную задачу пошагово. В конце напиши краткий ответ.";
  if (mode === 'summary') systemPrompt = "Составь краткий и понятный конспект темы. Используй списки.";

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
          { role: "system", content: systemPrompt },
          { role: "user", content: input }
        ],
        temperature: 0.7
      })
    });

    const data = await response.json();

    if (data.choices && data.choices[0]) {
      output.innerText = data.choices[0].message.content;
    } else {
      output.innerText = "Ошибка: " + (data.error ? data.error.message : "Проверьте ключ");
    }

  } catch (error) {
    output.innerText = "Ошибка связи: " + error.message;
  } finally {
    buttons.forEach(b => b.disabled = false);
  }
}
</script>
</body>
</html>
