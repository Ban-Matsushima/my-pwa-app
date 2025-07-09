# my-pwa-app
テトリス
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>テトリスPWA</title>
    <!-- Tailwind CSSを読み込む -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Interフォントを適用 */
        body {
            font-family: "Inter", sans-serif;
        }

        /* カスタムCSS */
        #gameCanvas {
            background-color: #1a202c; /* ダークな背景 */
            border: 4px solid #4a5568; /* 濃いめのボーダー */
            border-radius: 8px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3), 0 6px 6px rgba(0, 0, 0, 0.2);
            display: block; /* 中央配置のためにブロック要素にする */
            margin: 0 auto; /* 中央配置 */
        }

        #nextPieceCanvas {
            background-color: #2d3748; /* ダークな背景 */
            border: 2px solid #4a5568;
            border-radius: 4px;
            box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        .game-info-panel {
            background-color: #2d3748;
            border-radius: 8px;
            padding: 1rem;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            color: #e2e8f0;
        }

        .game-button {
            transition: all 0.2s ease-in-out;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        .game-button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
        }

        .game-button:active {
            transform: translateY(0);
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        /* モーダル */
        .modal {
            background-color: rgba(0, 0, 0, 0.7);
        }
        .modal-content {
            background-color: #2d3748;
            color: #e2e8f0;
            border-radius: 8px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
        }
    </style>
    <!-- PWA Manifestファイルをリンク -->
    <!-- このファイルはプロジェクトのルートに 'manifest.json' として保存してください -->
    <link rel="manifest" href="/manifest.json">

    <script>
        // PWAのService Workerを登録
        // ページがロードされた後にService Workerの登録を試みる
        // このファイルはプロジェクトのルートに 'sw.js' として保存してください
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('/sw.js')
                    .then(registration => {
                        console.log('Service Worker 登録成功:', registration);
                    })
                    .catch(error => {
                        console.error('Service Worker 登録失敗:', error);
                    });
            });
        }
    </script>
