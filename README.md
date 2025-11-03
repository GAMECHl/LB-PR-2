# LB-PR-2-Hurzidze-Anton
## Хурцидзе Антон IПЗ 4.02 Лабораторна-Практична робота № 2

## Тема: Розробка додатку для візуалізації вимірювань радару
## Мета: Розробити додаток, який зчитує дані з емульованої вимірювальної частини радару, наданої у вигляді Docker image, та відображає задетектовані цілі на графіку в полярних координатах.

## 0. Розробити додаток для відображення цілей:

#### Перед початком створювання веб-додатку радару підключаємо контейнер командою ``docker pull iperekrestov/university:radar-emulation-service`` та командою ``docker run --name radar-emulator -p 4000:4000 iperekrestov/university:radar-emulation-service`` запускаємо його для получення данних по цілі:

![0](https://github.com/GAMECHl/LB-PR-2/blob/main/0.png)
#### Рис. 1 - підключенний контейнер

## 1. Розробити додаток для відображення цілей:

#### Створюємо веб-додаток, який підключається до WebSocket сервера та зчитує дані про задетектовані цілі. Додаток реалізовано у вигляді HTML файлу, який використовує протокол WebSocket для підключення до контейнера та за допомогою бібліотеки Plotly:

![1](https://github.com/GAMECHl/LB-PR-2/blob/main/1.png)
#### Рис. 2 - Веб-додаток радару

## 2. Обробка та візуалізація даних:

#### Далі налаштовуюємо додаток таким чином, щоб він обробляв отримані через WebSocket дані та відображав кожну ціль як точку на графіку з координатами (кут, відстань), а також надавав можливість зміни параметрів радару через API запити:

![2](https://github.com/GAMECHl/LB-PR-2/blob/main/2.png)
#### Рис. 3 - Оброблення данних
![3](https://github.com/GAMECHl/LB-PR-2/blob/main/3.png)
#### Рис. 4 - Зміна параметрів
![4](https://github.com/GAMECHl/LB-PR-2/blob/main/4.png)
#### Рис. 5 - Оновлення графіку

## 3. Налаштування графіка:

#### Наступним чином відображаємо відстань у кілометрах у радіальній осі та відображаємо різні кольори або стилі точок для відображення різних рівнів потужності сигналів :

![5](https://github.com/GAMECHl/LB-PR-2/blob/main/5.png)
#### Рис. 6 - Відображення відстані та пожуності

**radar.html**
```html
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Радар - Відображення Цілей</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.26.0/plotly.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: #fff;
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
        }
        
        h1 {
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .status-bar {
            background: rgba(255,255,255,0.1);
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            backdrop-filter: blur(10px);
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #ff4444;
            animation: pulse 2s infinite;
        }
        
        .status-dot.connected {
            background: #44ff44;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        
        .main-content {
            display: grid;
            grid-template-columns: 1fr 350px;
            gap: 20px;
        }
        
        .radar-container {
            background: rgba(255,255,255,0.05);
            border-radius: 15px;
            padding: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        
        #radarPlot {
            width: 100%;
            height: 700px;
        }
        
        .control-panel {
            background: rgba(255,255,255,0.05);
            border-radius: 15px;
            padding: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        
        .panel-section {
            margin-bottom: 25px;
        }
        
        .panel-section h3 {
            margin-bottom: 15px;
            color: #4fc3f7;
            font-size: 1.2em;
        }
        
        .input-group {
            margin-bottom: 15px;
        }
        
        .input-group label {
            display: block;
            margin-bottom: 5px;
            font-size: 0.9em;
            color: #b3e5fc;
        }
        
        .input-group input {
            width: 100%;
            padding: 10px;
            border: none;
            border-radius: 5px;
            background: rgba(255,255,255,0.1);
            color: #fff;
            font-size: 1em;
        }
        
        .input-group input:focus {
            outline: 2px solid #4fc3f7;
            background: rgba(255,255,255,0.15);
        }
        
        button {
            width: 100%;
            padding: 12px;
            border: none;
            border-radius: 5px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: #fff;
            font-size: 1em;
            font-weight: bold;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }
        
        button:active {
            transform: translateY(0);
        }
        
        .stats {
            background: rgba(255,255,255,0.05);
            padding: 15px;
            border-radius: 8px;
            margin-top: 10px;
        }
        
        .stats-item {
            display: flex;
            justify-content: space-between;
            padding: 8px 0;
            border-bottom: 1px solid rgba(255,255,255,0.1);
        }
        
        .stats-item:last-child {
            border-bottom: none;
        }
        
        .stats-label {
            color: #b3e5fc;
        }
        
        .stats-value {
            font-weight: bold;
            color: #4fc3f7;
        }
        
        .log-container {
            background: rgba(0,0,0,0.3);
            padding: 10px;
            border-radius: 5px;
            max-height: 150px;
            overflow-y: auto;
            font-family: 'Courier New', monospace;
            font-size: 0.85em;
        }
        
        .log-entry {
            padding: 3px 0;
            color: #b3e5fc;
        }
        
        .log-entry.error {
            color: #ff6b6b;
        }
        
        .log-entry.success {
            color: #51cf66;
        }
        
        @media (max-width: 1024px) {
            .main-content {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Радар - Відображення Цілей</h1>
        
        <div class="status-bar">
            <div class="status-indicator">
                <div class="status-dot" id="statusDot"></div>
                <span id="statusText">Відключено</span>
            </div>
        </div>
        
        <div class="main-content">
            <div class="radar-container">
                <div id="radarPlot"></div>
            </div>
            
            <div class="control-panel">
                <div class="panel-section">
                    <h3> Параметри Радару</h3>
                    <div class="input-group">
                        <label>Вимірювань на оберт:</label>
                        <input type="number" id="measurementsPerRotation" value="360" min="1" max="720">
                    </div>
                    <div class="input-group">
                        <label>Швидкість обертання (RPM):</label>
                        <input type="number" id="rotationSpeed" value="60" min="1" max="120">
                    </div>
                    <div class="input-group">
                        <label>Швидкість цілей (км/год):</label>
                        <input type="number" id="targetSpeed" value="100" min="0" max="1000">
                    </div>
                    <button onclick="updateConfig()">Застосувати параметри</button>
                </div>
                
                <div class="panel-section">
                    <h3> Логи</h3>
                    <div class="log-container" id="logContainer"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        let socket = null;
        let allTargets = []; // Всі цілі з відміткою часу
        let packetCount = 0;
        const SPEED_OF_LIGHT = 299792458; // м/с
        const TARGET_LIFETIME = 5000; // Час життя цілі на екрані (5 секунд)
        
        // Ініціалізація графіка
        const layout = {
            polar: {
                radialaxis: {
                    visible: true,
                    range: [0, 200],
                    title: 'Відстань (км)',
                    gridcolor: 'rgba(255,255,255,0.2)',
                    color: '#fff'
                },
                angularaxis: {
                    direction: 'clockwise',
                    rotation: 90,
                    gridcolor: 'rgba(255,255,255,0.2)',
                    color: '#fff'
                },
                bgcolor: 'rgba(0,0,0,0.3)'
            },
            paper_bgcolor: 'rgba(0,0,0,0)',
            plot_bgcolor: 'rgba(0,0,0,0)',
            showlegend: false,
            margin: {t: 40, b: 40, l: 40, r: 40}
        };
        
        const initialData = [{
            type: 'scatterpolar',
            r: [],
            theta: [],
            mode: 'markers',
            marker: {
                size: 10,
                color: [],
                colorscale: [
                    [0, 'rgba(255,255,0,0.3)'],
                    [0.5, 'rgba(255,128,0,0.7)'],
                    [1, 'rgba(255,0,0,1)']
                ],
                showscale: true,
                colorbar: {
                    title: 'Потужність',
                    titleside: 'right',
                    tickmode: 'linear',
                    tick0: 0,
                    dtick: 0.2
                }
            }
        }];
        
        Plotly.newPlot('radarPlot', initialData, layout, {responsive: true});
        
        // Функція логування
        function addLog(message, type = 'info') {
            const logContainer = document.getElementById('logContainer');
            const entry = document.createElement('div');
            entry.className = `log-entry ${type}`;
            const time = new Date().toLocaleTimeString();
            entry.textContent = `[${time}] ${message}`;
            logContainer.insertBefore(entry, logContainer.firstChild);
            
            // Обмеження кількості логів
            while (logContainer.children.length > 50) {
                logContainer.removeChild(logContainer.lastChild);
            }
        }
        
        // Підключення до WebSocket
        function connectWebSocket() {
            try {
                socket = new WebSocket('ws://localhost:4000');
                
                socket.onopen = () => {
                    document.getElementById('statusDot').classList.add('connected');
                    document.getElementById('statusText').textContent = 'Підключено';
                    addLog('Підключено до WebSocket сервера', 'success');
                };
                
                socket.onmessage = (event) => {
                    try {
                        const data = JSON.parse(event.data);
                        processRadarData(data);
                    } catch (e) {
                        console.error('Помилка обробки даних:', e);
                    }
                };
                
                socket.onclose = () => {
                    document.getElementById('statusDot').classList.remove('connected');
                    document.getElementById('statusText').textContent = 'Відключено';
                    addLog('З\'єднання закрито', 'error');
                    
                    // Спроба повторного підключення через 3 секунди
                    setTimeout(connectWebSocket, 3000);
                };
                
                socket.onerror = (error) => {
                    addLog('Помилка WebSocket: ' + error.message, 'error');
                };
                
            } catch (e) {
                addLog('Помилка підключення: ' + e.message, 'error');
                setTimeout(connectWebSocket, 3000);
            }
        }
        
        // Обробка даних радару
        function processRadarData(data) {
            packetCount++;
            
            const currentTime = Date.now();
            
            // Видалення старих цілей
            allTargets = allTargets.filter(t => currentTime - t.timestamp < TARGET_LIFETIME);
            
            // Обробка ехо-відповідей
            if (data.echoResponses && data.echoResponses.length > 0) {
                data.echoResponses.forEach(echo => {
                    // Розрахунок відстані: d = (c * t) / 2
                    const distanceMeters = (SPEED_OF_LIGHT * echo.time) / 2;
                    const distanceKm = distanceMeters / 1000;
                    
                    allTargets.push({
                        angle: data.scanAngle,
                        distance: distanceKm,
                        power: echo.power,
                        timestamp: currentTime
                    });
                });
                
                updateRadarPlot();
            }
        }
        
        // Оновлення графіка
        function updateRadarPlot() {
            if (allTargets.length === 0) return;
            
            const angles = allTargets.map(t => t.angle);
            const distances = allTargets.map(t => t.distance);
            const powers = allTargets.map(t => t.power);
            
            const update = {
                r: [distances],
                theta: [angles],
                'marker.color': [powers]
            };
            
            Plotly.restyle('radarPlot', update, [0]);
        }
        
        // Оновлення конфігурації
        async function updateConfig() {
            const config = {
                measurementsPerRotation: parseInt(document.getElementById('measurementsPerRotation').value),
                rotationSpeed: parseInt(document.getElementById('rotationSpeed').value),
                targetSpeed: parseInt(document.getElementById('targetSpeed').value)
            };
            
            try {
                const response = await fetch('http://localhost:4000/config', {
                    method: 'PUT',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(config)
                });
                
                if (response.ok) {
                    addLog('Параметри успішно оновлено', 'success');
                } else {
                    addLog('Помилка оновлення параметрів', 'error');
                }
            } catch (e) {
                addLog('Помилка з\'єднання з API: ' + e.message, 'error');
            }
        }
        
        // Запуск при завантаженні сторінки
        connectWebSocket();
    </script>
</body>
</html>
```

Висновок: Протягом виконання лабораторно-практичної роботи я розробив додаток, який зчитує дані з емульованої вимірювальної частини радару, наданої у вигляді Docker image, та відображає задетектовані цілі на графіку в полярних координатах.
