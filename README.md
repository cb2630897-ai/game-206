<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Institutional Trading Terminal v4.0</title>
    <style>
        :root {
            --bg-dark: #0b0e14;
            --panel-bg: #131722;
            --panel-border: #2a2e39;
            --text-main: #d1d4dc;
            --text-muted: #787b86;
            --green: #089981;
            --green-hover: #06705f;
            --green-alpha: rgba(8, 153, 129, 0.15);
            --red: #f23645;
            --red-hover: #b82834;
            --red-alpha: rgba(242, 54, 69, 0.15);
            --blue: #2962ff;
            --yellow: #f5c03b;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Trebuchet MS", Roboto, sans-serif; user-select: none; }
        body { background-color: var(--bg-dark); color: var(--text-main); height: 100vh; overflow: hidden; display: flex; flex-direction: column; }

        /* Top Bar */
        .top-bar { height: 42px; background: var(--panel-bg); border-bottom: 1px solid var(--panel-border); display: flex; align-items: center; justify-content: space-between; padding: 0 12px; font-size: 12px; }
        .symbol-info { display: flex; align-items: center; gap: 12px; }
        .ticker { font-weight: bold; font-size: 14px; color: #fff; display: flex; align-items: center; gap: 6px; }
        .badge { background: #2a2e39; color: var(--blue); padding: 1px 5px; border-radius: 3px; font-size: 10px; }
        .price-badge { font-family: 'Courier New', monospace; font-weight: bold; font-size: 14px; }

        .stat-group { display: flex; gap: 14px; }
        .stat-item { display: flex; flex-direction: column; }
        .stat-label { font-size: 9px; color: var(--text-muted); text-transform: uppercase; }
        .stat-val { font-size: 11px; font-weight: 600; font-family: monospace; }

        /* Workspace Grid */
        .workspace { flex: 1; display: grid; grid-template-columns: 44px 1fr 280px 240px; grid-template-rows: 1fr 180px; height: calc(100vh - 42px); }

        /* Toolbar */
        .toolbar { background: var(--panel-bg); border-right: 1px solid var(--panel-border); display: flex; flex-direction: column; align-items: center; padding-top: 8px; gap: 8px; grid-row: 1 / 3; }
        .tool-btn { width: 30px; height: 30px; border-radius: 4px; border: none; background: transparent; color: var(--text-muted); cursor: pointer; display: flex; align-items: center; justify-content: center; font-size: 14px; }
        .tool-btn:hover, .tool-btn.active { background: #2a2e39; color: var(--text-main); }

        /* Chart Area */
        .chart-viewport { position: relative; background: var(--bg-dark); grid-column: 2; grid-row: 1; overflow: hidden; }
        canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }

        /* Sidebar 1: Orders */
        .sidebar-order { background: var(--panel-bg); border-left: 1px solid var(--panel-border); grid-column: 3; grid-row: 1 / 3; display: flex; flex-direction: column; }
        
        /* Sidebar 2: Order Book & Tape */
        .sidebar-book { background: var(--panel-bg); border-left: 1px solid var(--panel-border); grid-column: 4; grid-row: 1 / 3; display: flex; flex-direction: column; }

        .tab-header { display: flex; border-bottom: 1px solid var(--panel-border); height: 32px; }
        .tab-btn { flex: 1; background: transparent; border: none; color: var(--text-muted); font-size: 11px; font-weight: bold; cursor: pointer; }
        .tab-btn.active { color: var(--blue); border-bottom: 2px solid var(--blue); background: rgba(41, 98, 255, 0.05); }

        .order-form { padding: 12px; display: flex; flex-direction: column; gap: 10px; }
        .form-row { display: flex; flex-direction: column; gap: 3px; }
        .form-row label { font-size: 10px; color: var(--text-muted); text-transform: uppercase; }
        .form-row input, .form-row select { background: var(--bg-dark); border: 1px solid var(--panel-border); color: #fff; padding: 6px 8px; border-radius: 4px; font-size: 12px; font-family: monospace; outline: none; }
        .form-row input:focus { border-color: var(--blue); }

        .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 6px; }

        .btn-trade { border: none; padding: 10px; border-radius: 4px; font-weight: bold; cursor: pointer; color: white; font-size: 12px; text-transform: uppercase; }
        .btn-buy { background: var(--green); }
        .btn-buy:hover { background: var(--green-hover); }
        .btn-sell { background: var(--red); }
        .btn-sell:hover { background: var(--red-hover); }

        /* Depth Order Book */
        .order-book { flex: 1; padding: 8px; display: flex; flex-direction: column; font-family: monospace; font-size: 10px; overflow: hidden; }
        .ob-header { display: flex; justify-content: space-between; color: var(--text-muted); padding-bottom: 4px; border-bottom: 1px solid var(--panel-border); margin-bottom: 4px; }
        .ob-row { display: flex; justify-content: space-between; position: relative; height: 15px; align-items: center; padding: 0 2px; }
        .ob-bar { position: absolute; right: 0; top: 0; bottom: 0; opacity: 0.2; pointer-events: none; }
        .ob-ask .ob-bar { background: var(--red); }
        .ob-bid .ob-bar { background: var(--green); }

        /* Time & Sales Tape */
        .trade-tape { height: 180px; border-top: 1px solid var(--panel-border); padding: 8px; font-family: monospace; font-size: 10px; overflow-y: hidden; display: flex; flex-direction: column; }
        .tape-row { display: flex; justify-content: space-between; height: 14px; align-items: center; }

        /* Bottom Positions & Orders Panel */
        .bottom-panel { background: var(--panel-bg); border-top: 1px solid var(--panel-border); grid-column: 2; grid-row: 2; display: flex; flex-direction: column; }
        .table-container { flex: 1; overflow-y: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 11px; text-align: left; }
        th { background: var(--bg-dark); color: var(--text-muted); padding: 6px 10px; font-weight: normal; position: sticky; top: 0; }
        td { padding: 6px 10px; border-bottom: 1px solid var(--panel-border); font-family: monospace; }
        .close-btn { background: #2a2e39; color: var(--text-main); border: none; padding: 2px 6px; border-radius: 3px; cursor: pointer; font-size: 10px; }
        .close-btn:hover { background: var(--red); color: white; }

        /* Utilities */
        .text-green { color: var(--green); }
        .text-red { color: var(--red); }
        .indicator-toggle { display: flex; gap: 6px; padding: 6px 12px; background: #181c27; border-bottom: 1px solid var(--panel-border); font-size: 10px; }
        .ind-chip { background: #2a2e39; padding: 2px 6px; border-radius: 3px; cursor: pointer; color: var(--text-muted); }
        .ind-chip.active { background: var(--blue); color: #fff; }
    </style>
</head>
<body>

    <!-- Header Bar -->
    <div class="top-bar">
        <div class="symbol-info">
            <div class="ticker">BTC/USD <span class="badge">PERP 100x</span></div>
            <div class="price-badge text-green" id="headerPrice">0.00</div>
        </div>
        <div class="stat-group">
            <div class="stat-item"><span class="stat-label">24h High</span><span class="stat-val" id="high24h">0.00</span></div>
            <div class="stat-item"><span class="stat-label">24h Low</span><span class="stat-val" id="low24h">0.00</span></div>
            <div class="stat-item"><span class="stat-label">24h Vol</span><span class="stat-val" id="vol24h">38,419 BTC</span></div>
            <div class="stat-item"><span class="stat-label">Wallet Balance</span><span class="stat-val text-green" id="accBalance">$100,000.00</span></div>
            <div class="stat-item"><span class="stat-label">Equity</span><span class="stat-val" id="accEquity">$100,000.00</span></div>
            <div class="stat-item"><span class="stat-label">Margin Used</span><span class="stat-val text-red" id="marginUsed">$0.00</span></div>
        </div>
    </div>

    <!-- Main Workspace -->
    <div class="workspace">
        
        <!-- Left Toolbar -->
        <div class="toolbar">
            <button class="tool-btn active" title="Crosshair">┼</button>
            <button class="tool-btn" title="Trend Line" id="toolLine" onclick="setTool('line')">╱</button>
            <button class="tool-btn" title="Horizontal Support/Resistance" id="toolRay" onclick="setTool('ray')">━</button>
            <button class="tool-btn" title="Clear Drawings" onclick="clearDrawings()">🗑️</button>
        </div>

        <!-- Center Chart Section -->
        <div class="chart-viewport" id="chartContainer">
            <div class="indicator-toggle">
                <span class="ind-chip active" id="chipEMA" onclick="toggleIndicator('EMA')">EMA (20)</span>
                <span class="ind-chip active" id="chipBB" onclick="toggleIndicator('BB')">Bollinger Bands</span>
                <span class="ind-chip active" id="chipRSI" onclick="toggleIndicator('RSI')">RSI Sub-pane</span>
            </div>
            <canvas id="mainChart"></canvas>
        </div>

        <!-- Sidebar 1: Order Execution -->
        <div class="sidebar-order">
            <div class="tab-header">
                <button class="tab-btn active">ORDER ENTRY</button>
            </div>

            <div class="order-form">
                <div class="form-row">
                    <label>Execution Mode</label>
                    <select id="orderType" onchange="toggleLimitPrice()">
                        <option value="MARKET">Market Order</option>
                        <option value="LIMIT">Limit Order</option>
                    </select>
                </div>

                <div class="form-row" id="limitPriceRow" style="display: none;">
                    <label>Limit Price ($)</label>
                    <input type="number" id="limitPrice" placeholder="0.00">
                </div>

                <div class="form-row">
                    <label>Leverage</label>
                    <select id="leverage">
                        <option value="1">1x (Spot)</option>
                        <option value="5">5x</option>
                        <option value="10" selected>10x</option>
                        <option value="25">25x</option>
                        <option value="50">50x</option>
                        <option value="100">100x</option>
                    </select>
                </div>

                <div class="form-row">
                    <label>Position Size (BTC)</label>
                    <input type="number" id="orderQty" value="1.0" step="0.1" min="0.01">
                </div>

                <div class="grid-2">
                    <div class="form-row">
                        <label>Take Profit ($)</label>
                        <input type="number" id="orderTP" placeholder="Target">
                    </div>
                    <div class="form-row">
                        <label>Stop Loss ($)</label>
                        <input type="number" id="orderSL" placeholder="Stop">
                    </div>
                </div>

                <div class="grid-2" style="margin-top: 6px;">
                    <button class="btn-trade btn-buy" onclick="executeOrder('BUY')">Buy / Long</button>
                    <button class="btn-trade btn-sell" onclick="executeOrder('SELL')">Sell / Short</button>
                </div>
            </div>
        </div>

        <!-- Sidebar 2: Live L2 Orderbook & Time/Sales -->
        <div class="sidebar-book">
            <div class="tab-header">
                <button class="tab-btn active">L2 DEPTH</button>
            </div>
            <div class="order-book">
                <div class="ob-header"><span>Price</span><span>Size</span><span>Total</span></div>
                <div id="askRows" style="display:flex; flex-direction:column-reverse;"></div>
                <div style="padding: 3px 0; font-weight:bold; text-align:center; border-top: 1px solid var(--panel-border); border-bottom: 1px solid var(--panel-border)" id="obSpread">-</div>
                <div id="bidRows"></div>
            </div>

            <div class="tab-header">
                <button class="tab-btn active">TIME & SALES</button>
            </div>
            <div class="trade-tape" id="tradeTape">
                <!-- Ticker prints live -->
            </div>
        </div>

        <!-- Bottom Panel: Active Positions & Pending Orders -->
        <div class="bottom-panel">
            <div class="tab-header">
                <button class="tab-btn active" id="tabPos" onclick="switchBottomTab('POS')">POSITIONS (<span id="posCount">0</span>)</button>
                <button class="tab-btn" id="tabOrders" onclick="switchBottomTab('ORDERS')">PENDING ORDERS (<span id="ordersCount">0</span>)</button>
            </div>
            <div class="table-container" id="posTableContainer">
                <table>
                    <thead>
                        <tr>
                            <th>Symbol</th>
                            <th>Side</th>
                            <th>Size</th>
                            <th>Entry</th>
                            <th>Mark</th>
                            <th>Liq. Price</th>
                            <th>Margin</th>
                            <th>TP / SL</th>
                            <th>Unrealized PNL</th>
                            <th>Action</th>
                        </tr>
                    </thead>
                    <tbody id="positionsTable"></tbody>
                </table>
            </div>
            <div class="table-container" id="ordersTableContainer" style="display: none;">
                <table>
                    <thead>
                        <tr>
                            <th>Symbol</th>
                            <th>Type</th>
                            <th>Side</th>
                            <th>Size</th>
                            <th>Trigger Price</th>
                            <th>Action</th>
                        </tr>
                    </thead>
                    <tbody id="ordersTable"></tbody>
                </table>
            </div>
        </div>
    </div>

<script>
    // State Engine
    let balance = 100000.00;
    let currentPrice = 65000.00;
    let candles = [];
    let positions = [];
    let pendingOrders = [];
    let drawings = [];
    let activeTool = null;
    let drawStart = null;

    // Technical Indicators State
    let showEMA = true;
    let showBB = true;
    let showRSI = true;

    // Canvas Engine Parameters
    let candleWidth = 7;
    let candleGap = 3;
    let panOffsetX = 0;
    let mouseX = 0, mouseY = 0;
    let isHoveringChart = false;
    let isMouseDown = false;

    // Canvas Element
    const container = document.getElementById('chartContainer');
    const canvas = document.getElementById('mainChart');
    const ctx = canvas.getContext('2d');

    function initCanvas() {
        canvas.width = container.clientWidth;
        canvas.height = container.clientHeight;
    }
    window.addEventListener('resize', () => { initCanvas(); draw(); });
    initCanvas();

    // Generate Initial Technical Data
    function generateInitialHistory() {
        let price = currentPrice;
        const now = Date.now();
        for (let i = 250; i >= 0; i--) {
            let volatility = (Math.random() - 0.495) * 50;
            let open = price;
            let close = open + volatility;
            let high = Math.max(open, close) + Math.random() * 20;
            let low = Math.min(open, close) - Math.random() * 20;
            let volume = Math.floor(Math.random() * 80) + 10;

            candles.push({ time: now - (i * 60000), open, high, low, close, volume });
            price = close;
        }
        currentPrice = price;
    }

    // Indicator Mathematical Calculators
    function calculateEMA(period) {
        let k = 2 / (period + 1);
        let ema = [];
        let prevEma = candles[0].close;
        for (let i = 0; i < candles.length; i++) {
            let val = (candles[i].close * k) + (prevEma * (1 - k));
            ema.push(val);
            prevEma = val;
        }
        return ema;
    }

    function calculateBB(period, stdDev) {
        let upper = [], lower = [], middle = [];
        for (let i = 0; i < candles.length; i++) {
            if (i < period - 1) {
                middle.push(null); upper.push(null); lower.push(null);
                continue;
            }
            let slice = candles.slice(i - period + 1, i + 1);
            let mean = slice.reduce((acc, c) => acc + c.close, 0) / period;
            let variance = slice.reduce((acc, c) => acc + Math.pow(c.close - mean, 2), 0) / period;
            let sd = Math.sqrt(variance);

            middle.push(mean);
            upper.push(mean + (sd * stdDev));
            lower.push(mean - (sd * stdDev));
        }
        return { upper, lower, middle };
    }

    function calculateRSI(period) {
        let rsi = [];
        let gains = 0, losses = 0;

        for (let i = 1; i <= period; i++) {
            let diff = candles[i].close - candles[i - 1].close;
            if (diff >= 0) gains += diff; else losses -= diff;
        }
        let avgGain = gains / period;
        let avgLoss = losses / period;
        rsi[period] = 100 - (100 / (1 + (avgGain / (avgLoss || 1))));

        for (let i = period + 1; i < candles.length; i++) {
            let diff = candles[i].close - candles[i - 1].close;
            let gain = diff >= 0 ? diff : 0;
            let loss = diff < 0 ? -diff : 0;

            avgGain = ((avgGain * (period - 1)) + gain) / period;
            avgLoss = ((avgLoss * (period - 1)) + loss) / period;

            let rs = avgGain / (avgLoss || 1);
            rsi.push(100 - (100 / (1 + rs)));
        }
        while (rsi.length < candles.length) rsi.unshift(50);
        return rsi;
    }

    // High Frequency Ticker Engine
    setInterval(() => {
        let lastCandle = candles[candles.length - 1];
        let tickDelta = (Math.random() - 0.495) * 8;
        
        currentPrice += tickDelta;
        lastCandle.close = currentPrice;
        if (currentPrice > lastCandle.high) lastCandle.high = currentPrice;
        if (currentPrice < lastCandle.low) lastCandle.low = currentPrice;
        lastCandle.volume += Math.abs(tickDelta) * 0.05;

        // Print Time & Sales Tape
        printTape(currentPrice, Math.abs(tickDelta * 0.1).toFixed(2), tickDelta >= 0);

        // Process Pending Limit Orders
        pendingOrders.forEach((ord, idx) => {
            if ((ord.side === 'BUY' && currentPrice <= ord.price) || (ord.side === 'SELL' && currentPrice >= ord.price)) {
                executeOrder(ord.side, ord.qty, ord.price, ord.leverage);
                pendingOrders.splice(idx, 1);
            }
        });

        // Construct New Candle Every 10 Ticks
        if (Math.random() < 0.1) {
            candles.push({
                time: Date.now(),
                open: currentPrice,
                high: currentPrice,
                low: currentPrice,
                close: currentPrice,
                volume: 0.1
            });
            if (candles.length > 500) candles.shift();
        }

        updateEngine();
        draw();
    }, 150);

    // Business Logic Engine
    function updateEngine() {
        document.getElementById('headerPrice').innerText = currentPrice.toFixed(2);
        
        let totalUnrealizedPNL = 0;
        let totalMarginUsed = 0;

        positions.forEach((pos, idx) => {
            let pnl = pos.type === 'BUY' ? (currentPrice - pos.entry) * pos.size : (pos.entry - currentPrice) * pos.size;
            pos.pnl = pnl;
            totalUnrealizedPNL += pnl;
            totalMarg