<!DOCTYPE html>

<html>
<head>
    <base target="_top">
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700&display=swap');
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background: linear-gradient(180deg, #fef3c7 0%, #fffbeb 100%); 
            min-height: 100vh; 
            margin: 0; 
        }
        .bounce { animation: bounce 2s infinite; }
        @keyframes bounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-15px); } }
        .progress-bar { transition: width 0.5s ease-in-out; }
        #startScreen { transition: all 0.5s ease; }
        .fade-out { opacity: 0; visibility: hidden; transform: scale(1.1); }
    </style>
</head>
<body class="antialiased">

    <!-- 開始介面 -->
    <div id="startScreen" class="fixed inset-0 z-50 flex flex-col items-center justify-center bg-white p-6 text-center">
        <div class="mb-10">
            <div class="text-9xl mb-6 bounce">🐤</div>
            <h1 class="text-5xl font-black text-amber-800 tracking-wider">心靈小雞</h1>
            <p class="text-amber-600 mt-3 text-lg font-bold">你的專屬雲端日記伴侶</p>
        </div>
        <button onclick="startGame()" class="bg-amber-500 hover:bg-amber-600 text-white font-black py-4 px-14 rounded-full text-2xl shadow-2xl transition transform active:scale-95">
            進入養殖場
        </button>
        <p class="mt-8 text-gray-400 text-sm">數據將即時同步至 Google 試算表</p>
    </div>

    <!-- 遊戲主內容 (預設隱藏) -->
    <div id="gameContent" class="hidden p-4 max-w-md mx-auto pt-8">
        <!-- 頂部狀態欄 -->
        <div class="bg-white/80 backdrop-blur-md p-5 rounded-3xl shadow-sm border-b-4 border-amber-200 flex justify-between items-center mb-8">
            <div>
                <h2 class="text-xl font-bold text-amber-800">養殖場</h2>
                <div class="flex items-center text-[10px] text-green-600 font-bold">
                    <span class="w-2 h-2 bg-green-500 rounded-full mr-1 animate-pulse"></span>
                    雲端連線中
                </div>
            </div>
            <div class="text-right">
                <span class="text-amber-600 font-black text-2xl">Lv.<span id="lvDisplay">1</span></span>
            </div>
        </div>

        <!-- 小雞展示區 -->
        <div class="flex flex-col items-center justify-center my-12">
            <div id="chicken" class="text-9xl bounce">🥚</div>
            <div class="w-full max-w-[250px] bg-amber-100 h-3 rounded-full mt-8 overflow-hidden shadow-inner">
                <div id="expBar" class="bg-amber-500 h-full progress-bar" style="width: 0%"></div>
            </div>
            <p class="text-amber-700/50 text-xs mt-3 font-bold">經驗值: <span id="expText">0</span>/10</p>
        </div>

        <!-- 功能切換按鈕 -->
        <div class="flex space-x-2 mb-6">
            <button onclick="switchTab('write')" id="btnWrite" class="flex-1 py-3 rounded-2xl font-bold bg-amber-500 text-white shadow-md">餵食小雞</button>
            <button onclick="switchTab('history')" id="btnHistory" class="flex-1 py-3 rounded-2xl font-bold bg-white text-amber-500 border border-amber-100">回顧日記</button>
        </div>

        <!-- 寫作區域 -->
        <div id="writeSection" class="bg-white p-6 rounded-3xl shadow-xl space-y-4 border border-amber-50">
            <textarea id="diaryInput" class="w-full h-32 p-4 rounded-2xl bg-amber-50/30 border-2 border-amber-50 focus:border-amber-400 outline-none resize-none text-gray-700" placeholder="今天心情好嗎？寫下來餵食小雞..."></textarea>
            <button onclick="handleSave()" id="saveBtn" class="w-full bg-amber-500 hover:bg-amber-600 text-white font-black py-4 rounded-2xl shadow-lg transition-all active:scale-95">
                同步紀錄並餵食
            </button>
        </div>

        <!-- 歷史區域 (預設隱藏) -->
        <div id="historySection" class="hidden space-y-4 pb-10">
            <div id="logList" class="space-y-4">
                <p class="text-center text-amber-800/40 text-sm py-10">正在同步雲端資料...</p>
            </div>
        </div>
    </div>

    <script>
        // 本地存儲等級數據
        let stats = JSON.parse(localStorage.getItem('chicken_v4_data')) || { lv: 1, exp: 0 };

        // 進入遊戲按鈕
        function startGame() {
            document.getElementById('startScreen').classList.add('fade-out');
            setTimeout(() => {
                document.getElementById('startScreen').style.display = 'none';
                document.getElementById('gameContent').classList.remove('hidden');
                updateUI();
                loadLogs();
            }, 500);
        }

        // 更新介面
        function updateUI() {
            document.getElementById('lvDisplay').textContent = stats.lv;
            document.getElementById('expText').textContent = stats.exp;
            document.getElementById('expBar').style.width = (stats.exp * 10) + '%';
            
            const chicken = document.getElementById('chicken');
            if (stats.lv < 2) chicken.textContent = '🥚';
            else if (stats.lv < 5) chicken.textContent = '🐣';
            else if (stats.lv < 10) chicken.textContent = '🐥';
            else chicken.textContent = '🐓';
            
            localStorage.setItem('chicken_v4_data', JSON.stringify(stats));
        }

        // 切換分頁
        function switchTab(tab) {
            const isWrite = tab === 'write';
            document.getElementById('writeSection').classList.toggle('hidden', !isWrite);
            document.getElementById('historySection').classList.toggle('hidden', isWrite);
            
            document.getElementById('btnWrite').className = isWrite ? 'flex-1 py-3 rounded-2xl font-bold bg-amber-500 text-white shadow-md' : 'flex-1 py-3 rounded-2xl font-bold bg-white text-amber-500 border border-amber-100';
            document.getElementById('btnHistory').className = !isWrite ? 'flex-1 py-3 rounded-2xl font-bold bg-amber-500 text-white shadow-md' : 'flex-1 py-3 rounded-2xl font-bold bg-white text-amber-500 border border-amber-100';
            
            if (!isWrite) loadLogs();
        }

        // 儲存日記
        function handleSave() {
            const content = document.getElementById('diaryInput').value.trim();
            if (!content) return alert("請輸入內容喔！");

            const btn = document.getElementById('saveBtn');
            btn.disabled = true;
            btn.textContent = "正在同步至雲端...";

            // 呼叫 GAS 後端 Code.gs 中的 saveDiary 函數
            google.script.run.withSuccessHandler(() => {
                stats.exp += 1;
                if (stats.exp >= 10) {
                    stats.exp = 0;
                    stats.lv += 1;
                }
                updateUI();
                document.getElementById('diaryInput').value = '';
                btn.disabled = false;
                btn.textContent = "同步紀錄並餵食";
                alert("已成功寫入試算表！小雞長大了一點。");
                switchTab('history');
            }).saveDiary(content, stats.lv);
        }

        // 讀取紀錄
        function loadLogs() {
            const list = document.getElementById('logList');
            google.script.run.withSuccessHandler(data => {
                if (data.length === 0) {
                    list.innerHTML = '<p class="text-center text-gray-400 py-10">尚無紀錄，開始寫日記吧！</p>';
                    return;
                }
                list.innerHTML = data.map(item => `
                    <div class="bg-white/70 p-4 rounded-2xl border border-white shadow-sm text-sm">
                        <div class="text-amber-600 font-bold mb-1 text-xs">${item.time}</div>
                        <div class="text-gray-700 leading-relaxed">${item.content}</div>
                    </div>
                `).join('');
            }).getLogs();
        }

        // 初始化
        updateUI();
    </script>
</body>
</html>
