<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>MERI</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #f0f0f0;
    }

    .container {
      max-width: 600px;
      margin: auto;
      height: 100vh;
      display: flex;
      flex-direction: column;
      background: #fff;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }

    h2 {
      text-align: center;
      background: #0d6efd;
      color: white;
      margin: 0;
      padding: 15px;
      font-size: 22px;
    }

    #chat {
      flex: 1;
      overflow-y: auto;
      padding: 15px;
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .msg {
      padding: 12px 16px;
      border-radius: 20px;
      max-width: 75%;
      word-wrap: break-word;
      position: relative;
      opacity: 0;
      transform: translateY(10px);
      animation: fadeIn 0.3s forwards;
    }

    .user {
      background: #d0ebff;
      align-self: flex-end;
    }

    .bot {
      background: #e6f7e6;
      align-self: flex-start;
    }

    .typing {
      font-style: italic;
      color: #888;
      align-self: flex-start;
    }

    @keyframes fadeIn {
      to { opacity: 1; transform: translateY(0); }
    }

    .input-area {
      display: flex;
      padding: 12px;
      border-top: 1px solid #ccc;
      gap: 10px;
      background: #f8f8f8;
    }

    #input {
      flex: 1;
      padding: 14px;
      font-size: 16px;
      border-radius: 10px;
      border: 1px solid #ccc;
    }

    button {
      padding: 14px 20px;
      font-size: 16px;
      border: none;
      border-radius: 10px;
      background: #28a745;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }

    button:active {
      transform: scale(0.97);
    }

  </style>
</head>

<body>

<div class="container">

  <h2>MERI</h2>

  <div id="chat"></div>

  <div class="input-area">
    <input type="text" id="input" placeholder="Escribe tu pregunta...">
    <button onclick="enviarPregunta()">Enviar</button>
  </div>

</div>

<script>
const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbxlwMz2bXBY_AJOseJ5bRbCr5gsPiMO7zETRFvkTEY1mkfl2y1h6IQeyj3E3FUFPKT5/exec";


function getQueryParam(param) {
  const urlParams = new URLSearchParams(window.location.search);
  return urlParams.get(param) || "";
}
const USER_EMAIL = getQueryParam("EMAIL");


let historial = JSON.parse(localStorage.getItem("chatHistory") || "[]");
const chatDiv = document.getElementById("chat");


historial.forEach(msg => agregarMensaje(msg.texto, msg.tipo));

function agregarMensaje(texto, tipo) {
  const div = document.createElement("div");
  div.className = tipo === "typing" ? "typing" : "msg " + tipo;
  div.innerText = texto;
  chatDiv.appendChild(div);
  chatDiv.scrollTop = chatDiv.scrollHeight;

  if(tipo !== "typing") {
    historial.push({texto, tipo});
    localStorage.setItem("chatHistory", JSON.stringify(historial));
  }
}

async function enviarPregunta() {
  const input = document.getElementById("input");
  const pregunta = input.value.trim();
  if (!pregunta) return;

  agregarMensaje("Tú: " + pregunta, "user");
  input.value = "";

    const typingDiv = document.createElement("div");
  typingDiv.className = "typing";
  typingDiv.innerText = "IA está escribiendo...";
  chatDiv.appendChild(typingDiv);
  chatDiv.scrollTop = chatDiv.scrollHeight;

  try {
    const formData = new FormData();
    formData.append("data", JSON.stringify({
      ID: Date.now().toString(),
      COMANDOS: pregunta,
      EMAIL: USER_EMAIL
    }));

    const response = await fetch(WEB_APP_URL, {
      method: 'POST',
      body: formData
    });

    const text = await response.text();

    let data;
    try { data = JSON.parse(text); }
    catch(e) {
      typingDiv.innerText = "Error: " + text;
      return;
    }

   
    typingDiv.remove();
    agregarMensaje("IA: " + data.RESPUESTA_BOT, "bot");

  } catch (error) {
    typingDiv.innerText = "Error conexión";
    console.log(error);
  }
}

document.getElementById("input").addEventListener("keydown", function(e) {
  if (e.key === "Enter") enviarPregunta();
});

</script>
</body>
</html>
