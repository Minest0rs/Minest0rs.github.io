# Minest0rs.github.io
<!DOCTYPE html>
<html>
<head>
    <title>Легкий поиск</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 10px;
            background: #f0f0f0;
        }
        .search-box {
            margin: 20px auto;
            max-width: 300px;
        }
        input[type="text"] {
            width: 100%;
            padding: 8px;
            font-size: 14px;
            border: 1px solid #ccc;
        }
        button {
            margin-top: 5px;
            padding: 8px 15px;
            background: #0066cc;
            color: white;
            border: none;
            width: 100%;
        }
        .engine-btn {
            display: block;
            margin: 5px 0;
            padding: 5px;
            background: #e0e0e0;
            border: 1px solid #ccc;
            text-decoration: none;
            color: black;
            text-align: center;
        }
    </style>
</head>
<body>
    <h2>🔍 Легкий поиск</h2>
    <div class="search-box">
        <!-- DuckDuckGo Lite (самый легкий) -->
        <form action="https://lite.duckduckgo.com/lite/" method="GET">
            <input type="text" name="q" placeholder="Введите запрос..." required>
            <button type="submit">Искать в DuckDuckGo Lite</button>
        </form>
        <br>
        <!-- Старый Google -->
        <form action="https://www.google.com/search" method="GET">
            <input type="hidden" name="nfpr" value="1">
            <input type="text" name="q" placeholder="Тот же запрос в Google">
            <button type="submit" style="background:#4285f4">Искать в Google</button>
        </form>
    </div>
    <h3>Быстрый доступ:</h3>
    <a href="https://lite.duckduckgo.com/lite/" class="engine-btn">DuckDuckGo Lite</a>
    <a href="https://www.google.com/xhtml" class="engine-btn">Google XHTML</a>
    <a href="https://textise.iitty" class="engine-btn">Textise (упрощает сайты)</a>
</body>
</html>
