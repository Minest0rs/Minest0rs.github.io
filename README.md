<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Бесплатный видео-звонок</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: Arial, sans-serif;
            background: #1a1a1a;
            color: white;
            padding: 20px;
            text-align: center;
        }
        
        .container {
            max-width: 500px;
            margin: 0 auto;
        }
        
        h1 {
            margin: 20px 0;
            color: #4CAF50;
        }
        
        button {
            padding: 15px 25px;
            font-size: 18px;
            margin: 10px;
            border: none;
            border-radius: 10px;
            background: #4CAF50;
            color: white;
            cursor: pointer;
            width: 90%;
        }
        
        button:disabled {
            background: #666;
            cursor: not-allowed;
        }
        
        button.secondary {
            background: #2196F3;
        }
        
        button.danger {
            background: #f44336;
        }
        
        .video-container {
            display: flex;
            flex-direction: column;
            gap: 15px;
            margin: 20px 0;
        }
        
        video {
            width: 100%;
            max-width: 400px;
            height: 300px;
            background: black;
            border-radius: 10px;
            margin: 0 auto;
        }
        
        input {
            padding: 15px;
            margin: 10px 0;
            width: 90%;
            border: 1px solid #444;
            border-radius: 10px;
            background: #2d2d2d;
            color: white;
            font-size: 16px;
        }
        
        .status {
            padding: 15px;
            margin: 15px 0;
            border-radius: 10px;
            background: #2d2d2d;
        }
        
        .instructions {
            background: #2d2d2d;
            padding: 15px;
            border-radius: 10px;
            margin: 15px 0;
            font-size: 14px;
            text-align: left;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📞 Бесплатный видео-звонок</h1>
        
        <button onclick="startCamera()" id="startBtn">
            🎥 Включить камеру и микрофон
        </button>

        <div class="video-container">
            <div>
                <video id="localVideo" autoplay muted playsinline></video>
                <div>Вы</div>
            </div>
            <div>
                <video id="remoteVideo" autoplay playsinline></video>
                <div>Собеседник</div>
            </div>
        </div>

        <div class="instructions">
            <strong>Как использовать:</strong><br>
            1. Нажмите "Включить камеру" и разрешите доступ<br>
            2. Скопируйте ваш ID и дайте собеседнику<br>
            3. Введите ID собеседника<br>
            4. Нажмите "Начать звонок"
        </div>

        <input type="text" id="peerIdInput" placeholder="Введите ID собеседника">
        
        <input type="text" id="myPeerId" readonly placeholder="Ваш ID появится здесь">
        
        <div>
            <button onclick="connectToPeer()" class="secondary">Подключиться</button>
            <button onclick="startCall()" id="callBtn" disabled>📞 Начать звонок</button>
            <button onclick="endCall()" id="endBtn" class="danger" disabled>📵 Завершить</button>
            <button onclick="copyMyId()" style="background: #FF9800;">📋 Копировать мой ID</button>
        </div>

        <div id="status" class="status">
            Нажмите "Включить камеру и микрофон"
        </div>
    </div>

    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
    <script>
        let peer;
        let localStream;
        let currentCall;

        const localVideo = document.getElementById('localVideo');
        const remoteVideo = document.getElementById('remoteVideo');
        const startBtn = document.getElementById('startBtn');
        const callBtn = document.getElementById('callBtn');
        const endBtn = document.getElementById('endBtn');
        const statusDiv = document.getElementById('status');
        const myPeerIdInput = document.getElementById('myPeerId');
        const peerIdInput = document.getElementById('peerIdInput');

        async function startCamera() {
            try {
                statusDiv.innerHTML = "⌛ Запрос разрешений...";
                
                localStream = await navigator.mediaDevices.getUserMedia({
                    video: {
                        width: { ideal: 640 },
                        height: { ideal: 480 },
                        facingMode: 'user'
                    },
                    audio: {
                        echoCancellation: true,
                        noiseSuppression: true,
                        autoGainControl: true
                    }
                });

                localVideo.srcObject = localStream;
                statusDiv.innerHTML = "✅ Камера и микрофон включены!";
                startBtn.disabled = true;
                startBtn.textContent = "✅ Камера включена";
                callBtn.disabled = false;

                initializePeer();

            } catch (error) {
                console.error("Ошибка:", error);
                let errorMsg = "❌ Ошибка: ";
                
                if (error.name === 'NotAllowedError') {
                    errorMsg += "Вы отказали в доступе. Обновите страницу и попробуйте снова.";
                } else if (error.name === 'NotFoundError') {
                    errorMsg += "Камера не найдена.";
                } else {
                    errorMsg += error.message;
                }
                
                statusDiv.innerHTML = errorMsg;
            }
        }

        function initializePeer() {
            const peerId = generateRandomId();
            
            peer = new Peer(peerId, {
                host: '0.peerjs.com',
                port: 443,
                path: '/'
            });

            peer.on('open', (id) => {
                myPeerIdInput.value = id;
                statusDiv.innerHTML = "✅ Готов к звонку! Ваш ID: " + id;
            });

            peer.on('call', (call) => {
                statusDiv.innerHTML = "📞 Входящий звонок...";
                call.answer(localStream);
                handleCall(call);
            });

            peer.on('error', (err) => {
                console.error("PeerJS ошибка:", err);
                statusDiv.innerHTML = "❌ Ошибка: " + err.type;
            });
        }

        function generateRandomId() {
            return Math.random().toString(36).substring(2, 8);
        }

        function connectToPeer() {
            const peerId = peerIdInput.value.trim();
            if (!peerId) {
                alert("Введите ID собеседника");
                return;
            }
            
            const conn = peer.connect(peerId);
            conn.on('open', () => {
                statusDiv.innerHTML = "✅ Подключено к " + peerId;
            });
        }

        function startCall() {
            const peerId = peerIdInput.value.trim();
            if (!peerId) {
                alert("Введите ID собеседника");
                return;
            }

            statusDiv.innerHTML = "⌛ Установка соединения...";
            
            const call = peer.call(peerId, localStream);
            handleCall(call);
        }

        function handleCall(call) {
            currentCall = call;
            
            call.on('stream', (remoteStream) => {
                remoteVideo.srcObject = remoteStream;
                statusDiv.innerHTML = "✅ Звонок активен!";
                callBtn.disabled = true;
                endBtn.disabled = false;
            });

            call.on('close', () => {
                endCall();
            });

            call.on('error', (err) => {
                console.error("Ошибка звонка:", err);
                statusDiv.innerHTML = "❌ Ошибка звонка";
                endCall();
            });
        }

        function endCall() {
            if (currentCall) {
                currentCall.close();
                currentCall = null;
            }
            
            if (remoteVideo.srcObject) {
                remoteVideo.srcObject = null;
            }
            
            statusDiv.innerHTML = "❌ Звонок завершен";
            callBtn.disabled = false;
            endBtn.disabled = true;
        }

        function copyMyId() {
            myPeerIdInput.select();
            document.execCommand('copy');
            alert("✅ ID скопирован!");
        }

        window.addEventListener('beforeunload', () => {
            if (currentCall) currentCall.close();
            if (peer) peer.destroy();
            if (localStream) localStream.getTracks().forEach(track => track.stop());
        });
    </script>
</body>
</html><li><a href="http://m.wikipedia.org">Википедия HTTP</a></li>
<li><a href="http://mobile.yandex.ru">Яндекс Mobile</a></li>
<li><a href="http://textise.iitty">Textise HTTP</a></li>
</ul>

<h2>Новости:</h2>
<ul>
<li><a href="http://wap.rambler.ru">Rambler WAP</a></li>
<li><a href="http://m.lenta.ru">Lenta.ru Mobile</a></li>
</ul>
</body>
</html>
