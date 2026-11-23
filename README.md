# godot-assignment

index.html
{
	<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Secure Sandbox Controller</title>
    <style>
        body {
            background: #1e1e1e; margin: 0; color: white;
            font-family: Arial; display: flex; flex-direction: column;
            height: 100vh;
        }
        header {
            padding: 15px; background: #252526; display: flex;
            justify-content: space-between; align-items: center;
            border-bottom: 1px solid #333;
        }
        button {
            padding: 10px 20px; border: none; cursor: pointer;
            font-weight: bold; border-radius: 5px; color: white;
        }
        #btn-start { background: #28a745; }
        #btn-stop { background: #dc3545; }
        #btn-stop:disabled { opacity: 0.6; }

        #screen {
            flex-grow: 1; background: black; position: relative;
        }

        iframe {
            width: 100%; height: 100%; border: none;
            position: absolute; top: 0; left: 0;
        }

        #msg {
            position: absolute; top: 0; left: 0;
            width: 100%; height: 100%;
            display: flex; justify-content: center; align-items: center;
            flex-direction: column; background: #111;
        }
    </style>
</head>
<body>

<header>
    <h2>🛡 Secure Sandbox Controller</h2>
    <div>
        <button id="btn-start">Start</button>
        <button id="btn-stop" disabled>Stop</button>
    </div>
</header>

<div id="screen">
    <div id="msg">
        <h2>System Offline</h2>
        <p>Press Start.</p>
    </div>
    <iframe id="vnc"></iframe>
</div>

<script>
    const startBtn = document.getElementById("btn-start");
    const stopBtn = document.getElementById("btn-stop");
    const msg = document.getElementById("msg");
    const iframe = document.getElementById("vnc");

    startBtn.onclick = async () => {
        startBtn.disabled = true;
        msg.innerHTML = "<h2>Starting Docker...</h2><p>Please wait...</p>";

        const res = await window.sandboxAPI.startSandbox();

        if (!res.success) {
            msg.innerHTML = `<h2>Error</h2><p>${res.output}</p>`;
            startBtn.disabled = false;
            return;
        }

        stopBtn.disabled = false;

        setTimeout(() => {
            msg.style.display = "none";
            iframe.src = "http://localhost:7900/?autoconnect=true&resize=scale";
        }, 8000);
    };

    stopBtn.onclick = async () => {
        stopBtn.disabled = true;

        await window.sandboxAPI.stopSandbox();

        iframe.src = "";
        msg.style.display = "flex";
        msg.innerHTML = "<h2>System Offline</h2>";
        startBtn.disabled = false;
    };
</script>

<div id="logs" style="background:#111; color:#0f0; padding:10px; height:150px; font-size:12px; overflow-y:auto; border-top:1px solid #333;">
    <b>System Logs:</b><br>
</div>

</body>
</html>
}





package.json
{
{
  "name": "sandbox-app",
  "version": "1.0.0",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  },
  "devDependencies": {
    "electron": "^33.0.0"
  }
}
}




main.js
{
const { app, BrowserWindow, ipcMain } = require('electron');
const { exec } = require('child_process');
const path = require('path');

function createWindow() {
    const win = new BrowserWindow({
        width: 1100,
        height: 800,
        webPreferences: {
            preload: path.join(__dirname, 'preload.js'),
            nodeIntegration: false,
            contextIsolation: true
        }
    });

    win.loadFile('index.html');
}

app.whenReady().then(() => {
    createWindow();
});

// START DOCKER
ipcMain.handle('start-docker', () => {
    return new Promise((resolve) => {
        exec('docker compose up -d', { cwd: __dirname }, (error, stdout, stderr) => {
            if (error) {
                resolve({ success: false, output: stderr });
            } else {
                resolve({ success: true, output: stdout });
            }
        });
    });
});

// STOP DOCKER
ipcMain.handle('stop-docker', () => {
    return new Promise((resolve) => {
        exec('docker compose down', { cwd: __dirname }, () => {
            resolve({ success: true });
        });
    });
});
}






prload.js
{
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('sandboxAPI', {
    startSandbox: () => ipcRenderer.invoke('start-docker'),
    stopSandbox: () => ipcRenderer.invoke('stop-docker')
});
}





docker-compose.yml
{
services:
  secure-browser:
    image: selenium/standalone-chrome:latest
    container_name: sandbox-browser
    shm_size: 2gb
    ports:
      - "7900:7900"
    environment:
      - SE_VNC_NO_PASSWORD=1
      - SE_SCREEN_WIDTH=1360
      - SE_SCREEN_HEIGHT=1020
}

