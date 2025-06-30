#include <WiFi.h>
#include <WebServer.h>
#include <ArduinoJson.h>

const int redPin = 25;
const int greenPin = 26;
const int bluePin = 27;
const int ldrPin = 2;

const char* ssid = "Akilli_Isik";
const char* password = "12345678";

WebServer server(80);

int brightness = 50;
bool autoMode = false;
String currentMode = "kapali";
bool partyMode = false;
unsigned long lastPartyUpdate = 0;
int partyStep = 0;

const int pwmFreq = 5000;
const int pwmResolution = 8;

void setup() {
  Serial.begin(115200);
  
  ledcAttach(redPin, pwmFreq, pwmResolution);
  ledcAttach(greenPin, pwmFreq, pwmResolution);
  ledcAttach(bluePin, pwmFreq, pwmResolution);
  
  pinMode(ldrPin, INPUT);
  
  setColor(0, 0, 0);
  
  WiFi.softAP(ssid, password);
  Serial.println("WiFi AP baslatildi");
  Serial.print("IP adresi: ");
  Serial.println(WiFi.softAPIP());
  
  server.on("/", handleRoot);
  server.on("/status", handleStatus);
  server.on("/control", HTTP_POST, handleControl);
  
  server.begin();
  Serial.println("Web sunucu baslatildi");
}

void loop() {
  server.handleClient();
  
  if (partyMode && currentMode == "party") {
    if (millis() - lastPartyUpdate > 500) {
      partyAnimation();
      lastPartyUpdate = millis();
    }
  }
  
  if (autoMode) {
    checkAutoMode();
  }
  
  delay(10);
}

void setColor(int red, int green, int blue) {
  int r = map(red * brightness, 0, 25500, 0, 255);
  int g = map(green * brightness, 0, 25500, 0, 255);
  int b = map(blue * brightness, 0, 25500, 0, 255);
  
  ledcWrite(redPin, r);
  ledcWrite(greenPin, g);
  ledcWrite(bluePin, b);
}

void partyAnimation() {
  switch(partyStep) {
    case 0: setColor(255, 0, 0); break;     // Kirmizi
    case 1: setColor(255, 127, 0); break;   // Turuncu
    case 2: setColor(255, 255, 0); break;   // Sari
    case 3: setColor(0, 255, 0); break;     // Yesil
    case 4: setColor(0, 0, 255); break;     // Mavi
    case 5: setColor(75, 0, 130); break;    // Indigo
    case 6: setColor(148, 0, 211); break;   // Mor
  }
  partyStep = (partyStep + 1) % 7;
}

void checkAutoMode() {
  bool isDark = digitalRead(ldrPin) == HIGH;
  
  if (isDark && currentMode != "beyaz") {
    currentMode = "beyaz";
    setColor(255, 255, 255);
  } else if (!isDark && currentMode == "beyaz") {
    currentMode = "kapali";
    setColor(0, 0, 0);
  }
}