</head>
<body class="flex flex-col items-center justify-center min-h-screen bg-gradient-to-br from-gray-800 to-gray-900 text-white p-4">
    <div class="text-center mb-8">
        <h1 class="text-5xl font-extrabold text-blue-400 mb-2">テトリスPWA</h1>
        <p class="text-xl text-gray-300">クラシックなブロックパズルゲーム</p>
    </div>

    <div class="flex flex-col lg:flex-row items-start lg:items-center gap-8 bg-gray-700 p-6 rounded-xl shadow-2xl">
        <!-- ゲーム情報パネル -->
        <div class="game-info-panel w-full lg:w-auto flex-shrink-0">
            <h2 class="text-2xl font-bold mb-4 text-blue-300">ゲーム情報</h2>
            <div class="mb-3">
                <p class="text-lg">スコア: <span id="scoreDisplay" class="font-bold text-yellow-300">0</span></p>
            </div>
            <div class="mb-3">
                <p class="text-lg">レベル: <span id="levelDisplay" class="font-bold text-green-300">1</span></p>
            </div>
            <div class="mb-4">
                <p class="text-lg">次のブロック:</p>
                <canvas id="nextPieceCanvas" width="100" height="100" class="mt-2 mx-auto"></canvas>
            </div>
            <div class="flex flex-col gap-3">
                <button id="startButton" class="game-button px-6 py-3 bg-green-600 hover:bg-green-700 text-white font-bold rounded-lg text-lg">ゲーム開始</button>
                <button id="resetButton" class="game-button px-6 py-3 bg-red-600 hover:bg-red-700 text-white font-bold rounded-lg text-lg hidden">リセット</button>
            </div>
        </div>

        <!-- ゲームキャンバス -->
        <canvas id="gameCanvas" width="300" height="600"></canvas>
    </div>

    <!-- 操作説明 -->
    <div class="mt-8 bg-gray-700 p-6 rounded-xl shadow-2xl text-gray-300 max-w-lg w-full">
        <h2 class="text-2xl font-bold mb-4 text-blue-300 text-center">操作方法</h2>
        <ul class="list-disc list-inside text-lg text-left mx-auto max-w-xs">
            <li><span class="font-bold text-white">← / →</span>: 移動</li>
            <li><span class="font-bold text-white">↑</span>: 回転</li>
            <li><span class="font-bold text-white">↓</span>: ソフトドロップ</li>
            <li><span class="font-bold text-white">スペースキー</span>: ハードドロップ</li>
        </ul>
    </div>

    <!-- ゲームオーバー/勝利モーダル -->
    <div id="gameOverModal" class="modal fixed inset-0 flex items-center justify-center hidden z-50">
        <div class="modal-content p-8 text-center">
            <h2 id="modalTitle" class="text-3xl font-bold mb-4">ゲームオーバー！</h2>
            <p id="modalMessage" class="text-xl mb-6">スコア: <span id="finalScore" class="font-bold text-yellow-300">0</span></p>
            <button id="playAgainButton" class="game-button px-8 py-4 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-full text-xl">もう一度プレイ</button>
        </div>
    </div>

    <script>
        // JavaScriptのゲームロジック
        const COLS = 10; // 列数
        const ROWS = 20; // 行数
        const BLOCK_SIZE = 30; // 各ブロックのサイズ（ピクセル）
        const VACANT = 'white'; // 空のマス目の色 (描画しない)

        const gameCanvas = document.getElementById('gameCanvas');
        const ctx = gameCanvas.getContext('2d');
        const nextPieceCanvas = document.getElementById('nextPieceCanvas');
        const nextCtx = nextPieceCanvas.getContext('2d');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const levelDisplay = document.getElementById('levelDisplay');
        const startButton = document.getElementById('startButton');
        const resetButton = document.getElementById('resetButton');
        const gameOverModal = document.getElementById('gameOverModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const finalScoreDisplay = document.getElementById('finalScore');
        const playAgainButton = document.getElementById('playAgainButton');

        let board = []; // ゲームボードの状態
        let currentPiece; // 現在操作中のブロック
        let nextPiece; // 次のブロック
        let score = 0;
        let level = 1;
        let dropInterval; // ブロックが自動で落ちる間隔のタイマー
        let gameStarted = false;
        let gameOver = false;

        // テトリミノの形状と色
        const TETROMINOES = [
            // Z
            {
                shape: [[1, 1, 0], [0, 1, 1], [0, 0, 0]],
                color: '#ef4444' // red-500
            },
            // S
            {
                shape: [[0, 1, 1], [1, 1, 0], [0, 0, 0]],
                color: '#22c55e' // green-500
            },
            // T
            {
                shape: [[0, 1, 0], [1, 1, 1], [0, 0, 0]],
                color: '#a855f7' // purple-500
            },
            // O
            {
                shape: [[1, 1], [1, 1]],
                color: '#facc15' // yellow-500
            },
            // L
            {
                shape: [[0, 0, 1], [1, 1, 1], [0, 0, 0]],
                color: '#f97316' // orange-500
            },
            // J
            {
                shape: [[1, 0, 0], [1, 1, 1], [0, 0, 0]],
                color: '#3b82f6' // blue-500
            },
            // I
            {
                shape: [[0, 0, 0, 0], [1, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0]],
                color: '#06b6d4' // cyan-500
            }
        ];

        // ボードを初期化する関数
        function initBoard() {
            for (let r = 0; r < ROWS; r++) {
                board[r] = [];
                for (let c = 0; c < COLS; c++) {
                    board[r][c] = VACANT;
                }
            }
        }

        // 1つのブロックを描画する関数
        function drawBlock(x, y, color, context) {
            context.fillStyle = color;
            context.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
            context.strokeStyle = '#4a5568'; // ブロックの境界線
            context.strokeRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
        }

        // ボードを描画する関数
        function drawBoard() {
            for (let r = 0; r < ROWS; r++) {
                for (let c = 0; c < COLS; c++) {
                    drawBlock(c, r, board[r][c], ctx);
                }
            }
        }

        // ランダムなテトリミノを生成する関数
        function getRandomPiece() {
            const randN = Math.floor(Math.random() * TETROMINOES.length);
            const p = TETROMINOES[randN];
            return {
                shape: p.shape,
                color: p.color,
                x: Math.floor(COLS / 2) - Math.floor(p.shape[0].length / 2), // 中央に配置
                y: 0, // 上部から開始
                rotation: 0 // 初期回転状態
            };
        }

        // 現在のブロックを描画する関数
        function drawCurrentPiece() {
            for (let r = 0; r < currentPiece.shape.length; r++) {
                for (let c = 0; c < currentPiece.shape[0].length; c++) {
                    if (currentPiece.shape[r][c]) {
                        drawBlock(currentPiece.x + c, currentPiece.y + r, currentPiece.color, ctx);
                    }
                }
            }
        }

        // 次のブロックを描画する関数
        function drawNextPiece() {
            nextCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height); // キャンバスをクリア
            const nextShape = nextPiece.shape;
            const offsetX = (nextPieceCanvas.width / BLOCK_SIZE - nextShape[0].length) / 2;
            const offsetY = (nextPieceCanvas.height / BLOCK_SIZE - nextShape.length) / 2;

            for (let r = 0; r < nextShape.length; r++) {
                for (let c = 0; c < nextShape[0].length; c++) {
                    if (nextShape[r][c]) {
                        drawBlock(c + offsetX, r + offsetY, nextPiece.color, nextCtx);
                    }
                }
            }
        }

        // ブロックを消去する関数 (移動前に元の位置をクリア)
        function undrawCurrentPiece() {
            for (let r = 0; r < currentPiece.shape.length; r++) {
                for (let c = 0; c < currentPiece.shape[0].length; c++) {
                    if (currentPiece.shape[r][c]) {
                        drawBlock(currentPiece.x + c, currentPiece.y + r, VACANT, ctx);
                    }
                }
            }
        }

        // 衝突判定
        function collision(x, y, piece) {
            for (let r = 0; r < piece.length; r++) {
                for (let c = 0; c < piece[0].length; c++) {
                    if (!piece[r][c]) continue; // ブロックがない部分はスキップ

                    let newX = x + c;
                    let newY = y + r;

                    // 境界線チェックと既に配置されたブロックとの衝突チェック
                    if (newX < 0 || newX >= COLS || newY >= ROWS || (newY < 0 && piece[r][c])) {
                        return true; // 画面外または上部で衝突
                    }
                    if (newY >= 0 && board[newY][newX] !== VACANT) {
                        return true; // 既存のブロックと衝突
                    }
                }
            }
            return false;
        }

        // ブロックを下に移動
        function moveDown() {
            if (!gameStarted || gameOver) return;
            if (!collision(currentPiece.x, currentPiece.y + 1, currentPiece.shape)) {
                undrawCurrentPiece();
                currentPiece.y++;
                drawCurrentPiece();
            } else {
                // ブロックを固定
                lockPiece();
                // 行をクリア
                clearLines();
                // 新しいブロックを生成
                currentPiece = nextPiece;
                nextPiece = getRandomPiece();
                drawNextPiece();
                // ゲームオーバー判定
                if (collision(currentPiece.x, currentPiece.y, currentPiece.shape)) {
                    gameOver = true;
                    clearInterval(dropInterval);
                    drawBoard(); // 最終状態を描画
                    showModal("ゲームオーバー！", `スコア: ${score}`);
                }
            }
        }

        // ブロックを左右に移動
        function moveHorizontal(deltaX) {
            if (!gameStarted || gameOver) return;
            if (!collision(currentPiece.x + deltaX, currentPiece.y, currentPiece.shape)) {
                undrawCurrentPiece();
                currentPiece.x += deltaX;
                drawCurrentPiece();
            }
        }

        // ブロックを回転
        function rotate() {
            if (!gameStarted || gameOver) return;
            let nextRotation = (currentPiece.rotation + 1) % 4;
            let nextShape = rotateMatrix(currentPiece.shape);

            // 壁キック処理
            let kick = 0;
            if (collision(currentPiece.x, currentPiece.y, nextShape)) {
                if (currentPiece.x > COLS / 2) { // 右端に近い場合
                    kick = -1; // 左に移動を試みる
                } else { // 左端に近い場合
                    kick = 1; // 右に移動を試みる
                }
            }

            if (!collision(currentPiece.x + kick, currentPiece.y, nextShape)) {
                undrawCurrentPiece();
                currentPiece.x += kick;
                currentPiece.shape = nextShape;
                currentPiece.rotation = nextRotation;
                drawCurrentPiece();
            }
        }

        // 行列を回転させるヘルパー関数
        function rotateMatrix(matrix) {
            const N = matrix.length - 1;
            const result = matrix.map((row, i) =>
                row.map((val, j) => matrix[N - j][i])
            );
            return result;
        }

        // ブロックをボードに固定
        function lockPiece() {
            for (let r = 0; r < currentPiece.shape.length; r++) {
                for (let c = 0; c < currentPiece.shape[0].length; c++) {
                    if (currentPiece.shape[r][c]) {
                        board[currentPiece.y + r][currentPiece.x + c] = currentPiece.color;
                    }
                }
            }
        }

        // 行をクリア
        function clearLines() {
            let linesCleared = 0;
            for (let r = 0; r < ROWS; r++) {
                let isRowFull = true;
                for (let c = 0; c < COLS; c++) {
                    if (board[r][c] === VACANT) {
                        isRowFull = false;
                        break;
                    }
                }
                if (isRowFull) {
                    linesCleared++;
                    // 行を削除し、上から新しい空行を追加
                    for (let y = r; y > 0; y--) {
                        for (let c = 0; c < COLS; c++) {
                            board[y][c] = board[y - 1][c];
                        }
                    }
                    // 最上行を空にする
                    for (let c = 0; c < COLS; c++) {
                        board[0][c] = VACANT;
                    }
                }
            }
            if (linesCleared > 0) {
                score += linesCleared * 100 * level; // スコア加算
                scoreDisplay.textContent = score;
                // レベルアップ判定 (例: 10行クリアごとにレベルアップ)
                if (score >= level * 1000) { // 仮のレベルアップ条件
                    level++;
                    levelDisplay.textContent = level;
                    // ドロップ速度を速くする
                    clearInterval(dropInterval);
                    dropInterval = setInterval(moveDown, 1000 / level); // レベルに応じて加速
                }
                drawBoard(); // ボードを再描画
            }
        }

        // ゲームループ
        function drop() {
            if (!gameOver) {
                moveDown();
            }
        }

        // キーボードイベントリスナー
        document.addEventListener('keydown', e => {
            if (!gameStarted || gameOver) return;
            if (e.keyCode === 37) { // 左矢印
                moveHorizontal(-1);
            } else if (e.keyCode === 39) { // 右矢印
                moveHorizontal(1);
            } else if (e.keyCode === 38) { // 上矢印 (回転)
                rotate();
            } else if (e.keyCode === 40) { // 下矢印 (ソフトドロップ)
                moveDown();
            } else if (e.keyCode === 32) { // スペースキー (ハードドロップ)
                while (!collision(currentPiece.x, currentPiece.y + 1, currentPiece.shape)) {
                    undrawCurrentPiece();
                    currentPiece.y++;
                    drawCurrentPiece();
                }
                lockPiece();
                clearLines();
                currentPiece = nextPiece;
                nextPiece = getRandomPiece();
                drawNextPiece();
                if (collision(currentPiece.x, currentPiece.y, currentPiece.shape)) {
                    gameOver = true;
                    clearInterval(dropInterval);
                    drawBoard();
                    showModal("ゲームオーバー！", `スコア: ${score}`);
                }
            }
        });

        // モーダル表示関数
        function showModal(title, message) {
            modalTitle.textContent = title;
            finalScoreDisplay.textContent = score;
            gameOverModal.classList.remove('hidden');
        }

        // モーダル非表示関数
        function hideModal() {
            gameOverModal.classList.add('hidden');
        }

        // ゲーム開始関数
        function startGame() {
            hideModal();
            initBoard();
            score = 0;
            level = 1;
            scoreDisplay.textContent = score;
            levelDisplay.textContent = level;
            gameOver = false;
            gameStarted = true;
            startButton.textContent = '一時停止'; // ゲーム開始後は一時停止ボタンに
            resetButton.classList.remove('hidden'); // リセットボタンを表示

            currentPiece = getRandomPiece();
            nextPiece = getRandomPiece();
            drawBoard();
            drawCurrentPiece();
            drawNextPiece();

            clearInterval(dropInterval); // 既存のタイマーがあればクリア
            dropInterval = setInterval(drop, 1000); // 1秒ごとにブロックを落とす
        }

        // ゲームリセット関数
        function resetGame() {
            clearInterval(dropInterval);
            gameStarted = false;
            gameOver = false;
            initBoard();
            drawBoard();
            score = 0;
            level = 1;
            scoreDisplay.textContent = score;
            levelDisplay.textContent = level;
            startButton.textContent = 'ゲーム開始';
            resetButton.classList.add('hidden');
            ctx.clearRect(0, 0, gameCanvas.width, gameCanvas.height); // キャンバスをクリア
            nextCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height); // 次のブロック表示もクリア
            hideModal();
        }

        // イベントリスナー
        startButton.addEventListener('click', () => {
            if (!gameStarted) {
                startGame();
            } else {
                // 一時停止/再開機能 (今回は実装しないが、ボタンテキストは変更)
                // clearInterval(dropInterval);
                // gameStarted = false;
                // startButton.textContent = '再開';
            }
        });
        resetButton.addEventListener('click', resetGame);
        playAgainButton.addEventListener('click', startGame); // モーダルからの再プレイ

        // 初期描画
        initBoard();
        drawBoard(); // 空のボードを描画
        // ゲーム開始前は、次のブロック表示を空にするか、最初のブロックを表示しない
        nextCtx.clearRect(0, 0, nextPieceCanvas.width, nextPieceCanvas.height);
    </script>
</body>
</html>
