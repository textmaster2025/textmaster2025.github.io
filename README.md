# textmaster2025.github.io
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Сборщик текстов</title>
  <style>
    body { font-family: sans-serif; padding: 20px; background: #f9f9f9; }
    h1 { color: #333; }
    textarea { width: 100%; height: 150px; margin-bottom: 10px; }
    button { padding: 10px 15px; font-size: 16px; margin-right: 10px; margin-bottom: 10px; }
    #fileList { margin-top: 20px; }
    li { margin-bottom: 5px; }
    select, input[type=range] { margin: 5px 0 15px; display: block; }
  </style>
</head>
<body>
  <h1>Сборщик текстов</h1>

  <label for="voiceSelect">Выбрать голос:</label>
  <select id="voiceSelect"></select>

  <label for="rateRange">Скорость речи:</label>
  <input type="range" id="rateRange" min="0.5" max="2" value="1" step="0.1">

  <textarea id="textInput" placeholder="Введите текст..."></textarea><br>
  <button onclick="addText()">Добавить текст</button>
  <button onclick="downloadFile()">Скачать файл</button>
  <button onclick="speakAll()">Озвучить всё как историю</button>

  <ul id="fileList"></ul>

  <script>
    const texts = [];
    let voices = [];

    function populateVoiceList() {
      voices = speechSynthesis.getVoices().filter(v => v.lang.startsWith("ru"));
      const voiceSelect = document.getElementById("voiceSelect");
      voiceSelect.innerHTML = "";
      voices.forEach((voice, i) => {
        const option = document.createElement("option");
        option.value = i;
        option.textContent = `${voice.name} (${voice.lang})`;
        voiceSelect.appendChild(option);
      });
    }

    populateVoiceList();
    if (typeof speechSynthesis !== 'undefined' && speechSynthesis.onvoiceschanged !== undefined) {
      speechSynthesis.onvoiceschanged = populateVoiceList;
    }

    function addText() {
      const text = document.getElementById("textInput").value.trim();
      if (text) {
        texts.push(text);
        const li = document.createElement("li");
        li.textContent = text.substring(0, 30) + "... ";

        const speakBtn = document.createElement("button");
        speakBtn.textContent = "Озвучить";
        speakBtn.onclick = () => speakText(text);
        li.appendChild(speakBtn);

        document.getElementById("fileList").appendChild(li);
        document.getElementById("textInput").value = "";
      }
    }

    function downloadFile() {
      if (!texts.length) return alert("Добавьте хотя бы один текст!");
      const blob = new Blob([texts.join("\n\n---\n\n")], { type: "text/plain" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "texts.txt";
      link.click();
    }

    function speakText(text) {
      const utterance = new SpeechSynthesisUtterance(text);
      utterance.lang = "ru-RU";
      const voiceSelect = document.getElementById("voiceSelect");
      const selectedVoice = voices[voiceSelect.value];
      if (selectedVoice) utterance.voice = selectedVoice;
      utterance.rate = parseFloat(document.getElementById("rateRange").value);
      speechSynthesis.speak(utterance);
    }

    function speakAll() {
      if (!texts.length) return alert("Нет добавленных текстов!");
      const voiceSelect = document.getElementById("voiceSelect");
      const selectedVoice = voices[voiceSelect.value];
      const rate = parseFloat(document.getElementById("rateRange").value);

      let index = 0;
      function speakNext() {
        if (index >= texts.length) return;
        const utterance = new SpeechSynthesisUtterance(texts[index]);
        utterance.lang = "ru-RU";
        if (selectedVoice) utterance.voice = selectedVoice;
        utterance.rate = rate;
        utterance.onend = () => {
          index++;
          setTimeout(speakNext, 1500); // пауза 1.5 секунды между частями
        };
        speechSynthesis.speak(utterance);
      }

      speakNext();
    }
  </script>
</body>
</html>
