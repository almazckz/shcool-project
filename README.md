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
      width: 400px;
      box-shadow: 0 0 20px rgba(0,0,0,0.5);
    }
    h1 { text-align: center; }
    textarea {
      width: 100%;
      height: 100px;
      border-radius: 8px;
      border: none;
      padding: 10px;
      margin-bottom: 10px;
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
      margin-bottom: 10px;
    }
    .output {
      background: #020617;
      padding: 10px;
      border-radius: 8px;
      min-height: 100px;
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
      output.innerText = "Думаю...";

      let systemPrompt = "Ты помощник для школьника. Объясняй просто.";

      if (mode === 'solve') {
        systemPrompt = "Реши задачу пошагово и просто.";
      }
      if (mode === 'summary') {
        systemPrompt = "Сделай краткое объяснение.";
      }

      try {
        const response = await fetch("https://api.openai.com/v1/chat/completions", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            "Authorization": "Bearer YOUR_API_KEY"
          },
          body: JSON.stringify({
            model: "gpt-4o-mini",
            messages: [
              { role: "system", content: systemPrompt },
              { role: "user", content: input }
            ]
          })
        });

        const data = await response.json();
        output.innerText = data.choices[0].message.content;
      } catch (error) {
        output.innerText = "Ошибка: " + error;
      }
    }
  </script>
</body>
</html>
