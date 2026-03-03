<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PowerSignal Pro AI Trading Bot</title>

<script src="https://cdn.jsdelivr.net/npm/technicalindicators@3.0.0/dist/browser.js"></script>
<script src="https://s3.tradingview.com/tv.js"></script>

<style>
body{
    margin:0;
    font-family:Arial;
    background:#0f172a;
    color:white;
}

header{
    background:#111827;
    padding:15px;
    text-align:center;
    font-size:22px;
    font-weight:bold;
}

.container{
    padding:20px;
}

.card{
    background:#1e293b;
    padding:20px;
    border-radius:10px;
    margin-bottom:20px;
}

.buy{color:#00ff88;font-weight:bold;}
.sell{color:#ff4d4d;font-weight:bold;}
.hold{color:#facc15;font-weight:bold;}

button{
    padding:10px 15px;
    background:#2563eb;
    border:none;
    color:white;
    border-radius:5px;
    cursor:pointer;
}

button:hover{
    background:#1d4ed8;
}
</style>
</head>

<body>

<header>
🚀 PowerSignal PRO AI Trading Bot
</header>

<div class="container">

<div class="card">
<h2>📊 Live Signal Engine</h2>
<p>Symbol:
<select id="symbol">
<option value="BTCUSDT">BTCUSDT</option>
<option value="ETHUSDT">ETHUSDT</option>
<option value="BNBUSDT">BNBUSDT</option>
</select>
</p>

<p id="signalResult">Loading...</p>
<button onclick="analyzeMarket()">Analyze Market</button>
</div>

<div id="chart"></div>

</div>

<script>
new TradingView.widget({
    "width": "100%",
    "height": 500,
    "symbol": "BINANCE:BTCUSDT",
    "interval": "15",
    "theme": "dark",
    "container_id": "chart"
});

// Notification permission
if (Notification.permission !== "granted") {
    Notification.requestPermission();
}

async function analyzeMarket(){

    const symbol = document.getElementById("symbol").value;

    const response = await fetch(
        `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=15m&limit=100`
    );
    const data = await response.json();

    const closes = data.map(c => parseFloat(c[4]));

    const rsi = window.technicalindicators.RSI.calculate({
        values: closes,
        period: 14
    });

    const emaFast = window.technicalindicators.EMA.calculate({
        values: closes,
        period: 9
    });

    const emaSlow = window.technicalindicators.EMA.calculate({
        values: closes,
        period: 21
    });

    const lastPrice = closes[closes.length-1];
    const lastRSI = rsi[rsi.length-1];
    const lastFast = emaFast[emaFast.length-1];
    const lastSlow = emaSlow[emaSlow.length-1];

    let signal = "HOLD";

    if(lastRSI < 30 && lastFast > lastSlow){
        signal = "BUY";
    }
    else if(lastRSI > 70 && lastFast < lastSlow){
        signal = "SELL";
    }

    let sl = signal==="BUY" ? lastPrice-200 : lastPrice+200;
    let tp = signal==="BUY" ? lastPrice+400 : lastPrice-400;

    let colorClass = signal.toLowerCase();

    document.getElementById("signalResult").innerHTML =
        `<span class="${colorClass}">${signal}</span>
        <br>Entry: ${lastPrice.toFixed(2)}
        <br>Stop Loss: ${sl.toFixed(2)}
        <br>Take Profit: ${tp.toFixed(2)}
        <br>RSI: ${lastRSI.toFixed(2)}`;

    if(signal !== "HOLD"){
        new Notification("New Trading Signal!",{
            body:`${signal} ${symbol} at ${lastPrice}`
        });
    }
}

analyzeMarket();
</script>

</body>
</html>
