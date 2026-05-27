<!DOCTYPE html>
<html lang="ku" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>یاری کۆنکەنی ئۆنلاین پێشکەوتوو</title>
    <style>
        :root {
            --felt-green: #14532d;
            --gold: #f39c12;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #061f0d;
            background-image: radial-gradient(#166534, #062312);
            color: white;
            text-align: center;
            margin: 0;
            padding: 15px;
            user-select: none;
        }
        .game-board {
            max-width: 1000px;
            margin: 15px auto;
            background: rgba(0, 0, 0, 0.2);
            border: 5px solid #451a03;
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.7), inset 0 0 100px rgba(0,0,0,0.5);
        }
        .table-center {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 60px;
            margin: 25px 0;
            background: rgba(0,0,0,0.15);
            padding: 20px;
            border-radius: 15px;
        }
        .zone {
            display: flex;
            flex-direction: column;
            align-items: center;
            font-size: 0.9rem;
            color: #ddd;
        }
        .card {
            width: 80px;
            height: 120px;
            background: white;
            color: #111;
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 8px;
            font-weight: bold;
            font-size: 1.3rem;
            box-shadow: 0 5px 10px rgba(0,0,0,0.4);
            cursor: grab;
            transition: transform 0.1s;
            box-sizing: border-box;
            border: 1px solid #bbb;
        }
        .card:active { cursor: grabbing; }
        .card.back {
            background: linear-gradient(135px, #991b1b 25%, #7f1d1d 25%, #7f1d1d 50%, #991b1b 50%, #991b1b 75%, #7f1d1d 75%, #7f1d1d 100%);
            background-size: 15px 15px;
            border: 2px solid white;
            cursor: pointer;
        }
        .card .suit-bottom { align-self: flex-end; transform: rotate(180deg); }
        .red { color: #e11d48; }
        
        .discard-drop-zone, .meld-drop-zone {
            width: 90px;
            height: 130px;
            border: 2px dashed rgba(255,255,255,0.4);
            border-radius: 10px;
            display: flex;
            justify-content: center;
            align-items: center;
            background: rgba(0,0,0,0.2);
            transition: 0.2s;
        }
        .meld-drop-zone {
            width: auto;
            min-width: 100px;
            max-width: 200px;
            height: 130px;
            padding: 5px;
            display: flex;
            flex-direction: row;
        }
        .meld-area-container {
            display: flex;
            justify-content: center;
            gap: 15px;
            background: rgba(0, 0, 0, 0.3);
            padding: 15px;
            border-radius: 12px;
            margin: 20px 0;
            border: 1px solid rgba(255,255,255,0.1);
        }
        .drag-over {
            border-color: var(--gold) !important;
            background: rgba(243, 156, 18, 0.2) !important;
        }
        .player-area {
            margin-top: 25px;
            background: rgba(0,0,0,0.25);
            padding: 20px;
            border-radius: 15px;
        }
        .hand-drop-zone {
            display: flex;
            justify-content: center;
            gap: 10px;
            min-height: 140px;
            flex-wrap: wrap;
            padding: 15px;
            border: 2px dashed transparent;
        }
        button {
            background: linear-gradient(to bottom, #f39c12, #d35400);
            border: none;
            color: white;
            padding: 10px 22px;
            font-size: 1rem;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
        }
        .status-bar { font-size: 1.2rem; color: #f1c40f; margin-bottom: 20px; font-weight: bold; }
        .setup-screen { padding: 50px; background: rgba(0,0,0,0.5); border-radius: 15px; margin: 50px auto; max-width: 400px; }
        input { padding: 10px; font-size: 1rem; border-radius: 5px; border: none; margin-left: 10px; width: 150px; text-align: center; }
    </style>
    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
</head>
<body>

    <!-- شاشەی سەرەتا -->
    <div id="setup-screen" class="setup-screen">
        <h2>یاری کۆنکەنی ئۆنلاین</h2>
        <p>کۆدی ژوور بنووسە:</p>
        <input type="text" id="game-id" value="room555">
        <br><br>
        <button onclick="startAsHost()">دروستکردنی ژوور (P1)</button>
        <br><br>
        <button onclick="joinAsGuest()">چوونە ناو ژوور (P2)</button>
    </div>

    <!-- شاشەی یاری -->
    <div id="game-screen" class="game-board" style="display: none;">
        <h2 id="player-title">یاری کۆنکەن</h2>
        <div id="status" class="status-bar">چاوەڕوانی یاریزانی بەرامبەر...</div>

        <div class="table-center">
            <div class="zone">
                <span>دەستەی سەر زەوی (کلیک بۆ ڕاکێشان)</span>
                <div id="main-deck" class="card back" onclick="handleDeckClick()"></div>
                <span id="deck-count">52 کارت</span>
            </div>
            
            <div class="zone">
                <span>کارتە فڕێدراوەکان (کلیک بۆ هەڵگرتنەوە)</span>
                <div id="discard-zone" class="discard-drop-zone" onclick="handleDiscardClick()">
                    <div id="empty-discard-text" style="color: #aaa;">فڕێدان</div>
                </div>
            </div>
        </div>

        <!-- ناوچەی داگرتنی کارتەکان -->
        <div style="text-align: right; padding-right: 20px;">🔻 کارتەکان لێرە دابەزێنە (بە دەست ڕایبکێشە بۆ سەرەوە بۆ گەڕاندنەوەیان):</div>
        <div class="meld-area-container">
            <div id="meld-zone-1" class="meld-drop-zone" data-zone="1">کۆمەڵەی ١</div>
            <div id="meld-zone-2" class="meld-drop-zone" data-zone="2">کۆمەڵەی ٢</div>
            <div id="meld-zone-3" class="meld-drop-zone" data-zone="3">کۆمەڵەی ٣</div>
            <div id="meld-zone-4" class="meld-drop-zone" data-zone="4">کۆمەڵەی ٤</div>
        </div>
        <div style="margin-bottom: 15px;">خاڵە دابەزیوەکانی تۆ: <span id="current-points" style="color: #f1c40f;">0</span> / 51</div>

        <div class="player-area">
            <h3>کارتەکانی دەستی تۆ (<span id="card-count">0</span> کارت)</h3>
            <div id="player-hand" class="hand-drop-zone"></div>
            <div class="controls">
                <button onclick="sortHand()">ڕێکخستنی خودکار</button>
            </div>
        </div>
    </div>

    <script>
        const suits = ['♠', '♥', '♦', '♣'];
        const values = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
        const cardValues = { 'A': 11, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9, '10': 10, 'J': 10, 'Q': 10, 'K': 10 };

        let peer, conn, isHost = false, myRole = "";
        let dragSource = null; 
        let dragIndex = null;
        let dragZoneId = null;
        
        let gameState = {
            deck: [], discardPile: [],
            p1Hand: [], p2Hand: [],
            p1Melds: { 1: [], 2: [], 3: [], 4: [] },
            p2Melds: { 1: [], 2: [], 3: [], 4: [] },
            turn: "P1", hasDrawn: false
        };

        function startAsHost() {
            const roomId = document.getElementById('game-id').value;
            myRole = "P1"; isHost = true;
            peer = new Peer('konkan-pro-' + roomId);
            peer.on('open', () => {
                document.getElementById('setup-screen').style.display = 'none';
                document.getElementById('game-screen').style.display = 'block';
                document.getElementById('player-title').innerText = "تۆ یاریزانی ١ کانی";
                initDeck();
            });
            peer.on('connection', (c) => { conn = c; setupConnection(); sendState(); renderBoard(); });
        }

        function joinAsGuest() {
            const roomId = document.getElementById('game-id').value;
            myRole = "P2"; isHost = false;
            peer = new Peer();
            peer.on('open', () => {
                conn = peer.connect('konkan-pro-' + roomId);
                setupConnection();
                document.getElementById('setup-screen').style.display = 'none';
                document.getElementById('game-screen').style.display = 'block';
                document.getElementById('player-title').innerText = "تۆ یاریزانی ٢ یت";
            });
        }

        function setupConnection() {
            conn.on('data', (data) => { gameState = data; renderBoard(); });
            setupDragAndDrop();
        }

        function sendState() { if (conn && conn.open) conn.send(gameState); }

        function initDeck() {
            gameState.deck = [];
            for (let suit of suits) { for (let value of values) { gameState.deck.push({ suit, value }); } }
            for (let i = gameState.deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [gameState.deck[i], gameState.deck[j]] = [gameState.deck[j], gameState.deck[i]];
            }
            gameState.p1Hand = gameState.deck.splice(0, 10);
            gameState.p2Hand = gameState.deck.splice(0, 10);
        }

        // ڕاکێشان لە دەستەی سەرەکی
        function handleDeckClick() {
            if (gameState.turn !== myRole || gameState.hasDrawn) return;
            const card = gameState.deck.pop();
            if (myRole === "P1") gameState.p1Hand.push(card); else gameState.p2Hand.push(card);
            gameState.hasDrawn = true; sendState(); renderBoard();
        }

        // هەڵگرتنەوەی کارتی فڕێدراوی سەر زەوی
        function handleDiscardClick() {
            if (gameState.turn !== myRole || gameState.hasDrawn) return;
            if (gameState.discardPile.length === 0) return;

            // هەڵگرتنی دوایین کارتی سەر زەوی
            const card = gameState.discardPile.pop();
            if (myRole === "P1") gameState.p1Hand.push(card); else gameState.p2Hand.push(card);
            
            gameState.hasDrawn = true; 
            sendState(); 
            renderBoard();
        }

        function sortHand() {
            let myHand = myRole === "P1" ? gameState.p1Hand : gameState.p2Hand;
            myHand.sort((a, b) => {
                if (a.suit !== b.suit) return suits.indexOf(a.suit) - suits.indexOf(b.suit);
                return values.indexOf(a.value) - values.indexOf(b.value);
            });
            renderBoard();
        }

        function calculateMeldPoints() {
            let myMelds = myRole === "P1" ? gameState.p1Melds : gameState.p2Melds;
            let total = 0;
            for (let zone in myMelds) {
                myMelds[zone].forEach(card => { total += cardValues[card.value]; });
            }
            return total;
        }

        function createCardElement(card, index, source, zoneId = null) {
            const cardEl = document.createElement('div');
            cardEl.className = 'card';
            cardEl.draggable = true;
            
            if (card.suit === '♥' || card.suit === '♦') cardEl.classList.add('red');

            cardEl.innerHTML = `
                <div>${card.value}</div>
                <div style="font-size: 1.8rem; align-self: center;">${card.suit}</div>
                <div class="suit-bottom">${card.value}</div>
            `;

            cardEl.addEventListener('dragstart', (e) => {
                if (gameState.turn !== myRole) { e.preventDefault(); return; }
                dragSource = source;
                dragIndex = index;
                dragZoneId = zoneId;
            });

            return cardEl;
        }

        function setupDragAndDrop() {
            const discardZone = document.getElementById('discard-zone');
            const handZone = document.getElementById('player-hand');

            handZone.addEventListener('dragover', (e) => e.preventDefault());
            handZone.addEventListener('drop', (e) => {
                e.preventDefault();
                let myHand = myRole === "P1" ? gameState.p1Hand : gameState.p2Hand;
                let myMelds = myRole === "P1" ? gameState.p1Melds : gameState.p2Melds;

                if (dragSource === "meld" && dragZoneId !== null && dragIndex !== null) {
                    const card = myMelds[dragZoneId].splice(dragIndex, 1)[0];
                    myHand.push(card);
                    resetDrag(); sendState(); renderBoard();
                } 
                else if (dragSource === "hand" && dragIndex !== null) {
                    const targetCard = e.target.closest('.card');
                    if (targetCard) {
                        const targetIndex = parseInt(targetCard.dataset.index);
                        const movedCard = myHand.splice(dragIndex, 1)[0];
                        myHand.splice(targetIndex, 0, movedCard);
                        resetDrag(); renderBoard();
                    }
                }
            });

            // فڕێدانی کارت بۆ سەر زەوی
            discardZone.addEventListener('dragover', (e) => e.preventDefault());
            discardZone.addEventListener('drop', (e) => {
                e.preventDefault();
                if (dragSource === "hand" && dragIndex !== null && gameState.turn === myRole && gameState.hasDrawn) {
                    let myHand = myRole === "P1" ? gameState.p1Hand : gameState.p2Hand;
                    
                    let currentPoints = calculateMeldPoints();
                    let droppedAny = Object.values(myRole === "P1" ? gameState.p1Melds : gameState.p2Melds).some(z => z.length > 0);
                    if (droppedAny && currentPoints < 51) {
                        alert("ناتوانیت کارت فڕێ بدەیت! خاڵەکانت کەمترە لە ٥١ خاڵ. کارتەکان بگەڕێنەوە دەستت یان پێڕی زیاتر دابەزێنە.");
                        return;
                    }

                    const card = myHand.splice(dragIndex, 1)[0];
                    gameState.discardPile.push(card);
                    gameState.turn = myRole === "P1" ? "P2" : "P1";
                    gameState.hasDrawn = false; resetDrag();
                    sendState(); renderBoard();
                }
            });

            document.querySelectorAll('.meld-drop-zone').forEach(zoneEl => {
                zoneEl.addEventListener('dragover', (e) => e.preventDefault());
                zoneEl.addEventListener('drop', (e) => {
                    e.preventDefault();
                    let myHand = myRole === "P1" ? gameState.p1Hand : gameState.p2Hand;
                    let myMelds = myRole === "P1" ? gameState.p1Melds : gameState.p2Melds;
                    const targetZoneId = zoneEl.dataset.zone;

                    if (dragSource === "hand" && dragIndex !== null) {
                        const card = myHand.splice(dragIndex, 1)[0];
                        myMelds[targetZoneId].push(card);
                        resetDrag(); sendState(); renderBoard();
                    }
                });
            });
        }

        function resetDrag() {
            dragSource = null; dragIndex = null; dragZoneId = null;
        }

        function renderBoard() {
            const myHand = myRole === "P1" ? gameState.p1Hand : gameState.p2Hand;
            const handDiv = document.getElementById('player-hand');
            handDiv.innerHTML = '';
            myHand.forEach((card, index) => { 
                let cEl = createCardElement(card, index, "hand");
                cEl.dataset.index = index;
                handDiv.appendChild(cEl); 
            });

            document.getElementById('card-count').innerText = myHand.length;
            document.getElementById('deck-count').innerText = `${gameState.deck.length} کارت ماوە`;

            const discardZone = document.getElementById('discard-zone');
            const emptyText = document.getElementById('empty-discard-text');
            const oldCard = discardZone.querySelector('.card');
            if (oldCard) oldCard.remove();

            if (gameState.discardPile.length > 0) {
                emptyText.style.display = 'none';
                const topCard = gameState.discardPile[gameState.discardPile.length - 1];
                let cEl = createCardElement(topCard, null, "discard");
                // کارتی سەر زەوی ئێستا دەکرێت کلیکی لێبکرێت بۆ هەڵگرتنەوە
                cEl.style.cursor = (gameState.turn === myRole && !gameState.hasDrawn) ? "pointer" : "not-allowed";
                discardZone.appendChild(cEl);
            } else { emptyText.style.display = 'block'; }

            let myMelds = myRole === "P1" ? gameState.p1Melds : gameState.p2Melds;
            for (let i = 1; i <= 4; i++) {
                const zoneEl = document.getElementById(`meld-zone-${i}`);
                zoneEl.innerHTML = '';
                if (myMelds[i].length > 0) {
                    myMelds[i].forEach((card, index) => {
                        let cEl = createCardElement(card, index, "meld", i);
                        cEl.style.width = "65px"; cEl.style.height = "95px"; cEl.style.fontSize = "1rem";
                        zoneEl.appendChild(cEl);
                    });
                } else { zoneEl.innerText = `کۆمەڵەی ${i}`; }
            }

            let totalPoints = calculateMeldPoints();
            document.getElementById('current-points').innerText = totalPoints;
            document.getElementById('current-points').style.color = totalPoints >= 51 ? "#22c55e" : "#f1c40f";

            if (gameState.turn === myRole) {
                if (!gameState.hasDrawn) {
                    document.getElementById('status').innerText = "نۆبەتی تۆیە! کارتێک لە دەستەی سەرەکی یان کارتی فڕێدراوی سەر زەوی ڕابکێشە.";
                } else {
                    document.getElementById('status').innerText = "تۆ کارتت ڕاکێشا، ئێستا پێڕ دابەزێنە یان کارتێک فڕێ بدە.";
                }
            } else {
                document.getElementById('status').innerText = "چاوەڕوان بە، نۆبەتی یاریزانی بەرامبەرە...";
            }
        }
    </script>
</body>
</html>