void handleRoot() {
  String html = R"(
<!DOCTYPE html>
<html>
<head>
    <meta charset='UTF-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <title>Akilli Isik Kontrolu</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            color: white;
        }
        .container {
            max-width: 500px;
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }
        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }
        .status-card {
            background: rgba(255, 255, 255, 0.15);
            border-radius: 15px;
            padding: 20px;
            margin-bottom: 25px;
            text-align: center;
        }
        .control-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 25px;
        }
        .btn {
            padding: 15px;
            border: none;
            border-radius: 12px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            backdrop-filter: blur(5px);
        }
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }
        .btn:active {
            transform: translateY(0);
        }
        .btn-off { background: rgba(108, 117, 125, 0.8); }
        .btn-white { background: rgba(248, 249, 250, 0.8); color: #333; }
        .btn-blue { background: rgba(0, 123, 255, 0.8); }
        .btn-red { background: rgba(220, 53, 69, 0.8); }
        .btn-green { background: rgba(40, 167, 69, 0.8); }
        .btn-party { background: linear-gradient(45deg, #ff006e, #8338ec, #3a86ff); }
        .btn-auto {
            grid-column: span 2;
            background: rgba(255, 193, 7, 0.8);
            color: #333;
        }
        .btn-auto.active {
            background: rgba(40, 167, 69, 0.8);
            color: white;
        }
        .brightness-control {
            margin-top: 25px;
        }
        .brightness-label {
            display: block;
            margin-bottom: 10px;
            font-weight: 600;
            text-align: center;
        }
        .brightness-slider {
            width: 100%;
            height: 8px;
            border-radius: 5px;
            background: rgba(255, 255, 255, 0.3);
            outline: none;
            -webkit-appearance: none;
        }
        .brightness-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 25px;
            height: 25px;
            border-radius: 50%;
            background: #fff;
            cursor: pointer;
            box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
        }
        .brightness-value {
            text-align: center;
            margin-top: 10px;
            font-size: 18px;
            font-weight: 600;
        }
        .ldr-status {
            padding: 10px;
            border-radius: 10px;
            margin-top: 15px;
            font-weight: 600;
            text-align: center;
        }
        .dark { background: rgba(52, 58, 64, 0.8); }
        .light { background: rgba(255, 235, 59, 0.8); color: #333; }
    </style>
</head>
<body>
    <div class='container'>
        <h1>💡 Akilli Isik</h1>
        
        <div class='status-card'>
            <h3>Durum</h3>
            <div id='current-mode'>Mod: <span id='mode-text'>Yukleniyor...</span></div>
            <div class='ldr-status' id='ldr-status'>Ortam: Yukleniyor...</div>
        </div>
        
        <div class='control-grid'>
            <button class='btn btn-off' onclick='setMode("kapali")'>🔴 Kapat</button>
            <button class='btn btn-white' onclick='setMode("beyaz")'>⚪ Beyaz</button>
            <button class='btn btn-blue' onclick='setMode("mavi")'>🔵 Mavi</button>
            <button class='btn btn-red' onclick='setMode("kirmizi")'>🔴 Kirmizi</button>
            <button class='btn btn-green' onclick='setMode("yesil")'>🟢 Yesil</button>
            <button class='btn btn-party' onclick='setMode("party")'>🎉 Party</button>
            <button class='btn btn-auto' id='auto-btn' onclick='toggleAuto()'>🤖 LDR Auto</button>
        </div>
        
        <div class='brightness-control'>
            <label class='brightness-label'>Parlaklik Kontrolu</label>
            <input type='range' min='1' max='100' value='50' class='brightness-slider' id='brightness'>
            <div class='brightness-value' id='brightness-value'>50%</div>
        </div>
    </div>

    <script>
        let currentStatus = {};
        let isUpdating = false;
        
        // Brightness slider kontrolu
        document.getElementById('brightness').addEventListener('input', function() {
            if (!isUpdating) {
                document.getElementById('brightness-value').textContent = this.value + '%';
                setBrightness(this.value);
            }
        });
        
        // Durum guncelleme
        function updateStatus() {
            fetch('/status')
                .then(response => response.json())
                .then(data => {
                    currentStatus = data;
                    
                    // Mod gosterimi
                    let modeText = '';
                    switch(data.mode) {
                        case 'kapali': modeText = '🔴 Kapali'; break;
                        case 'beyaz': modeText = '⚪ Beyaz'; break;
                        case 'mavi': modeText = '🔵 Mavi'; break;
                        case 'kirmizi': modeText = '🔴 Kirmizi'; break;
                        case 'yesil': modeText = '🟢 Yesil'; break;
                        case 'party': modeText = '🎉 Party'; break;
                    }
                    document.getElementById('mode-text').textContent = modeText;
                    
                    // LDR durumu
                    const ldrStatus = document.getElementById('ldr-status');
                    if (data.isDark) {
                        ldrStatus.textContent = '🌙 Ortam: Karanlik';
                        ldrStatus.className = 'ldr-status dark';
                    } else {
                        ldrStatus.textContent = '☀️ Ortam: Aydinlik';
                        ldrStatus.className = 'ldr-status light';
                    }
                    
                    // Auto buton durumu
                    const autoBtn = document.getElementById('auto-btn');
                    if (data.autoMode) {
                        autoBtn.classList.add('active');
                        autoBtn.textContent = '🤖 LDR Auto (Aktif)';
                    } else {
                        autoBtn.classList.remove('active');
                        autoBtn.textContent = '🤖 LDR Auto';
                    }
                    
                    // Brightness slider guncelle
                    if (!isUpdating) {
                        document.getElementById('brightness').value = data.brightness;
                        document.getElementById('brightness-value').textContent = data.brightness + '%';
                    }
                })
                .catch(error => console.error('Hata:', error));
        }
        
        function setMode(mode) {
            fetch('/control', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ action: 'setMode', mode: mode })
            });
        }
        
        function toggleAuto() {
            fetch('/control', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ action: 'toggleAuto' })
            });
        }
        
        function setBrightness(value) {
            isUpdating = true;
            fetch('/control', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ action: 'setBrightness', brightness: parseInt(value) })
            }).then(() => {
                setTimeout(() => { isUpdating = false; }, 100);
            });
        }
        
        // Otomatik guncelleme
        updateStatus();
        setInterval(updateStatus, 1000);
    </script>
</body>
</html>
  )";
  
  server.send(200, "text/html", html);
}

void handleStatus() {
  DynamicJsonDocument doc(1024);
  
  doc["mode"] = currentMode;
  doc["brightness"] = brightness;
  doc["autoMode"] = autoMode;
  doc["isDark"] = digitalRead(ldrPin) == HIGH;
  
  String response;
  serializeJson(doc, response);
  
  server.send(200, "application/json", response);
}

void handleControl() {
  if (server.hasArg("plain")) {
    DynamicJsonDocument doc(1024);
    deserializeJson(doc, server.arg("plain"));
    
    String action = doc["action"];
    
    if (action == "setMode") {
      String mode = doc["mode"];
      currentMode = mode;
      partyMode = false;
      
      if (mode == "kapali") {
        setColor(0, 0, 0);
      } else if (mode == "beyaz") {
        setColor(255, 255, 255);
      } else if (mode == "mavi") {
        setColor(0, 0, 255);
      } else if (mode == "kirmizi") {
        setColor(255, 0, 0);
      } else if (mode == "yesil") {
        setColor(0, 255, 0);
      } else if (mode == "party") {
        partyMode = true;
        partyStep = 0;
      }
    } else if (action == "toggleAuto") {
      autoMode = !autoMode;
      if (!autoMode) {
        // Auto mod kapandiginda mevcut durumu koru
      }
    } else if (action == "setBrightness") {
      brightness = doc["brightness"];
      // Mevcut rengi yeni parlaklikla guncelle
      if (currentMode == "beyaz") {
        setColor(255, 255, 255);
      } else if (currentMode == "mavi") {
        setColor(0, 0, 255);
      } else if (currentMode == "kirmizi") {
        setColor(255, 0, 0);
      } else if (currentMode == "yesil") {
        setColor(0, 255, 0);
      }
    }
  }
  
  server.send(200, "text/plain", "OK");
}
