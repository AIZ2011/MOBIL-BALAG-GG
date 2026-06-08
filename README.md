<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>NIGHT FURY HD - 120FPS RACING</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            min-height: 100vh;
            background: radial-gradient(circle at 20% 30%, #0a1a1f, #020408);
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Poppins', 'Orbitron', 'Arial', sans-serif;
            padding: 16px;
            overflow: hidden;
        }

        /* Wrapper game dengan efek neon */
        .racing-container {
            background: #050b0f;
            border-radius: 48px;
            padding: 20px 24px 28px;
            box-shadow: 0 30px 50px rgba(0,0,0,0.8), inset 0 1px 3px rgba(0,255,255,0.2);
            border: 1px solid rgba(0, 255, 255, 0.4);
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 28px;
            box-shadow: 0 0 0 2px #00ffff30, 0 20px 35px black;
            cursor: none;
            background: #0d1c22;
        }

        /* Panel informasi HD */
        .hud {
            display: flex;
            justify-content: space-between;
            gap: 20px;
            margin-top: 22px;
            margin-bottom: 22px;
            flex-wrap: wrap;
        }

        .gauge {
            background: rgba(0, 20, 30, 0.75);
            backdrop-filter: blur(12px);
            border-radius: 60px;
            padding: 8px 24px;
            flex: 1;
            text-align: center;
            border: 1px solid #0ff8;
            box-shadow: 0 4px 12px rgba(0,0,0,0.5);
        }

        .gauge h3 {
            font-size: 0.9rem;
            letter-spacing: 2px;
            color: #0ff;
            text-shadow: 0 0 5px cyan;
            margin-bottom: 6px;
        }

        .stat-value {
            font-size: 2rem;
            font-weight: bold;
            font-family: 'Orbitron', monospace;
            background: linear-gradient(135deg, #ffffff, #0ff);
            background-clip: text;
            -webkit-background-clip: text;
            color: transparent;
        }

        .btn-group {
            display: flex;
            gap: 20px;
            justify-content: center;
            flex-wrap: wrap;
        }

        .btn {
            background: linear-gradient(135deg, #1f3e48, #0a1a22);
            border: none;
            font-size: 1.2rem;
            font-weight: bold;
            padding: 10px 28px;
            border-radius: 60px;
            color: #0ff;
            font-family: 'Orbitron', monospace;
            cursor: pointer;
            transition: 0.1s linear;
            box-shadow: 0 5px 0 #002d38;
            letter-spacing: 1px;
        }
        .btn:active {
            transform: translateY(3px);
            box-shadow: 0 2px 0 #002d38;
        }
        .btn-primary {
            background: linear-gradient(135deg, #f39c12, #d35400);
            color: #fff;
            box-shadow: 0 5px 0 #7e2e00;
            text-shadow: 0 1px 1px black;
        }
        .btn-reset {
            background: linear-gradient(135deg, #2c5368, #0f2027);
            color: #bbffff;
        }
        
        .speed-panel {
            background: #000000aa;
            border-radius: 28px;
            padding: 8px 16px;
            margin-top: 18px;
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            flex-wrap: wrap;
            gap: 12px;
            border-left: 6px solid #0ff;
        }
        .speed-text {
            font-size: 1.8rem;
            font-weight: bold;
            font-family: monospace;
            color: #0ff;
            text-shadow: 0 0 8px cyan;
        }
        .fps-badge {
            font-family: monospace;
            background: #00000099;
            padding: 4px 12px;
            border-radius: 20px;
            color: #7eff9e;
        }
        
        @keyframes nitroFlash {
            0% { text-shadow: 0 0 2px cyan; }
            100% { text-shadow: 0 0 18px #ffaa33; }
        }
        .nitro-active {
            animation: nitroFlash 0.1s infinite alternate;
        }
    </style>
</head>
<body>
<div>
    <div class="racing-container">
        <canvas id="raceCanvas" width="1100" height="600" style="width:100%; height:auto; max-width:1100px; aspect-ratio:1100/600"></canvas>

        <div class="hud">
            <div class="gauge"><h3>🏁 KECEPATAN (KM/H)</h3><div class="stat-value" id="speedValue">0</div></div>
            <div class="gauge"><h3>⚡ NITRO</h3><div class="stat-value" id="nitroValue">100%</div></div>
            <div class="gauge"><h3>🏆 LAP / SCORE</h3><div class="stat-value" id="lapValue">0</div></div>
        </div>

        <div class="btn-group">
            <button class="btn btn-primary" id="nitroBtn">💨 NITRO BOOST 💨</button>
            <button class="btn" id="resetRaceBtn">🔄 RESTART RACE</button>
        </div>
        <div class="speed-panel">
            <span>🎮 TEKAN [SPASI] / KLIK NITRO | GAS OTOMATIS 🚗💨</span>
            <span class="fps-badge" id="fpsMeter">⚡ FPS: --</span>
        </div>
    </div>
</div>

<script>
    (function() {
        // ---------- GAME BALAP MOBIL HD 120FPS, HALUS, SERU ----------
        // Canvas element
        const canvas = document.getElementById("raceCanvas");
        const ctx = canvas.getContext("2d");
        
        // Ukuran jalan dan world
        let roadWidth = 540;
        let roadX = (canvas.width - roadWidth) / 2;
        
        // Parameter mobil player (posisi lateral)
        let playerLane = 0.5;     // 0 = kiri, 1 = kanan (normalized di jalur)
        let targetLane = 0.5;
        let laneSpeed = 0.18;      // kelincahan steering
        
        // ----- data musuh / traffic (mobil lawan) HD -----
        let opponents = [];
        let scoreLaps = 0;          // setiap melewati checkpoint / menyalip kumpulan musuh + poin
        let distanceTravel = 0;
        
        // Kecepatan dasar (akan bertambah tiap detik)
        let speed = 85.0;           // km/h style, untuk animasi scroll
        let baseSpeed = 85;
        let maxSpeed = 380;
        let nitroActive = false;
        let nitroFuel = 100.0;       // persen
        let nitroRegen = 0.45;       // regen per frame (60fps asumsi, tapi dengan delta time)
        
        // Untuk delta time dan FPS tinggi (120fps friendly)
        let lastTimestamp = 0;
        let deltaTime = 1 / 60;      // default
        let frameCount = 0;
        let fps = 60;
        let lastFpsUpdate = 0;
        
        // efek visual
        let screenShake = 0;
        let scrollOffset = 0;
        let roadLines = [];
        
        // spawn musuh & koordinat dunia (Y pseudo 3D)
        let worldScroll = 0;
        let nextSpawnDistance = 0;
        
        // Konfigurasi musuh
        const OPPONENT_CONFIG = [
            { name: "Coupe", color1: "#c02424", color2: "#6a0f0f", width: 48, height: 28, speedFactor: 0.92 },
            { name: "SUV", color1: "#2a6f8f", color2: "#144d66", width: 54, height: 32, speedFactor: 0.85 },
            { name: "Racer", color1: "#e6b422", color2: "#b87c10", width: 46, height: 26, speedFactor: 1.0 },
            { name: "Supercar", color1: "#3cc7d6", color2: "#1b7a8a", width: 50, height: 28, speedFactor: 1.05 }
        ];
        
        // Inisialisasi jalur dan garis jalan
        function initRoadLines() {
            roadLines = [];
            for (let i = 0; i < 35; i++) {
                roadLines.push({ y: (i * 45) % 800, active: true });
            }
        }
        
        // spawn musuh
        function spawnOpponent() {
            let rand = Math.floor(Math.random() * OPPONENT_CONFIG.length);
            let config = { ...OPPONENT_CONFIG[rand] };
            let lanePos = Math.random(); // 0..1 posisi di jalur
            // hindari tabrakan langsung jika terlalu banyak
            opponents.push({
                x: lanePos,
                y: -80 - Math.random() * 100,
                width: config.width,
                height: config.height,
                color1: config.color1,
                color2: config.color2,
                name: config.name,
                speedFactor: config.speedFactor,
                passed: false
            });
        }
        
        // reset race
        function resetRace() {
            opponents = [];
            scoreLaps = 0;
            distanceTravel = 0;
            speed = baseSpeed;
            nitroFuel = 100;
            nitroActive = false;
            playerLane = 0.5;
            targetLane = 0.5;
            worldScroll = 0;
            nextSpawnDistance = 40;
            spawnOpponent();
            spawnOpponent();
            spawnOpponent();
            // tambahkan beberapa musuh di depan
            for(let i=0;i<3;i++) {
                let opp = spawnOpponentManual();
                if(opp) opp.y = -150 - i*180;
            }
            document.getElementById("lapValue").innerText = scoreLaps;
            updateUI();
        }
        
        function spawnOpponentManual() {
            let rand = Math.floor(Math.random() * OPPONENT_CONFIG.length);
            let config = { ...OPPONENT_CONFIG[rand] };
            let lanePos = Math.random();
            opponents.push({
                x: lanePos,
                y: -80 - Math.random() * 150,
                width: config.width,
                height: config.height,
                color1: config.color1,
                color2: config.color2,
                name: config.name,
                speedFactor: config.speedFactor,
                passed: false
            });
        }
        
        // update logika berdasarkan delta time (agar kecepatan frame tidak mempengaruhi gameplay)
        function updateGame(delta) {
            // batas delta maks 0.033 (30fps minimal)
            let dt = Math.min(delta, 0.033);
            
            // --- Nitro logic ---
            if (nitroActive && nitroFuel > 0) {
                let boost = 2.4 * dt;
                speed += boost * 45;
                nitroFuel -= 48 * dt;  // konsumsi cepat
                if (nitroFuel <= 0) {
                    nitroFuel = 0;
                    nitroActive = false;
                }
                screenShake = 5;
            } else {
                nitroActive = false;
                // regenerasi nitro
                if (nitroFuel < 100) {
                    nitroFuel += nitroRegen * dt * 35;
                    if (nitroFuel > 100) nitroFuel = 100;
                }
                // speed berkurang sedikit jika tidak nitro menuju base speed
                if (speed > baseSpeed) {
                    speed -= 18 * dt;
                    if (speed < baseSpeed) speed = baseSpeed;
                } else if (speed < baseSpeed) speed = baseSpeed;
            }
            // batas speed
            speed = Math.min(maxSpeed, Math.max(45, speed));
            
            // --- Pergerakan mobil lateral (halus) ---
            playerLane += (targetLane - playerLane) * Math.min(1, laneSpeed * (dt * 45));
            playerLane = Math.min(0.92, Math.max(0.08, playerLane));
            
            // --- World Scroll & Spawn ---
            let scrollAmount = (speed / 130) * dt * 55;
            worldScroll += scrollAmount;
            distanceTravel += scrollAmount;
            
            // spawn musuh berdasarkan jarak
            if (nextSpawnDistance <= 0) {
                spawnOpponent();
                nextSpawnDistance = 35 + Math.random() * 45;
            } else {
                nextSpawnDistance -= scrollAmount * 1.8;
            }
            
            // update posisi musuh & cek tabrakan & melewati (score)
            for (let i = 0; i < opponents.length; i++) {
                const opp = opponents[i];
                // kecepatan musuh relatif terhadap player speed (musuh sedikit lebih lambat/cepat membuat overtake seru)
                let opponentSpeedDelta = (speed * (opp.speedFactor || 0.92)) * dt * 0.32;
                opp.y += opponentSpeedDelta;
                
                // cek jika musuh melewati player (sebagai penanda menyalip)
                if (!opp.passed && opp.y > (canvas.height - 100)) {
                    opp.passed = true;
                    // menyalip musuh => dapat poin
                    scoreLaps += 1;
                    document.getElementById("lapValue").innerText = scoreLaps;
                    // efek tambahan regenerasi nitro sedikit
                    nitroFuel = Math.min(100, nitroFuel + 6);
                    addFlashEffect();
                }
                
                // tabrakan (collision) antara player dan musuh (dengan hitbox presisi)
                let playerXPos = roadX + (playerLane * roadWidth);
                let playerWidth = 52;
                let playerHeight = 34;
                let oppX = roadX + (opp.x * roadWidth);
                let oppWidth = opp.width;
                let oppHeight = opp.height;
                
                if (opp.y + oppHeight > canvas.height - 110 && opp.y < canvas.height - 50) {
                    if (playerXPos + playerWidth > oppX && playerXPos < oppX + oppWidth) {
                        // KRASH!
                        speed = Math.max(45, speed * 0.55);
                        nitroActive = false;
                        screenShake = 18;
                        // mengurangi nitro
                        nitroFuel = Math.max(0, nitroFuel - 22);
                        // push back player sedikit
                        targetLane = playerLane + (Math.random() - 0.5) * 0.4;
                        targetLane = Math.min(0.92, Math.max(0.08, targetLane));
                        // efek visual pukulan
                        addCrashEffect(oppX, opp.y);
                        // hapus musuh agar tidak double crash (opsional)
                        opponents.splice(i,1);
                        i--;
                        continue;
                    }
                }
                
                // hapus musuh jika sudah terlalu bawah
                if (opp.y > canvas.height + 120) {
                    opponents.splice(i,1);
                    i--;
                }
            }
            
            // menjaga jumlah musuh minimal biar seru
            if (opponents.length < 3 && Math.random() < 0.02 * (dt*60)) {
                spawnOpponent();
            }
            
            // redupkan shake
            screenShake = Math.max(0, screenShake - 2.5 * dt * 50);
            
            // Update UI Kecepatan & Nitro
            updateUI();
        }
        
        let lastCrashTime = 0;
        function addCrashEffect(x, y) {
            // efek visual tidak perlu data lama, cukup untuk animasi canvas
        }
        
        function addFlashEffect() {
            // dipakai untuk overtake flash
        }
        
        function updateUI() {
            document.getElementById("speedValue").innerHTML = Math.floor(speed);
            document.getElementById("nitroValue").innerHTML = Math.floor(nitroFuel) + "%";
            if (nitroActive) {
                document.getElementById("nitroValue").style.color = "#ffaa44";
                document.getElementById("nitroValue").style.textShadow = "0 0 6px orange";
            } else {
                document.getElementById("nitroValue").style.color = "#0ff";
                document.getElementById("nitroValue").style.textShadow = "none";
            }
        }
        
        // ----- GAMBAR HD (detail mobil, jalan, efek lampu, bayangan) -----
        function drawRoad() {
            // aspal
            ctx.fillStyle = "#151e22";
            ctx.fillRect(roadX-8, 0, roadWidth+16, canvas.height);
            ctx.fillStyle = "#1e2c32";
            ctx.fillRect(roadX, 0, roadWidth, canvas.height);
            // garis tengah putus-putus (animasi scroll)
            ctx.lineWidth = 6;
            ctx.strokeStyle = "#f5e56b";
            ctx.shadowBlur = 4;
            ctx.shadowColor = "#ffcc55";
            let lineSpacing = 42;
            let offset = (worldScroll * 2.4) % lineSpacing;
            for (let y = -20; y < canvas.height + 40; y += lineSpacing) {
                let lineY = y + offset;
                if (lineY > 0 && lineY < canvas.height) {
                    ctx.beginPath();
                    ctx.moveTo(roadX + roadWidth/2, lineY);
                    ctx.lineTo(roadX + roadWidth/2, lineY + 24);
                    ctx.stroke();
                }
            }
            // garis pinggir
            ctx.lineWidth = 3;
            ctx.strokeStyle = "#66ffff";
            ctx.shadowBlur = 2;
            ctx.strokeRect(roadX, 0, roadWidth, canvas.height);
            // efek marka pinggir
            for(let i=0;i<12;i++) {
                ctx.fillStyle = "#c9e265";
                ctx.fillRect(roadX-6, (i*70 + worldScroll*3)%canvas.height, 5, 28);
                ctx.fillRect(roadX+roadWidth+2, (i*70 + worldScroll*3)%canvas.height, 5, 28);
            }
            ctx.shadowBlur = 0;
        }
        
        function drawPlayerCar() {
            let carX = roadX + (playerLane * roadWidth) - 26;
            let carY = canvas.height - 82;
            let shakeX = (Math.random() - 0.5) * screenShake * 0.8;
            let shakeY = (Math.random() - 0.5) * screenShake * 0.5;
            // body mobil HD
            ctx.shadowBlur = 12;
            ctx.shadowColor = "#00ddff";
            ctx.fillStyle = "#26c5ff";
            ctx.beginPath();
            ctx.roundRect(carX+shakeX, carY+shakeY, 52, 34, 12);
            ctx.fill();
            ctx.fillStyle = "#0a2f44";
            ctx.beginPath();
            ctx.roundRect(carX+8+shakeX, carY-4+shakeY, 36, 12, 6);
            ctx.fill();
            // kaca
            ctx.fillStyle = "#aaf0ff";
            ctx.fillRect(carX+12+shakeX, carY+6+shakeY, 28, 12);
            ctx.fillStyle = "#ffd966";
            ctx.fillRect(carX+20+shakeX, carY-2+shakeY, 12, 8);
            // lampu
            ctx.fillStyle = "#ff4d4d";
            ctx.fillRect(carX+2+shakeX, carY+22+shakeY, 8, 6);
            ctx.fillRect(carX+42+shakeX, carY+22+shakeY, 8, 6);
            ctx.fillStyle = "#ffff99";
            ctx.fillRect(carX+48+shakeX, carY+8+shakeY, 5, 8);
            // roda
            ctx.fillStyle = "#222";
            ctx.fillRect(carX+4+shakeX, carY+26+shakeY, 10, 8);
            ctx.fillRect(carX+38+shakeX, carY+26+shakeY, 10, 8);
            ctx.fillStyle = "#444";
            ctx.fillRect(carX+6+shakeX, carY+28+shakeY, 6, 5);
            ctx.fillRect(carX+40+shakeX, carY+28+shakeY, 6, 5);
            // efek nitro flame
            if (nitroActive) {
                ctx.fillStyle = "#ff8800";
                ctx.beginPath();
                ctx.moveTo(carX+20+shakeX, carY+38+shakeY);
                ctx.lineTo(carX+30+shakeX, carY+52+shakeY);
                ctx.lineTo(carX+40+shakeX, carY+38+shakeY);
                ctx.fill();
            }
            ctx.shadowBlur = 0;
        }
        
        function drawOpponents() {
            for (let opp of opponents) {
                let oppX = roadX + (opp.x * roadWidth) - opp.width/2;
                let oppY = opp.y;
                if (oppY + opp.height > 0 && oppY < canvas.height) {
                    ctx.shadowBlur = 8;
                    ctx.fillStyle = opp.color1;
                    ctx.beginPath();
                    ctx.roundRect(oppX, oppY, opp.width, opp.height, 10);
                    ctx.fill();
                    ctx.fillStyle = opp.color2;
                    ctx.fillRect(oppX+6, oppY-4, opp.width-12, 12);
                    ctx.fillStyle = "#abc";
                    ctx.fillRect(oppX+10, oppY+8, opp.width-20, 8);
                    // lampu
                    ctx.fillStyle = "#f66";
                    ctx.fillRect(oppX+4, oppY+opp.height-10, 8, 6);
                    ctx.fillRect(oppX+opp.width-12, oppY+opp.height-10, 8, 6);
                }
            }
            ctx.shadowBlur = 0;
        }
        
        function drawSpeedLines() {
            if (speed > 180) {
                ctx.globalAlpha = Math.min(0.6, (speed-150)/250);
                for(let i=0;i<12;i++) {
                    ctx.beginPath();
                    ctx.moveTo(40 + i*90, canvas.height);
                    ctx.lineTo(20 + i*100, canvas.height - 70 - (worldScroll*2)%130);
                    ctx.lineWidth = 3;
                    ctx.strokeStyle = `rgba(0, 200, 255, 0.7)`;
                    ctx.stroke();
                }
                ctx.globalAlpha = 1;
            }
        }
        
        function drawHUDvisuals() {
            ctx.font = "bold 18px 'Orbitron'";
            ctx.fillStyle = "#bbffff";
            ctx.shadowBlur = 0;
            ctx.fillText("🏎️ NITRO READY", roadX+20, 50);
            if (nitroActive) {
                ctx.font = "bold 24px monospace";
                ctx.fillStyle = "#f90";
                ctx.fillText("🔥 NITRO ACTIVE! 🔥", roadX+roadWidth-210, 70);
            }
        }
        
        // helper canvas roundRect
        if (!CanvasRenderingContext2D.prototype.roundRect) {
            CanvasRenderingContext2D.prototype.roundRect = function(x, y, w, h, r) {
                if (w < 2 * r) r = w / 2;
                if (h < 2 * r) r = h / 2;
                this.moveTo(x+r, y);
                this.lineTo(x+w-r, y);
                this.quadraticCurveTo(x+w, y, x+w, y+r);
                this.lineTo(x+w, y+h-r);
                this.quadraticCurveTo(x+w, y+h, x+w-r, y+h);
                this.lineTo(x+r, y+h);
                this.quadraticCurveTo(x, y+h, x, y+h-r);
                this.lineTo(x, y+r);
                this.quadraticCurveTo(x, y, x+r, y);
                return this;
            };
        }
        
        // --- FPS Counter & Loop utama ---
        let frameRequest;
        let lastFrameTime = performance.now();
        
        function gameLoop(now) {
            frameRequest = requestAnimationFrame(gameLoop);
            let dt = Math.min(0.033, (now - lastFrameTime) / 1000);
            if (dt <= 0) { lastFrameTime = now; return; }
            lastFrameTime = now;
            deltaTime = dt;
            
            // Update game dengan delta halus
            updateGame(dt);
            
            // Render semua elemen HD
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            // Efek bayangan kabut
            ctx.fillStyle = "#031016";
            ctx.fillRect(0,0,canvas.width,canvas.height);
            drawRoad();
            drawOpponents();
            drawPlayerCar();
            drawSpeedLines();
            drawHUDvisuals();
            
            // additional HD flare
            if (screenShake > 2) {
                ctx.fillStyle = `rgba(255,80,40,0.1)`;
                ctx.fillRect(0,0,canvas.width,canvas.height);
            }
            
            // update fps meter (setiap 0.5 detik)
            frameCount++;
            let nowSec = performance.now();
            if (nowSec - lastFpsUpdate > 500) {
                fps = Math.round(frameCount / ((nowSec - lastFpsUpdate) / 1000));
                document.getElementById("fpsMeter").innerHTML = `⚡ FPS: ${Math.min(144, fps)}`;
                frameCount = 0;
                lastFpsUpdate = nowSec;
            }
        }
        
        // Event input keyboard & tombol
        function handleKeyDown(e) {
            if (e.code === "Space") {
                e.preventDefault();
                if (!nitroActive && nitroFuel > 5) {
                    nitroActive = true;
                } else if (nitroActive) {
                    // tetap nitro
                }
            }
            if (e.code === "ArrowLeft") {
                targetLane = Math.max(0.08, targetLane - 0.12);
            }
            if (e.code === "ArrowRight") {
                targetLane = Math.min(0.92, targetLane + 0.12);
            }
        }
        
        function handleKeyUp(e) {
            if (e.code === "Space") {
                // nitro bisa dilepas, tetap aktif hingga fuel habis tapi tidak dimatikan langsung agar seru
                // tidak perlu aksi
            }
        }
        
        window.addEventListener("keydown", handleKeyDown);
        window.addEventListener("keyup", handleKeyUp);
        document.getElementById("nitroBtn").addEventListener("click", () => {
            if (!nitroActive && nitroFuel > 5) nitroActive = true;
        });
        document.getElementById("resetRaceBtn").addEventListener("click", () => {
            resetRace();
        });
        
        // Touch/mouse untuk kontrol lateral (opsional)
        canvas.addEventListener("mousemove", (e) => {
            let rect = canvas.getBoundingClientRect();
            let scaleX = canvas.width / rect.width;
            let mouseX = (e.clientX - rect.left) * scaleX;
            if (mouseX > roadX && mouseX < roadX+roadWidth) {
                let rawLane = (mouseX - roadX) / roadWidth;
                targetLane = Math.min(0.92, Math.max(0.08, rawLane));
            }
        });
        
        // init game
        initRoadLines();
        resetRace();
        
        lastFrameTime = performance.now();
        gameLoop(lastFrameTime);
    })();
</script>
</body>
</html>
