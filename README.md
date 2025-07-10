<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>词语搭配连连看 - 小朋友学习游戏</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#4F46E5',
                        secondary: '#F97316',
                        accent: '#10B981',
                        neutral: '#F3F4F6',
                        dark: '#1F2937',
                    },
                    fontFamily: {
                        comic: ['"Comic Sans MS"', '"Marker Felt"', 'Arial', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .text-shadow {
                text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.25);
            }
            .animate-bounce-slow {
                animation: bounce 3s infinite;
            }
            .card-hover {
                transition: all 0.3s ease;
            }
            .card-hover:hover {
                transform: translateY(-5px);
                box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            }
            .dragging {
                opacity: 0.5;
                transform: scale(1.05);
            }
            .success {
                animation: success 0.5s ease-in-out;
            }
            .error {
                animation: error 0.5s ease-in-out;
            }
            @keyframes success {
                0%, 100% { transform: scale(1); }
                50% { transform: scale(1.1); background-color: #ECFDF5; }
            }
            @keyframes error {
                0%, 100% { transform: translateX(0); }
                25% { transform: translateX(-5px); }
                75% { transform: translateX(5px); }
            }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-purple-50 min-h-screen font-comic text-dark">
    <!-- 顶部导航 -->
    <header class="bg-primary text-white shadow-lg">
        <div class="container mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa fa-puzzle-piece text-2xl"></i>
                <h1 class="text-xl md:text-2xl font-bold text-shadow">词语搭配连连看</h1>
            </div>
            <div class="hidden md:flex items-center">
                <span id="score" class="bg-white text-primary px-3 py-1 rounded-full font-bold">得分: 0</span>
            </div>
        </div>
    </header>

    <!-- 移动端计分板 -->
    <div class="md:hidden container mx-auto px-4 py-2 flex justify-end bg-white shadow-md rounded-b-lg">
        <span id="mobile-score" class="text-primary font-bold">得分: 0</span>
    </div>

    <!-- 游戏区域 -->
    <main class="container mx-auto px-4 py-8">
        <!-- 游戏介绍 -->
        <div id="intro" class="max-w-3xl mx-auto text-center mb-8 bg-white rounded-xl shadow-lg p-6 transform transition-all duration-500">
            <h2 class="text-2xl md:text-3xl font-bold text-primary mb-4">欢迎来到词语搭配游戏!</h2>
            <p class="text-lg mb-6">把左边的动词和右边合适的名词连起来，看看你能匹配多少正确的词语对!</p>
            <button id="start-game" class="bg-secondary hover:bg-orange-600 text-white font-bold py-3 px-8 rounded-full text-lg shadow-lg transform transition-all duration-300 hover:scale-105 focus:outline-none focus:ring-2 focus:ring-orange-400 focus:ring-opacity-50">
                <i class="fa fa-play mr-2"></i>开始游戏
            </button>
        </div>

        <!-- 游戏界面 (默认隐藏) -->
        <div id="game-container" class="hidden">
            <!-- 游戏提示 -->
            <div class="bg-white rounded-xl shadow-lg p-4 mb-6 max-w-3xl mx-auto">
                <div class="flex items-center">
                    <i class="fa fa-lightbulb-o text-secondary text-2xl mr-3"></i>
                    <p class="font-medium">提示: 拖动左边的动词卡片，与右边合适的名词卡片配对!</p>
                </div>
            </div>

            <!-- 游戏区域 -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-8 max-w-5xl mx-auto">
                <!-- 动词卡片区 -->
                <div id="verbs-container" class="bg-blue-50 rounded-xl shadow-lg p-6 min-h-[400px]">
                    <h3 class="text-xl font-bold text-primary mb-4 flex items-center">
                        <i class="fa fa-hand-pointer-o mr-2"></i>动词卡片
                    </h3>
                    <div id="verbs" class="grid grid-cols-2 sm:grid-cols-3 gap-4">
                        <!-- 动词卡片将通过JS动态生成 -->
                    </div>
                </div>

                <!-- 名词卡片区 -->
                <div id="nouns-container" class="bg-green-50 rounded-xl shadow-lg p-6 min-h-[400px]">
                    <h3 class="text-xl font-bold text-accent mb-4 flex items-center">
                        <i class="fa fa-cubes mr-2"></i>名词卡片
                    </h3>
                    <div id="nouns" class="grid grid-cols-2 sm:grid-cols-3 gap-4">
                        <!-- 名词卡片将通过JS动态生成 -->
                    </div>
                </div>
            </div>

            <!-- 已匹配区域 -->
            <div id="matched-container" class="mt-8 bg-white rounded-xl shadow-lg p-6 max-w-5xl mx-auto hidden">
                <h3 class="text-xl font-bold text-purple-600 mb-4 flex items-center">
                    <i class="fa fa-check-circle mr-2"></i>已匹配的词语对
                </h3>
                <div id="matched-pairs" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
                    <!-- 已匹配的词语对将通过JS动态生成 -->
                </div>
            </div>
        </div>

        <!-- 游戏结束 (默认隐藏) -->
        <div id="game-over" class="hidden max-w-3xl mx-auto text-center mt-12 bg-white rounded-xl shadow-lg p-8 transform transition-all duration-500">
            <div class="mb-4 animate-bounce-slow">
                <i class="fa fa-trophy text-5xl text-yellow-500"></i>
            </div>
            <h2 class="text-2xl md:text-3xl font-bold text-primary mb-4">游戏结束!</h2>
            <p class="text-xl mb-2">你的得分: <span id="final-score" class="text-secondary font-bold">0</span></p>
            <p class="text-lg mb-6">你成功匹配了 <span id="pairs-count" class="text-accent font-bold">0</span> 对词语!</p>
            <div class="flex flex-col sm:flex-row justify-center gap-4">
                <button id="play-again" class="bg-secondary hover:bg-orange-600 text-white font-bold py-3 px-6 rounded-full text-lg shadow-lg transform transition-all duration-300 hover:scale-105 focus:outline-none focus:ring-2 focus:ring-orange-400 focus:ring-opacity-50">
                    <i class="fa fa-refresh mr-2"></i>再玩一次
                </button>
                <button id="share-game" class="bg-primary hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-full text-lg shadow-lg transform transition-all duration-300 hover:scale-105 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-50">
                    <i class="fa fa-share-alt mr-2"></i>分享游戏
                </button>
            </div>
        </div>
    </main>

    <!-- 页脚 -->
    <footer class="bg-dark text-white mt-12 py-4">
        <div class="container mx-auto px-4 text-center">
            <p>词语搭配连连看 - 让学习变得有趣!</p>
            <p class="text-sm mt-2">© 2025 教育游戏工作室</p>
        </div>
    </footer>

    <!-- 分享弹窗 (默认隐藏) -->
    <div id="share-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
        <div class="bg-white rounded-xl shadow-2xl p-6 max-w-md w-full mx-4 transform transition-all duration-300 scale-95 opacity-0" id="modal-content">
            <div class="text-right mb-4">
                <button id="close-modal" class="text-gray-500 hover:text-gray-700 focus:outline-none">
                    <i class="fa fa-times text-xl"></i>
                </button>
            </div>
            <h3 class="text-xl font-bold text-primary mb-4">分享游戏</h3>
            <p class="mb-4">复制下面的链接，发送给你的小伙伴一起玩!</p>
            <div class="flex mb-4">
                <input type="text" id="share-url" value="https://example.com/word-game" class="flex-1 px-4 py-2 border border-gray-300 rounded-l-lg focus:outline-none focus:ring-2 focus:ring-primary" readonly>
                <button id="copy-url" class="bg-primary hover:bg-indigo-700 text-white px-4 py-2 rounded-r-lg focus:outline-none">
                    <i class="fa fa-copy"></i>
                </button>
            </div>
            <div class="flex justify-center space-x-4">
                <a href="#" class="bg-blue-500 hover:bg-blue-600 text-white p-3 rounded-full w-12 h-12 flex items-center justify-center focus:outline-none">
                    <i class="fa fa-weixin text-xl"></i>
                </a>
                <a href="#" class="bg-green-500 hover:bg-green-600 text-white p-3 rounded-full w-12 h-12 flex items-center justify-center focus:outline-none">
                    <i class="fa fa-qq text-xl"></i>
                </a>
                <a href="#" class="bg-red-500 hover:bg-red-600 text-white p-3 rounded-full w-12 h-12 flex items-center justify-center focus:outline-none">
                    <i class="fa fa-weibo text-xl"></i>
                </a>
            </div>
        </div>
    </div>

    <script>
        // 游戏数据
        const wordPairs = [
            { verb: "爬", noun: "小山" },
            { verb: "玩", noun: "滑梯" },
            { verb: "坐", noun: "飞机" },
            { verb: "采", noun: "花蜜" },
            { verb: "搬", noun: "小虫" },
            { verb: "搭", noun: "积木" },
            { verb: "吃", noun: "晚饭" },
            { verb: "读", noun: "故事" },
            { verb: "做", noun: "早操" }
        ];

        // DOM元素
        const intro = document.getElementById('intro');
        const gameContainer = document.getElementById('game-container');
        const gameOver = document.getElementById('game-over');
        const verbsContainer = document.getElementById('verbs');
        const nounsContainer = document.getElementById('nouns');
        const matchedPairsContainer = document.getElementById('matched-pairs');
        const scoreElement = document.getElementById('score');
        const mobileScoreElement = document.getElementById('mobile-score');
        const finalScoreElement = document.getElementById('final-score');
        const pairsCountElement = document.getElementById('pairs-count');
        const startGameButton = document.getElementById('start-game');
        const playAgainButton = document.getElementById('play-again');
        const shareGameButton = document.getElementById('share-game');
        const shareModal = document.getElementById('share-modal');
        const modalContent = document.getElementById('modal-content');
        const closeModalButton = document.getElementById('close-modal');
        const copyUrlButton = document.getElementById('copy-url');
        const shareUrlInput = document.getElementById('share-url');
        const matchedContainer = document.getElementById('matched-container');

        // 游戏状态
        let gameState = {
            score: 0,
            matchedPairs: [],
            currentDragging: null,
            pairs: [...wordPairs] // 复制原始数据
        };

        // 更新游戏分数
        function updateScore(points) {
            gameState.score += points;
            scoreElement.textContent = `得分: ${gameState.score}`;
            mobileScoreElement.textContent = `得分: ${gameState.score}`;
        }

        // 开始游戏
        function startGame() {
            // 重置游戏状态
            gameState = {
                score: 0,
                matchedPairs: [],
                currentDragging: null,
                pairs: shuffleArray([...wordPairs])
            };

            // 更新UI
            intro.classList.add('hidden');
            gameContainer.classList.remove('hidden');
            gameOver.classList.add('hidden');
            matchedContainer.classList.add('hidden');
            updateScore(0);

            // 清空容器
            verbsContainer.innerHTML = '';
            nounsContainer.innerHTML = '';
            matchedPairsContainer.innerHTML = '';

            // 创建动词卡片
            gameState.pairs.forEach(({ verb }) => {
                const card = document.createElement('div');
                card.className = 'bg-white border-2 border-blue-300 rounded-xl p-4 text-center card-hover cursor-move';
                card.setAttribute('draggable', 'true');
                card.setAttribute('data-verb', verb);
                card.innerHTML = `
                    <div class="text-3xl font-bold mb-1">${verb}</div>
                    <div class="text-sm text-blue-600">动词</div>
                `;

                // 添加拖拽事件
                card.addEventListener('dragstart', (e) => {
                    gameState.currentDragging = verb;
                    card.classList.add('dragging');
                    e.dataTransfer.setData('text/plain', verb);
                });

                card.addEventListener('dragend', () => {
                    card.classList.remove('dragging');
                });

                verbsContainer.appendChild(card);
            });

            // 创建名词卡片
            const shuffledNouns = shuffleArray(gameState.pairs.map(pair => pair.noun));
            shuffledNouns.forEach(noun => {
                const card = document.createElement('div');
                card.className = 'bg-white border-2 border-green-300 rounded-xl p-4 text-center card-hover cursor-pointer';
                card.setAttribute('data-noun', noun);
                card.innerHTML = `
                    <div class="text-3xl font-bold">${noun}</div>
                    <div class="text-sm text-green-600">名词</div>
                `;

                // 添加拖放事件
                card.addEventListener('dragover', (e) => {
                    e.preventDefault();
                    card.classList.add('bg-green-100');
                });

                card.addEventListener('dragleave', () => {
                    card.classList.remove('bg-green-100');
                });

                card.addEventListener('drop', (e) => {
                    e.preventDefault();
                    card.classList.remove('bg-green-100');

                    if (gameState.currentDragging) {
                        checkMatch(gameState.currentDragging, noun, card);
                    }
                });

                nounsContainer.appendChild(card);
            });
        }

        // 检查匹配
        function checkMatch(verb, noun, nounCard) {
            // 查找正确的配对
            const correctPair = gameState.pairs.find(pair => pair.verb === verb && pair.noun === noun);

            if (correctPair && !gameState.matchedPairs.includes(`${verb}-${noun}`)) {
                // 匹配成功
                gameState.matchedPairs.push(`${verb}-${noun}`);
                
                // 从游戏区移除卡片
                const verbCard = document.querySelector(`[data-verb="${verb}"]`);
                
                // 添加成功动画
                verbCard.classList.add('success');
                nounCard.classList.add('success');
                
                // 延迟移除卡片
                setTimeout(() => {
                    verbCard.remove();
                    nounCard.remove();
                    
                    // 更新分数
                    updateScore(10);
                    
                    // 创建匹配成功的卡片 - 移除了动词和名词之间的空格
                    const matchedCard = document.createElement('div');
                    matchedCard.className = 'bg-white border-2 border-purple-300 rounded-lg p-3 flex items-center justify-between card-hover';
                    matchedCard.innerHTML = `
                        <div class="text-xl font-bold">${verb}${noun}</div>
                        <i class="fa fa-check-circle text-green-500 text-xl"></i>
                    `;

                    matchedPairsContainer.appendChild(matchedCard);
                    
                    // 显示已匹配区域
                    if (gameState.matchedPairs.length === 1) {
                        matchedContainer.classList.remove('hidden');
                    }
                    
                    // 检查游戏是否结束
                    if (gameState.matchedPairs.length === gameState.pairs.length) {
                        endGame();
                    }
                }, 500);
            } else {
                // 匹配失败
                updateScore(-2);
                
                // 显示错误动画
                nounCard.classList.add('error');
                setTimeout(() => {
                    nounCard.classList.remove('error');
                }, 500);
            }
        }

        // 结束游戏
        function endGame() {
            // 更新UI
            gameContainer.classList.add('hidden');
            gameOver.classList.remove('hidden');
            finalScoreElement.textContent = gameState.score;
            pairsCountElement.textContent = gameState.matchedPairs.length;
        }

        // 分享功能
        function openShareModal() {
            shareModal.classList.remove('hidden');
            setTimeout(() => {
                modalContent.classList.remove('scale-95', 'opacity-0');
                modalContent.classList.add('scale-100', 'opacity-100');
            }, 10);
            
            // 更新分享链接
            shareUrlInput.value = window.location.href;
        }

        function closeShareModal() {
            modalContent.classList.remove('scale-100', 'opacity-100');
            modalContent.classList.add('scale-95', 'opacity-0');
            setTimeout(() => {
                shareModal.classList.add('hidden');
            }, 300);
        }

        function copyUrl() {
            shareUrlInput.select();
            document.execCommand('copy');
            
            // 显示复制成功提示
            const originalText = copyUrlButton.innerHTML;
            copyUrlButton.innerHTML = '<i class="fa fa-check"></i>';
            copyUrlButton.classList.add('bg-green-500');
            
            setTimeout(() => {
                copyUrlButton.innerHTML = originalText;
                copyUrlButton.classList.remove('bg-green-500');
            }, 2000);
        }

        // 打乱数组顺序
        function shuffleArray(array) {
            const newArray = [...array];
            for (let i = newArray.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
            }
            return newArray;
        }

        // 事件监听
        startGameButton.addEventListener('click', startGame);
        playAgainButton.addEventListener('click', startGame);
        shareGameButton.addEventListener('click', openShareModal);
        closeModalButton.addEventListener('click', closeShareModal);
        copyUrlButton.addEventListener('click', copyUrl);
        shareModal.addEventListener('click', (e) => {
            if (e.target === shareModal) {
                closeShareModal();
            }
        });
    </script>
</body>
</html>
