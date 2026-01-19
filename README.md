<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <!-- 手機優化Meta標籤 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0, minimum-scale=1.0">
    <meta name="format-detection" content="telephone=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="theme-color" content="#3498db">
    
    <title>生產累計量時間計算器</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link rel="apple-touch-icon" sizes="180x180" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>📊</text></svg>">
    
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Microsoft JhengHei', 'PingFang TC', sans-serif;
            -webkit-tap-highlight-color: transparent; /* 移除手機點擊藍色高亮 */
            -webkit-text-size-adjust: 100%; /* 防止手機橫向時文字放大 */
        }
        
        html {
            font-size: 16px;
            height: 100%;
            overflow-x: hidden;
        }
        
        body {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            color: #333;
            line-height: 1.5;
            min-height: 100vh;
            padding: 15px;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
        }
        
        /* 手機上的安全區域處理（針對iPhone X以上機型） */
        @supports (padding: max(0px)) {
            body {
                padding-left: max(15px, env(safe-area-inset-left));
                padding-right: max(15px, env(safe-area-inset-right));
                padding-top: max(15px, env(safe-area-inset-top));
                padding-bottom: max(15px, env(safe-area-inset-bottom));
            }
        }
        
        .container {
            max-width: 1000px;
            margin: 0 auto;
            padding: 10px;
        }
        
        header {
            text-align: center;
            margin-bottom: 25px;
            padding: 20px 15px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
        }
        
        h1 {
            color: #2c3e50;
            font-size: 1.8rem;
            margin-bottom: 8px;
            font-weight: 700;
        }
        
        .subtitle {
            color: #7f8c8d;
            font-size: 1rem;
            line-height: 1.4;
        }
        
        .calculator-card {
            background: white;
            border-radius: 12px;
            padding: 20px 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
            margin-bottom: 25px;
        }
        
        .formula-box {
            background: #f1f8ff;
            border-radius: 10px;
            padding: 15px;
            margin-bottom: 20px;
            border-left: 4px solid #3498db;
            text-align: center;
        }
        
        .formula-box h3 {
            color: #2c3e50;
            margin-bottom: 8px;
            font-size: 1.1rem;
        }
        
        .formula-text {
            font-family: 'Courier New', monospace;
            font-size: 1.1rem;
            background: white;
            padding: 12px;
            border-radius: 6px;
            color: #e74c3c;
            font-weight: bold;
            margin: 8px 0;
            word-break: break-word;
            overflow-wrap: break-word;
        }
        
        .input-grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 15px;
            margin-bottom: 15px;
        }
        
        .input-group {
            margin-bottom: 0;
        }
        
        label {
            display: block;
            margin-bottom: 6px;
            font-weight: 600;
            color: #2c3e50;
            font-size: 1rem;
        }
        
        .input-with-unit {
            position: relative;
            display: flex;
        }
        
        input {
            width: 100%;
            padding: 14px 12px;
            border: 2px solid #e0e0e0;
            border-radius: 8px;
            font-size: 1rem;
            transition: all 0.3s;
            -webkit-appearance: none; /* 移除iOS預設樣式 */
            appearance: none;
        }
        
        /* 改善手機上input的點擊體驗 */
        input:focus {
            outline: none;
            border-color: #3498db;
            box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
        }
        
        /* 防止iOS Safari自動放大input */
        @media screen and (max-width: 768px) {
            input, select, textarea {
                font-size: 16px !important; /* 防止iOS自動縮放 */
            }
        }
        
        .unit {
            position: absolute;
            right: 12px;
            top: 50%;
            transform: translateY(-50%);
            color: #7f8c8d;
            font-weight: 500;
            pointer-events: none; /* 防止點擊影響input */
        }
        
        .buttons {
            display: flex;
            flex-direction: column;
            gap: 12px;
            margin-top: 25px;
        }
        
        button {
            width: 100%;
            padding: 16px;
            border: none;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            min-height: 50px; /* 手機上易於點擊的最小高度 */
        }
        
        /* 改善手機按鈕觸控體驗 */
        button:active {
            transform: scale(0.98);
        }
        
        .calculate-btn {
            background: linear-gradient(135deg, #3498db 0%, #2980b9 100%);
            color: white;
        }
        
        .calculate-btn:hover {
            background: linear-gradient(135deg, #2980b9 0%, #21618c 100%);
        }
        
        .reset-btn {
            background: #f8f9fa;
            color: #495057;
            border: 2px solid #e9ecef;
        }
        
        .reset-btn:hover {
            background: #e9ecef;
        }
        
        .results-card {
            background: white;
            border-radius: 12px;
            padding: 20px 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
            display: none;
        }
        
        .results-card.active {
            display: block;
            animation: fadeIn 0.3s ease-out;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(15px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .result-title {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 20px;
            font-size: 1.5rem;
        }
        
        .result-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 12px;
            margin-bottom: 20px;
        }
        
        .result-box {
            background: #f8f9fa;
            border-radius: 8px;
            padding: 15px;
            text-align: center;
        }
        
        .result-box h3 {
            color: #7f8c8d;
            font-size: 0.85rem;
            margin-bottom: 8px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        
        .result-value {
            font-size: 1.5rem;
            font-weight: 700;
            color: #2c3e50;
        }
        
        .result-unit {
            font-size: 0.9rem;
            color: #7f8c8d;
            margin-left: 3px;
        }
        
        .calculation-steps {
            background: #f1f8ff;
            border-radius: 8px;
            padding: 20px 15px;
            margin-bottom: 20px;
            border-left: 4px solid #3498db;
        }
        
        .calculation-steps h3 {
            color: #2c3e50;
            margin-bottom: 12px;
            font-size: 1.2rem;
        }
        
        .step {
            margin-bottom: 12px;
            padding-bottom: 12px;
            border-bottom: 1px dashed #c3dafe;
        }
        
        .step:last-child {
            border-bottom: none;
        }
        
        .step-formula {
            font-family: 'Courier New', monospace;
            background: white;
            padding: 8px 12px;
            border-radius: 5px;
            margin: 6px 0;
            color: #2c3e50;
            font-size: 0.95rem;
            word-break: break-word;
        }
        
        .step-result {
            font-weight: 600;
            color: #e67e22;
            margin-left: 8px;
        }
        
        .time-details {
            background: #fff9e6;
            border-radius: 8px;
            padding: 20px 15px;
            border-left: 4px solid #f1c40f;
        }
        
        .time-details h3 {
            color: #d35400;
            margin-bottom: 12px;
            font-size: 1.2rem;
        }
        
        .arrival-time {
            font-size: 1.5rem;
            font-weight: 700;
            color: #e67e22;
            text-align: center;
            margin: 12px 0;
            padding: 12px;
            background: white;
            border-radius: 6px;
            border: 2px dashed #f39c12;
            word-break: break-word;
        }
        
        .current-time {
            font-size: 1rem;
            color: #7f8c8d;
            text-align: center;
            margin-top: 8px;
        }
        
        footer {
            text-align: center;
            margin-top: 30px;
            color: #7f8c8d;
            font-size: 0.85rem;
            padding: 15px 0;
        }
        
        .error-message {
            color: #e74c3c;
            font-weight: 600;
            margin-top: 10px;
            padding: 10px;
            background: #ffeaea;
            border-radius: 6px;
            display: none;
            font-size: 0.9rem;
        }
        
        .info-note {
            background: #e8f4fc;
            border-radius: 8px;
            padding: 12px;
            margin-top: 15px;
            font-size: 0.9rem;
            color: #2c3e50;
            border-left: 3px solid #3498db;
            line-height: 1.4;
        }
        
        .progress-container {
            margin: 15px 0;
        }
        
        .progress-bar {
            height: 16px;
            background: #ecf0f1;
            border-radius: 8px;
            overflow: hidden;
            margin-bottom: 8px;
        }
        
        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #2ecc71, #1abc9c);
            border-radius: 8px;
            width: 0%;
            transition: width 0.8s ease;
        }
        
        .progress-labels {
            display: flex;
            justify-content: space-between;
            font-size: 0.85rem;
            color: #7f8c8d;
        }
        
        /* 平板電腦樣式 */
        @media screen and (min-width: 600px) and (max-width: 900px) {
            .container {
                padding: 20px;
            }
            
            h1 {
                font-size: 2rem;
            }
            
            .calculator-card, .results-card {
                padding: 25px 20px;
            }
            
            .input-grid {
                grid-template-columns: repeat(2, 1fr);
            }
            
            .buttons {
                flex-direction: row;
            }
            
            .result-grid {
                grid-template-columns: repeat(3, 1fr);
            }
            
            .result-value {
                font-size: 1.6rem;
            }
        }
        
        /* 桌面電腦樣式 */
        @media screen and (min-width: 901px) {
            body {
                padding: 30px;
            }
            
            .container {
                padding: 20px;
            }
            
            h1 {
                font-size: 2.2rem;
            }
            
            .calculator-card, .results-card {
                padding: 30px 25px;
            }
            
            .input-grid {
                grid-template-columns: repeat(4, 1fr);
                gap: 20px;
            }
            
            .buttons {
                flex-direction: row;
            }
            
            button:hover {
                transform: translateY(-2px);
                box-shadow: 0 5px 12px rgba(0, 0, 0, 0.15);
            }
            
            button:active {
                transform: translateY(0);
            }
            
            .result-grid {
                grid-template-columns: repeat(6, 1fr);
                gap: 15px;
            }
            
            .result-value {
                font-size: 1.8rem;
            }
        }
        
        /* 橫向手機樣式 */
        @media screen and (max-height: 500px) and (orientation: landscape) {
            body {
                padding: 10px;
            }
            
            .container {
                max-width: 100%;
            }
            
            .calculator-card {
                margin-bottom: 15px;
                padding: 15px;
            }
            
            .input-grid {
                grid-template-columns: repeat(2, 1fr);
                gap: 10px;
            }
            
            .result-grid {
                grid-template-columns: repeat(3, 1fr);
                gap: 10px;
            }
            
            .result-box {
                padding: 10px;
            }
        }
        
        /* 超大手機樣式 */
        @media screen and (min-width: 400px) and (max-width: 599px) {
            .input-grid {
                grid-template-columns: repeat(2, 1fr);
            }
            
            .result-grid {
                grid-template-columns: repeat(3, 1fr);
            }
        }
        
        /* 超小手機樣式 */
        @media screen and (max-width: 350px) {
            body {
                padding: 10px;
            }
            
            h1 {
                font-size: 1.6rem;
            }
            
            .formula-text {
                font-size: 1rem;
                padding: 10px;
            }
            
            .result-grid {
                grid-template-columns: 1fr;
            }
        }
        
        /* 防止iOS Safari的彈跳效果 */
        body {
            overscroll-behavior-y: contain;
        }
        
        /* 改善滾動體驗 */
        .calculator-card, .results-card {
            -webkit-overflow-scrolling: touch;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1><i class="fas fa-industry"></i> 生產累計量時間計算器</h1>
            <p class="subtitle">計算完成生產目標所需的時間和預計完成時間</p>
        </header>
        
        <div class="calculator-card">
            <div class="formula-box">
                <h3><i class="fas fa-calculator"></i> 計算公式</h3>
                <div class="formula-text">所需時間 = [(昨日累計量 + 今日需求量) - 目前累計量] ÷ 目前流量</div>
                <p>計算剩餘產量所需時間，並預測完成時間</p>
            </div>
            
            <h2 style="color: #2c3e50; margin-bottom: 20px; text-align: center;">
                <i class="fas fa-edit"></i> 輸入生產數據
            </h2>
            
            <div class="input-grid">
                <div class="input-group">
                    <label for="yesterday-accumulated"><i class="fas fa-history"></i> 昨日累計量</label>
                    <div class="input-with-unit">
                        <input type="number" id="yesterday-accumulated" min="0" step="0.1" placeholder="例如：5000" inputmode="decimal">
                        <span class="unit">噸</span>
                    </div>
                </div>
                
                <div class="input-group">
                    <label for="daily-demand"><i class="fas fa-bullseye"></i> 今日需求量</label>
                    <div class="input-with-unit">
                        <input type="number" id="daily-demand" min="0" step="0.1" placeholder="例如：800" inputmode="decimal">
                        <span class="unit">噸</span>
                    </div>
                </div>
                
                <div class="input-group">
                    <label for="current-accumulated"><i class="fas fa-chart-line"></i> 目前累計量</label>
                    <div class="input-with-unit">
                        <input type="number" id="current-accumulated" min="0" step="0.1" placeholder="例如：5300" inputmode="decimal">
                        <span class="unit">噸</span>
                    </div>
                    <small style="color: #7f8c8d; margin-top: 5px; display: block; font-size: 0.85rem;">註：從開始到現在的總累計量（含昨日）</small>
                </div>
                
                <div class="input-group">
                    <label for="flow"><i class="fas fa-tachometer-alt"></i> 目前流量</label>
                    <div class="input-with-unit">
                        <input type="number" id="flow" min="0.1" step="0.1" placeholder="例如：50" inputmode="decimal">
                        <span class="unit">噸/小時</span>
                    </div>
                </div>
            </div>
            
            <div class="progress-container">
                <div class="progress-bar">
                    <div class="progress-fill" id="progress-fill"></div>
                </div>
                <div class="progress-labels">
                    <span id="progress-start">昨日: 0 噸</span>
                    <span id="progress-current">目前: 0 噸</span>
                    <span id="progress-target">目標: 0 噸</span>
                </div>
            </div>
            
            <div class="info-note">
                <i class="fas fa-info-circle"></i> 
                <strong>計算說明：</strong> 今日目標總量 = 昨日累計量 + 今日需求量，剩餘量 = 目標總量 - 目前累計量，所需時間 = 剩餘量 ÷ 目前流量
            </div>
            
            <div class="error-message" id="error-message">
                <i class="fas fa-exclamation-triangle"></i> 
                <span id="error-text">請輸入有效的數字，且流量不能為0，目前累計量應大於等於昨日累計量</span>
            </div>
            
            <div class="buttons">
                <button class="calculate-btn" id="calculate-btn">
                    <i class="fas fa-calculator"></i> 計算所需時間
                </button>
                <button class="reset-btn" id="reset-btn">
                    <i class="fas fa-redo"></i> 重置
                </button>
            </div>
        </div>
        
        <div class="results-card" id="results-card">
            <h2 class="result-title"><i class="fas fa-clock"></i> 計算結果</h2>
            
            <div class="result-grid">
                <div class="result-box">
                    <h3>昨日累計量</h3>
                    <div>
                        <span class="result-value" id="result-yesterday">0</span>
                        <span class="result-unit">噸</span>
                    </div>
                </div>
                
                <div class="result-box">
                    <h3>今日需求量</h3>
                    <div>
                        <span class="result-value" id="result-demand">0</span>
                        <span class="result-unit">噸</span>
                    </div>
                </div>
                
                <div class="result-box">
                    <h3>目前累計量</h3>
                    <div>
                        <span class="result-value" id="result-current">0</span>
                        <span class="result-unit">噸</span>
                    </div>
                </div>
                
                <div class="result-box">
                    <h3>目前流量</h3>
                    <div>
                        <span class="result-value" id="result-flow">0</span>
                        <span class="result-unit">噸/小時</span>
                    </div>
                </div>
                
                <div class="result-box">
                    <h3>剩餘量</h3>
                    <div>
                        <span class="result-value" id="result-remaining">0</span>
                        <span class="result-unit">噸</span>
                    </div>
                </div>
                
                <div class="result-box">
                    <h3>所需時間</h3>
                    <div>
                        <span class="result-value" id="result-time">0</span>
                        <span class="result-unit" id="result-time-unit">小時</span>
                    </div>
                </div>
            </div>
            
            <div class="calculation-steps">
                <h3><i class="fas fa-calculator"></i> 計算步驟詳解</h3>
                
                <div class="step">
                    <p><strong>步驟 1:</strong> 計算今日目標總量</p>
                    <div class="step-formula">
                        目標總量 = 昨日累計量 + 今日需求量 = <span id="step-yesterday">0</span> + <span id="step-demand">0</span>
                    </div>
                    <div>結果: <span class="step-result" id="step-target">0 噸</span></div>
                </div>
                
                <div class="step">
                    <p><strong>步驟 2:</strong> 計算剩餘需要完成的量</p>
                    <div class="step-formula">
                        剩餘量 = 目標總量 - 目前累計量 = <span id="step-target-display">0</span> - <span id="step-current-display">0</span>
                    </div>
                    <div>結果: <span class="step-result" id="step-remaining">0 噸</span></div>
                </div>
                
                <div class="step">
                    <p><strong>步驟 3:</strong> 計算所需時間</p>
                    <div class="step-formula">
                        所需時間 = 剩餘量 ÷ 目前流量 = <span id="step-remaining-display">0</span> ÷ <span id="step-flow">0</span>
                    </div>
                    <div>結果: <span class="step-result" id="step-time">0 小時</span></div>
                </div>
            </div>
            
            <div class="time-details">
                <h3><i class="fas fa-calendar-alt"></i> 時間預測</h3>
                <p>根據目前進度，完成今日生產目標的時間是：</p>
                <div class="arrival-time" id="arrival-time">--</div>
                <div class="current-time">現在時間：<span id="current-time">--</span></div>
            </div>
        </div>
        
        <footer>
            <p>生產累計量時間計算器 &copy; 2023 | 專為生產排程與時間預測設計</p>
            <p style="font-size: 0.75rem; margin-top: 5px;">手機優化版本</p>
        </footer>
    </div>

    <script>
        // 保持原有的JavaScript代碼不變，因為它已經支援手機操作
        document.addEventListener('DOMContentLoaded', function() {
            // 獲取DOM元素
            const yesterdayInput = document.getElementById('yesterday-accumulated');
            const demandInput = document.getElementById('daily-demand');
            const currentInput = document.getElementById('current-accumulated');
            const flowInput = document.getElementById('flow');
            const calculateBtn = document.getElementById('calculate-btn');
            const resetBtn = document.getElementById('reset-btn');
            const resultsCard = document.getElementById('results-card');
            const errorMessage = document.getElementById('error-message');
            const errorText = document.getElementById('error-text');
            
            // 結果顯示元素
            const resultYesterday = document.getElementById('result-yesterday');
            const resultDemand = document.getElementById('result-demand');
            const resultCurrent = document.getElementById('result-current');
            const resultFlow = document.getElementById('result-flow');
            const resultRemaining = document.getElementById('result-remaining');
            const resultTime = document.getElementById('result-time');
            const resultTimeUnit = document.getElementById('result-time-unit');
            
            // 計算步驟元素
            const stepYesterday = document.getElementById('step-yesterday');
            const stepDemand = document.getElementById('step-demand');
            const stepTarget = document.getElementById('step-target');
            const stepTargetDisplay = document.getElementById('step-target-display');
            const stepCurrentDisplay = document.getElementById('step-current-display');
            const stepRemaining = document.getElementById('step-remaining');
            const stepRemainingDisplay = document.getElementById('step-remaining-display');
            const stepFlow = document.getElementById('step-flow');
            const stepTime = document.getElementById('step-time');
            
            // 時間顯示元素
            const arrivalTime = document.getElementById('arrival-time');
            const currentTime = document.getElementById('current-time');
            
            // 進度條元素
            const progressFill = document.getElementById('progress-fill');
            const progressStart = document.getElementById('progress-start');
            const progressCurrent = document.getElementById('progress-current');
            const progressTarget = document.getElementById('progress-target');
            
            // 更新當前時間
            function updateCurrentTime() {
                const now = new Date();
                const formattedTime = now.toLocaleString('zh-TW', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric',
                    weekday: 'long',
                    hour: '2-digit',
                    minute: '2-digit',
                    hour12: false
                });
                currentTime.textContent = formattedTime;
            }
            
            // 初始顯示當前時間
            updateCurrentTime();
            // 每秒更新一次時間
            setInterval(updateCurrentTime, 1000);
            
            // 更新進度條
            function updateProgressBar(yesterday, demand, current) {
                const start = yesterday;
                const target = yesterday + demand;
                
                // 更新標籤
                progressStart.textContent = "昨日: " + start.toLocaleString('zh-TW') + " 噸";
                progressCurrent.textContent = "目前: " + current.toLocaleString('zh-TW') + " 噸";
                progressTarget.textContent = "目標: " + target.toLocaleString('zh-TW') + " 噸";
                
                // 計算進度百分比
                let progressPercent = 0;
                if (target > start) {
                    progressPercent = Math.min(100, ((current - start) / demand) * 100);
                }
                
                // 如果目前累計量已超過目標，顯示100%
                if (current >= target) {
                    progressPercent = 100;
                }
                
                // 更新進度條
                setTimeout(() => {
                    progressFill.style.width = progressPercent + '%';
                }, 100);
            }
            
            // 計算按鈕點擊事件 - 優化手機觸控
            calculateBtn.addEventListener('click', function() {
                // 獲取輸入值
                const yesterday = parseFloat(yesterdayInput.value) || 0;
                const demand = parseFloat(demandInput.value);
                const current = parseFloat(currentInput.value) || 0;
                const flow = parseFloat(flowInput.value);
                
                // 驗證輸入
                if (isNaN(demand) || isNaN(flow) || flow === 0) {
                    errorText.textContent = "請輸入有效的數字，且流量不能為0";
                    errorMessage.style.display = 'block';
                    resultsCard.classList.remove('active');
                    return;
                }
                
                // 檢查目前累計量是否小於昨日累計量
                if (current < yesterday) {
                    errorText.textContent = "目前累計量不能小於昨日累計量";
                    errorMessage.style.display = 'block';
                    resultsCard.classList.remove('active');
                    return;
                }
                
                // 隱藏錯誤訊息
                errorMessage.style.display = 'none';
                
                // 計算目標總量
                const targetTotal = yesterday + demand;
                
                // 計算剩餘需要完成的量
                const remaining = targetTotal - current;
                
                // 如果剩餘量為0或負數，表示已經完成
                if (remaining <= 0) {
                    resultTime.textContent = "0";
                    resultTimeUnit.textContent = "小時";
                    resultRemaining.textContent = "0";
                    
                    arrivalTime.textContent = "已完成目標！";
                    arrivalTime.style.color = "#2ecc71";
                    
                    // 更新進度條
                    updateProgressBar(yesterday, demand, current);
                    
                    // 顯示結果卡片
                    resultsCard.classList.add('active');
                    
                    // 手機上滾動到結果
                    setTimeout(() => {
                        resultsCard.scrollIntoView({ behavior: 'smooth', block: 'start' });
                    }, 100);
                    
                    return;
                }
                
                // 計算所需時間
                const timeRequired = remaining / flow;
                
                // 計算到達時間
                const now = new Date();
                const arrival = new Date(now.getTime() + (timeRequired * 60 * 60 * 1000));
                
                // 格式化到達時間
                const formattedArrival = arrival.toLocaleString('zh-TW', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric',
                    weekday: 'long',
                    hour: '2-digit',
                    minute: '2-digit',
                    hour12: false
                });
                
                // 顯示結果
                resultYesterday.textContent = yesterday.toLocaleString('zh-TW');
                resultDemand.textContent = demand.toLocaleString('zh-TW');
                resultCurrent.textContent = current.toLocaleString('zh-TW');
                resultFlow.textContent = flow.toLocaleString('zh-TW');
                resultRemaining.textContent = remaining.toLocaleString('zh-TW');
                
                // 顯示計算步驟
                stepYesterday.textContent = yesterday.toLocaleString('zh-TW');
                stepDemand.textContent = demand.toLocaleString('zh-TW');
                stepTarget.textContent = targetTotal.toLocaleString('zh-TW') + " 噸";
                stepTargetDisplay.textContent = targetTotal.toLocaleString('zh-TW');
                stepCurrentDisplay.textContent = current.toLocaleString('zh-TW');
                stepRemaining.textContent = remaining.toLocaleString('zh-TW') + " 噸";
                stepRemainingDisplay.textContent = remaining.toLocaleString('zh-TW');
                stepFlow.textContent = flow.toLocaleString('zh-TW');
                stepTime.textContent = timeRequired.toFixed(2) + " 小時";
                
                // 根據時間長度選擇合適的顯示方式
                let displayTime, displayUnit;
                
                if (timeRequired < 1) {
                    // 少於1小時，顯示分鐘
                    displayTime = Math.round(timeRequired * 60);
                    displayUnit = '分鐘';
                } else if (timeRequired < 24) {
                    // 少於1天，顯示小時（保留1位小數）
                    displayTime = Math.round(timeRequired * 10) / 10;
                    displayUnit = '小時';
                } else {
                    // 超過1天，顯示天數（保留1位小數）
                    displayTime = Math.round((timeRequired / 24) * 10) / 10;
                    displayUnit = '天';
                }
                
                resultTime.textContent = displayTime;
                resultTimeUnit.textContent = displayUnit;
                
                arrivalTime.textContent = formattedArrival;
                arrivalTime.style.color = "#e67e22";
                
                // 更新進度條
                updateProgressBar(yesterday, demand, current);
                
                // 顯示結果卡片
                resultsCard.classList.add('active');
                
                // 手機上滾動到結果
                setTimeout(() => {
                    resultsCard.scrollIntoView({ behavior: 'smooth', block: 'start' });
                }, 100);
            });
            
            // 重置按鈕點擊事件
            resetBtn.addEventListener('click', function() {
                // 清空輸入
                yesterdayInput.value = '';
                demandInput.value = '';
                currentInput.value = '';
                flowInput.value = '';
                
                // 重置進度條
                progressFill.style.width = '0%';
                progressStart.textContent = '昨日: 0 噸';
                progressCurrent.textContent = '目前: 0 噸';
                progressTarget.textContent = '目標: 0 噸';
                
                // 隱藏錯誤訊息和結果
                errorMessage.style.display = 'none';
                resultsCard.classList.remove('active');
                
                // 聚焦到第一個輸入框
                yesterdayInput.focus();
            });
            
            // 輸入時隱藏錯誤訊息並更新進度條
            function handleInputChange() {
                errorMessage.style.display = 'none';
                
                // 嘗試更新進度條
                const yesterday = parseFloat(yesterdayInput.value) || 0;
                const demand = parseFloat(demandInput.value) || 0;
                const current = parseFloat(currentInput.value) || 0;
                
                if (demand > 0 && current >= yesterday) {
                    updateProgressBar(yesterday, demand, current);
                }
            }
            
            yesterdayInput.addEventListener('input', handleInputChange);
            demandInput.addEventListener('input', handleInputChange);
            currentInput.addEventListener('input', handleInputChange);
            flowInput.addEventListener('input', handleInputChange);
            
            // 按Enter鍵也可以計算
            const inputs = [yesterdayInput, demandInput, currentInput, flowInput];
            inputs.forEach(input => {
                input.addEventListener('keypress', function(e) {
                    if (e.key === 'Enter') {
                        calculateBtn.click();
                    }
                });
            });
            
            // 提供示例數據
            yesterdayInput.value = "5000";
            demandInput.value = "800";
            currentInput.value = "5300";
            flowInput.value = "50";
            
            // 初始更新進度條
            updateProgressBar(5000, 800, 5300);
            
            // 初始計算
            setTimeout(() => {
                calculateBtn.click();
            }, 500);
            
            // 手機虛擬鍵盤彈出時調整布局
            window.addEventListener('resize', function() {
                // 當鍵盤彈出時，確保焦點元素可見
                if (document.activeElement && document.activeElement.tagName === 'INPUT') {
                    setTimeout(() => {
                        document.activeElement.scrollIntoView({ behavior: 'smooth', block: 'center' });
                    }, 300);
                }
            });
        });
    </script>
</body>
</html>
