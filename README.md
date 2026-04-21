<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>抽籤機</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@700;900&display=swap');
        
        .reel {
            height: 220px;
            overflow: hidden;
            background: linear-gradient(#fff, #f8fafc, #fff);
            box-shadow: inset 0 0 30px rgba(0,0,0,0.3);
            border: 6px solid #1f2937;
            border-radius: 16px;
        }
        
        .reel-inner {
            display: flex;
            flex-direction: column;
            transition: transform 0ms linear;
            font-size: 5.2rem;
            font-weight: 900;
            line-height: 1;
        }
        
        .symbol {
            height: 220px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-bottom: 4px solid #e5e7eb;
        }
        
        @keyframes pop {
            0% { transform: scale(0.8); }
            50% { transform: scale(1.12); }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body class="min-h-screen bg-gradient-to-br from-amber-400 via-red-500 to-rose-600 flex items-center justify-center p-6">
    <div class="max-w-lg w-full bg-white rounded-3xl shadow-2xl overflow-hidden">
        <!-- 標題 -->
        <div class="bg-gradient-to-r from-red-600 to-amber-600 text-white py-8 text-center">
            <h1 class="text-5xl font-black tracking-widest">拉霸抽籤機</h1>
        </div>

        <!-- 拉霸機本體 -->
        <div class="p-8 bg-gray-900 rounded-t-3xl">
            <div class="flex gap-4 justify-center" id="reels">
                <!-- 三個輪子由 JavaScript 產生 -->
            </div>
        </div>

        <!-- 控制區 -->
        <div class="px-8 py-10 bg-white">
            <button id="drawBtn"
                    class="w-full py-8 bg-gradient-to-r from-red-600 to-amber-600 hover:from-red-700 hover:to-amber-700 text-white text-4xl font-black rounded-2xl shadow-xl active:scale-95 transition-all duration-200 flex items-center justify-center gap-4">
                <span id="btnText">🎰 開始抽籤</span>
            </button>
            
            <p id="status" class="text-center text-lg font-medium text-gray-600 mt-6 h-12"></p>
        </div>

        <div class="text-center text-xs text-gray-400 pb-6">
            已修正：三輪停止速度更接近 • 消除空白畫面 • 最後一個字不會突兀
        </div>
    </div>

    <script>
        // ===== 在這裡修改你們三個人的真實完整姓名（一定要剛好三個字）=====
        const participants = [
            { fullName: "詹凱程", chars: ["詹", "凱", "程"] },
            { fullName: "陳彗萱", chars: ["陳", "彗", "萱"] },
            { fullName: "潘英俊", chars: ["潘", "英", "俊"] }
        ];

        const allSymbols = [...new Set(participants.flatMap(p => p.chars))];
        const symbolHeight = 220;
        const cycleLength = allSymbols.length;

        let isSpinning = false;

        function createReel() {
            const reel = document.createElement('div');
            reel.className = 'reel flex-1';
            
            const inner = document.createElement('div');
            inner.className = 'reel-inner';
            
            // 更多圈數，讓滾動更順暢自然
            const numCycles = 35;
            for (let i = 0; i < numCycles; i++) {
                allSymbols.forEach(char => {
                    const symbol = document.createElement('div');
                    symbol.className = 'symbol';
                    symbol.textContent = char;
                    inner.appendChild(symbol);
                });
            }
            
            reel.appendChild(inner);
            return reel;
        }

        function initReels() {
            const reelsContainer = document.getElementById('reels');
            reelsContainer.innerHTML = '';
            
            const initialSurnames = participants.map(p => p.chars[0]);
            
            for (let i = 0; i < 3; i++) {
                const reel = createReel();
                const inner = reel.querySelector('.reel-inner');
                
                const posInCycle = allSymbols.indexOf(initialSurnames[i]);
                const initOffset = (8 * cycleLength + posInCycle) * symbolHeight;
                
                inner.style.transition = 'none';
                inner.style.transform = `translateY(-${initOffset}px)`;
                reelsContainer.appendChild(reel);
            }
        }

        function startSlotAnimation() {
            if (isSpinning) return;
            isSpinning = true;
            
            const btn = document.getElementById('drawBtn');
            const btnText = document.getElementById('btnText');
            const status = document.getElementById('status');
            const reelsContainer = document.getElementById('reels');
            
            btn.disabled = true;
            btnText.innerHTML = '🎰 抽籤中...';
            status.textContent = '';
            
            reelsContainer.innerHTML = '';
            const reels = [];
            
            // 決定這次中獎者
            const winnerIndex = Math.floor(Math.random() * participants.length);
            const winner = participants[winnerIndex];
            const finalChars = winner.chars;
            
            // 建立三個輪子 + 設定初始「詹　陳　潘」
            for (let i = 0; i < 3; i++) {
                const reel = createReel();
                reelsContainer.appendChild(reel);
                reels.push(reel);
                
                const inner = reel.querySelector('.reel-inner');
                const initChar = participants[i].chars[0];
                const posInCycle = allSymbols.indexOf(initChar);
                const initOffset = (8 * cycleLength + posInCycle) * symbolHeight;
                
                inner.style.transition = 'none';
                inner.style.transform = `translateY(-${initOffset}px)`;
                void inner.offsetWidth; // 強制重繪
            }
            
            let finishedCount = 0;
            
            reels.forEach((reel, index) => {
                const inner = reel.querySelector('.reel-inner');
                
                // 減少最後一輪的延遲差（原本差太多，現在三輪速度更接近）
                const baseDuration = 1000;           // 基礎時間
                const stagger = index * 1000;
                const duration = baseDuration + stagger;
                
                // 計算滾動圈數 + 最終要停的字
                const desiredChar = finalChars[index];
                const spins = 22 + index * 3;        // 更多圈，更刺激
                const posInCycle = allSymbols.indexOf(desiredChar);
                const targetOffset = (spins * cycleLength + posInCycle) * symbolHeight;
                
                // 設定動畫
                inner.style.transition = `transform ${duration}ms cubic-bezier(0.25, 0.1, 0.1, 1)`;
                inner.style.transform = `translateY(-${targetOffset}px)`;
                
                // 關鍵修正：當每一個輪子「真正停止」時，立刻鎖定位置（消除空白畫面）
                const lockWhenFinished = () => {
                    inner.style.transition = 'none';
                    // 強制鎖定在精準符號位置
                    const finalOffset = posInCycle * symbolHeight;
                    inner.style.transform = `translateY(-${finalOffset}px)`;
                    
                    finishedCount++;
                    
                    // 全部輪子都停完後才顯示結果
                    if (finishedCount === 3) {
                        status.innerHTML = `
                            🎉 恭喜！<br>
                            <span class="text-red-600 text-4xl font-black">${winner.fullName}</span>
                        `;
                        btn.disabled = false;
                        btnText.innerHTML = '🎰 再抽一次';
                        isSpinning = false;
                    }
                };
                
                inner.addEventListener('transitionend', lockWhenFinished, { once: true });
            });
        }
        
        // 啟動
        document.addEventListener('DOMContentLoaded', () => {
            initReels();
            
            document.getElementById('drawBtn').addEventListener('click', startSlotAnimation);
            
            document.addEventListener('keydown', (e) => {
                if (e.key === 'Enter' && !isSpinning) {
                    startSlotAnimation();
                }
            });
            
            console.log('%c✅ 已完全修正！最後一輪不再明顯變慢 + 消除空白突兀畫面', 'color:#f59e0b; font-weight:bold');
        });
    </script>
</body>
</html>
