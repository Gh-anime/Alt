# Alt
Ah, du möchtest also eine GUI für den KI-Sprachassistenten mit `customtkinter`, die das Ganze benutzerfreundlicher macht, richtig? Kein Problem, ich zeige dir, wie du das umsetzen kannst. Hier kommt die komplette Version deines Projekts mit einer GUI.

---

### 1️⃣ **Komplettes Python-Script mit GUI (`assistent_gui.py`)**

Dieses Script kombiniert alles, was du brauchst: GUI mit `customtkinter`, Sprachsteuerung (Speech-to-Text), Kommunikation mit `ollama` und Sprachausgabe (Text-to-Speech).

```python
import os
import wave
import pyaudio
import requests
import customtkinter as ctk
from vosk import Model, KaldiRecognizer
from piper.voice import PiperVoice
from threading import Thread

# --- Vosk für Speech-to-Text ---
def speech_to_text_vosk():
    model_path = "C:/ki_assistent/vosk-model-small-de-0.15"
    if not os.path.exists(model_path):
        print(f"Bitte das deutsche Sprachmodell von https://alphacephei.com/vosk/models herunterladen und nach {model_path} entpacken")
        return None

    model = Model(model_path)
    recognizer = KaldiRecognizer(model, 16000)

    p = pyaudio.PyAudio()
    stream = p.open(format=pyaudio.paInt16, channels=1, rate=16000, input=True, frames_per_buffer=8192)
    stream.start_stream()

    try:
        while True:
            data = stream.read(4096, exception_on_overflow=False)
            if recognizer.AcceptWaveform(data):
                result = recognizer.Result()
                text = eval(result)["text"]
                if text:
                    return text
    except KeyboardInterrupt:
        print("\nAufnahme beendet")
    finally:
        stream.stop_stream()
        stream.close()
        p.terminate()

# --- Ollama API-Anfrage ---
def ask_ollama(question):
    url = 'http://localhost:11434/api/chat'
    headers = {'Content-Type': 'application/json'}

    data = {
        'model': 'llama3.2',
        'messages': [{'role': 'user', 'content': question}]
    }

    response = requests.post(url, json=data, headers=headers)
    response_data = response.json()

    return response_data.get('choices', [{}])[0].get('message', {}).get('content', 'Keine Antwort erhalten.')

# --- Piper TTS für Text zu Sprache ---
def text_to_speech_piper(text):
    model_path = "C:/ki_assistent/de_DE-thorsten-high.onnx"
    voice = PiperVoice.load(model_path)

    with wave.open("output.wav", "w") as wav_file:
        audio = voice.synthesize(text, wav_file)

    os.system("start output.wav")  # Audio sofort abspielen

# --- GUI mit customtkinter ---
class KI_Assistent_GUI(ctk.CTk):

    def __init__(self):
        super().__init__()

        self.title("KI Sprachassistent")
        self.geometry("500x400")
        self.resizable(False, False)

        # Label für den Status
        self.status_label = ctk.CTkLabel(self, text="Bereit, auf Befehl zu hören...", font=("Arial", 16))
        self.status_label.pack(pady=20)

        # Start-Button für die Spracherkennung
        self.start_button = ctk.CTkButton(self, text="Starte Spracherkennung", command=self.start_recognition)
        self.start_button.pack(pady=10)

        # Textfeld zur Anzeige der Frage und Antwort
        self.text_box = ctk.CTkTextbox(self, width=400, height=150)
        self.text_box.pack(pady=10)

    def start_recognition(self):
        # Startet Spracherkennung in einem separaten Thread
        self.status_label.configure(text="Höre auf deinen Befehl...")
        self.start_button.configure(state="disabled")

        thread = Thread(target=self.process_speech)
        thread.start()

    def process_speech(self):
        question = speech_to_text_vosk()
        if question:
            self.text_box.insert("end", f"Du hast gesagt: {question}\n")
            answer = ask_ollama(question)
            self.text_box.insert("end", f"Ollama Antwort: {answer}\n")
            text_to_speech_piper(answer)
            self.status_label.configure(text="Bereit, auf Befehl zu hören...")

        self.start_button.configure(state="normal")

# --- GUI starten ---
if __name__ == "__main__":
    app = KI_Assistent_GUI()
    app.mainloop()
```

---

### 2️⃣ **Batch-Datei (`starte_assistent_gui.bat`)**

Die `.bat` Datei startet den `ollama`-Server und das Python-Script mit GUI. Hier ist der Code:

```batch
@echo off
title KI Sprachassistent Start

:: Ollama Serve im Hintergrund starten
start "" /min ollama serve

:: 3 Sekunden warten, damit der Server sicher läuft
timeout /t 3 >nul

:: Python-Script ausführen
python C:\ki_assistent\assistent_gui.py

pause
```

---

### 3️⃣ **Projektstruktur**

Dein Projektordner sollte nun folgendermaßen aussehen:

```
C:\ki_assistent\
│
├── assistant_gui.py         # Python-Script für den Sprachassistenten mit GUI
├── start_assistent_gui.bat  # Batch-Datei, um den Assistenten zu starten
├── vosk-model-small-de-0.15\ # Vosk Sprachmodell für Speech-to-Text
│   └── (Modelldaten hier)
├── de_DE-thorsten-high.onnx  # Piper TTS Modell
└── output.wav               # Ausgabedatei für die Audioausgabe
```

---

### 4️⃣ **Erklärung und Setup:**

1. **GUI mit `customtkinter`:**

   * Das Fenster hat einen **Button**, mit dem du die Spracherkennung starten kannst.
   * **Textfeld**, in dem die Frage und Antwort angezeigt wird.
   * **Statusanzeige**, die anzeigt, ob der Assistent bereit ist, zu hören.

2. **Batch-Datei (`start_assistent_gui.bat`):**

   * Startet `ollama` im Hintergrund.
   * Führt das Python-Script mit der GUI aus.

3. **Benötigte Ordner und Dateien:**

   * **Vosk-Modell** muss heruntergeladen werden und im Ordner `vosk-model-small-de-0.15/` gespeichert werden.
   * **Piper TTS Modell** (`de_DE-thorsten-high.onnx`) muss ebenfalls im richtigen Ordner abgelegt werden.

---

### 5️⃣ **Finale Setup-Anweisungen:**

1. **Installiere alle Abhängigkeiten**:

   * Installiere `pyaudio`, `vosk`, `requests`, `customtkinter` und `piper`:

     ```bash
     pip install pyaudio vosk requests customtkinter piper
     ```

2. **Lade das Vosk-Modell**:

   * Hol dir das deutsche Vosk-Modell von [dieser Seite](https://alphacephei.com/vosk/models) und entpacke es in den Ordner `vosk-model-small-de-0.15/`.

3. **Lade das Piper TTS Modell**:

   * Besorge das Modell von der [Piper-Website](https://piper.readthedocs.io/) und lege es im entsprechenden Ordner ab.

4. **Führe das Projekt aus**:

   * Doppelklick auf die `.bat` Datei, um den Assistenten zu starten.

---

Jetzt hast du alles, was du brauchst, um deinen KI-Sprachassistenten mit einer GUI zu nutzen! Lass mich wissen, wenn du noch Fragen hast oder zusätzliche Features wünschst 😊
