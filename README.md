<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Игра: Эмоции и Цвета на китайском</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
        }
        h1 {
            color: #333;
            margin-bottom: 5px;
        }
        p {
            color: #666;
            margin-bottom: 20px;
            text-align: center;
        }
        .game-board {
            display: flex;
            gap: 40px;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.1);
        }
        .column {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }
        .card {
            width: 180px;
            height: 70px;
            display: flex;
            justify-content: center;
            align-items: center;
            border: 2px solid #ccc;
            border-radius: 10px;
            cursor: pointer;
            background-color: #fff;
            transition: all 0.3s ease;
            user-select: none;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        .card:hover {
            transform: scale(1.05);
            border-color: #888;
        }
        .card.word {
            flex-direction: column;
        }
        .card .chinese {
            font-size: 22px;
            font-weight: bold;
            color: #2c3e50;
        }
        .card .pinyin {
            font-size: 14px;
            color: #7f8c8d;
        }
        .card.emoji {
            font-size: 40px;
        }
        /* Состояния карточек */
        .card.selected {
            border-color: #3498db;
            background-color: #e3f2fd;
            transform: scale(1.05);
        }
        .card.correct {
            border-color: #2ecc71;
            background-color: #d4edda;
            opacity: 0.3;
            pointer-events: none;
            transform: scale(0.9);
        }
        .card.wrong {
            border-color: #e74c3c;
            background-color: #f8d7da;
            animation: shake 0.4s;
        }
        @keyframes shake {
            0% { transform: translateX(0); }
            25% { transform: translateX(-5px); }
            50% { transform: translateX(5px); }
            75% { transform: translateX(-5px); }
            100% { transform: translateX(0); }
        }
        #message {
            margin-top: 20px;
            font-size: 24px;
            font-weight: bold;
            color: #27ae60;
            display: none;
        }
        .reset-btn {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 5px;
        }
        .reset-btn:hover {
            background-color: #2980b9;
        }
    </style>
</head>
<body>

    <h1>找一找 (Найди пару)</h1>
    <p>Соедини китайское слово с правильной эмоцией или цветом</p>

    <div class="game-board">
        <!-- Левая колонка: слова -->
        <div class="column" id="words-column">
            <!-- Эмоции -->
            <div class="card word" data-id="happy1">
                <div class="chinese">开心</div>
                <div class="pinyin">kāixīn (радостный)</div>
            </div>
            <div class="card word" data-id="happy2">
                <div class="chinese">高兴</div>
                <div class="pinyin">gāoxìng (счастливый)</div>
            </div>
            <div class="card word" data-id="sad">
                <div class="chinese">难过</div>
                <div class="pinyin">nánguò (грустный)</div>
            </div>
            <div class="card word" data-id="angry">
                <div class="chinese">生气</div>
                <div class="pinyin">shēngqì (злой)</div>
            </div>
            <!-- Цвета -->
            <div class="card word" data-id="red">
                <div class="chinese">红色</div>
                <div class="pinyin">hóngsè (красный)</div>
            </div>
            <div class="card word" data-id="blue">
                <div class="chinese">蓝色</div>
                <div class="pinyin">lánsè (синий)</div>
            </div>
            <div class="card word" data-id="yellow">
                <div class="chinese">黄色</div>
                <div class="pinyin">huángsè (желтый)</div>
            </div>
            <div class="card word" data-id="green">
                <div class="chinese">绿色</div>
                <div class="pinyin">lǜsè (зеленый)</div>
            </div>
        </div>

        <!-- Правая колонка: картинки (эмодзи) -->
        <div class="column" id="emojis-column">
            <!-- Порядок перемешан для интереса -->
            <div class="card emoji" data-id="blue">🟦</div>
            <div class="card emoji" data-id="happy1">😁</div> <!-- Для 开心 -->
            <div class="card emoji" data-id="red">🟥</div>
            <div class="card emoji" data-id="sad">😢</div>
            <div class="card emoji" data-id="green">🟩</div>
            <div class="card emoji" data-id="happy2">😊</div> <!-- Для 高兴 -->
            <div class="card emoji" data-id="yellow">🟨</div>
            <div class="card emoji" data-id="angry">😡</div>
        </div>
    </div>

    <div id="message">太棒了！你全对了！🎉 (Отлично! Все верно!)</div>
    <button class="reset-btn" onclick="resetGame()">玩一次 (Играть снова)</button>

    <script>
        const words = document.querySelectorAll('.card.word');
        const emojis = document.querySelectorAll('.card.emoji');
        const message = document.getElementById('message');
        
        let selectedWord = null;
        let selectedEmoji = null;
        let matchedPairs = 0;
        // Общее количество пар: 4 эмоции + 4 цвета = 8
        const totalPairs = 8;

        words.forEach(word => {
            word.addEventListener('click', () => selectCard(word, 'word'));
        });

        emojis.forEach(emoji => {
            emoji.addEventListener('click', () => selectCard(emoji, 'emoji'));
        });

        function selectCard(card, type) {
            if (card.classList.contains('correct') || card.classList.contains('selected')) return;

            if (type === 'word' && selectedWord) {
                selectedWord.classList.remove('selected');
                selectedWord = null;
            }
            if (type === 'emoji' && selectedEmoji) {
                selectedEmoji.classList.remove('selected');
                selectedEmoji = null;
            }

            card.classList.add('selected');

            if (type === 'word') {
                selectedWord = card;
            } else {
                selectedEmoji = card;
            }

            checkMatch();
        }

        function checkMatch() {
            if (selectedWord && selectedEmoji) {
                const wordId = selectedWord.getAttribute('data-id');
                const emojiId = selectedEmoji.getAttribute('data-id');

                if (wordId === emojiId) {
                    selectedWord.classList.remove('selected');
                    selectedEmoji.classList.remove('selected');
                    selectedWord.classList.add('correct');
                    selectedEmoji.classList.add('correct');
                    matchedPairs++;

                    selectedWord = null;
                    selectedEmoji = null;

                    if (matchedPairs === totalPairs) {
                        message.style.display = 'block';
                    }
                } else {
                    selectedWord.classList.add('wrong');
                    selectedEmoji.classList.add('wrong');

                    setTimeout(() => {
                        selectedWord.classList.remove('selected', 'wrong');
                        selectedEmoji.classList.remove('selected', 'wrong');
                        selectedWord = null;
                        selectedEmoji = null;
                    }, 600);
                }
            }
        }

        function resetGame() {
            selectedWord = null;
            selectedEmoji = null;
            matchedPairs = 0;
            message.style.display = 'none';

            const allCards = document.querySelectorAll('.card');
            allCards.forEach(card => {
                card.classList.remove('selected', 'correct', 'wrong');
            });
            
            shuffleColumn(document.getElementById('words-column'));
            shuffleColumn(document.getElementById('emojis-column'));
        }

        function shuffleColumn(column) {
            const cards = Array.from(column.children);
            for (let i = cards.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                column.appendChild(cards[j]);
                column.appendChild(cards[i]);
            }
        }
    </script>
</body>
</html>
