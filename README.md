<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>AI Помощник для учебы</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #0f172a;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }
    .container {
      background: #1e293b;
      padding: 20px;
      border-radius: 12px;
      width: 420px;
      box-shadow: 0 0 20px rgba(0,0,0,0.5);
    }
    h1 { text-align: center; font-size: 20px; }
    textarea {
      width: 100%;
      height: 100px;
      border-radius: 8px;
      border: none;
      padding: 10px;
      margin-bottom: 10px;
      resize: none;
    }
    button {
      width: 100%;
      padding: 10px;
      border: none;
      border-radius: 8px;
      background: #38bdf8;
      color: black;
      font-weight: bold;
      cursor: pointer;
      margin-bottom: 8px;
    }
    .output {
      background: #020617;
      padding: 10px;
      border-radius: 8px;
      min-height: 120px;
      white-space: pre-wrap;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>AI Помощник</h1>

    <textarea id="input" placeholder="Введите вопрос..."></textarea>

    <button onclick="askAI('explain')">Объясни тему</button>
    <button onclick="askAI('solve')">Реши задачу</button>
    <button onclick="askAI('summary')">Сделай кратко</button>

    <div class="output" id="output">Ответ появится здесь...</div>
  </div>

<script>
async function askAI(mode) {
  const input = document.getElementById('input').value;
  const output = document.getElementById('output');

  if (!input.trim()) {
    output.innerText = "Введи вопрос.";
    return;
  }

  output.innerText = "Думаю...";

  let systemPrompt = "Ты помощник для школьника 7 класса. Объясняй просто.";

  if (mode === 'solve') {
    systemPrompt = "Решай задачу пошагово и объясняй просто.";
  }
  if (mode === 'summary') {
    systemPrompt = "Сделай краткое и понятное объяснение.";
  }

  try {
    // 🔥 ПРАВИЛЬНЫЙ вариант (через backend proxy)
    const response = await fetch("/api/ask", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        input: input,
        system: systemPrompt
      })
    });

    const data = await response.json();

    if (data.answer) {
      output.innerText = data.answer;
    } else {
      output.innerText = "Ошибка сервера: " + JSON.stringify(data);
    }

  } catch (error) {
    // fallback режим (чтобы не ломалось)
    output.innerText =
      "Нет подключения к серверу AI.\n\n" +
      "Режим демо:\n" +
      "Ваш вопрос: " + input + "\n\n" +
      "(Чтобы работал настоящий AI, нужен backend сервер через Vercel/Netlify)";
  }
}
</script>
</body>
</html>

