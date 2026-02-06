[David_vs_Goalithv2.html](https://github.com/user-attachments/files/25129199/David_vs_Goalithv2.html)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>David vs. Goliath: Elite Challenge</title>
    <style>
        body {
            margin: 0;
            background: #1a1a1a;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            color: white;
            font-family: 'Segoe UI', sans-serif;
            overflow: hidden;
        }
        canvas {
            background: #87CEEB;
            border: 6px solid #3e2723;
            box-shadow: 0 0 30px rgba(0,0,0,0.7);
        }
        .ui-container {
            display: flex;
            justify-content: space-between;
            width: 800px;
            margin-bottom: 10px;
        }
        .stats { text-align: center; }
        .hearts { color: #e74c3c; font-size: 1.8rem; text-shadow: 2px 2px #000; }
        .giant-hearts { color: #f1c40f; font-size: 1.8rem; text-shadow: 2px 2px #000; }
        h2 { margin: 0; text-transform: uppercase; letter-spacing: 3px; }
    </style>
</head>
<body>


    <div class="ui-container">
        <div class="stats">
            <div>DAVID</div>
            <div id="davidHearts" class="hearts">❤️❤️❤️</div>
        </div>
        <div class="stats">
            <h2 id="message">THE FINAL STRIKE</h2>
        </div>
        <div class="stats">
            <div>GOLIATH</div>
            <div id="goliathHearts" class="giant-hearts">💛</div>
        </div>
    </div>


    <canvas id="gameCanvas" width="800" height="400"></canvas>


<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const msg = document.getElementById('message');
    const dHeartsDiv = document.getElementById('davidHearts');
    const gHeartsDiv = document.getElementById('goliathHearts');


    const GROUND_Y = 350;
    const GRAVITY = 0.22;


    let isCharging = false;
    let chargeLevel = 0;
    const maxCharge = 100;
    let gameState = "playing"; 


    let stone = { x: 0, y: 0, vx: 0, vy: 0, active: false };
    let spears = [];
    let david = { x: 80, y: GROUND_Y - 40, width: 30, height: 40, lives: 3 };
    
    let goliath = { 
        x: 650, y: GROUND_Y - 120, width: 60, height: 120, 
        lives: 1, attackTimer: 0, isWindingUp: false,
        speed: 1.5, direction: 1 // 1 for right, -1 for left
    };


    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' && !stone.active && gameState === "playing") isCharging = true;
    });
    window.addEventListener('keyup', (e) => {
        if (e.code === 'Space' && isCharging) { fireStone(); isCharging = false; }
    });


    function fireStone() {
        stone.active = true;
        stone.x = david.x + 30; stone.y = david.y + 10;
        stone.vx = (chargeLevel / 10) + 6.5; 
        stone.vy = -(chargeLevel / 28) - 4;
        chargeLevel = 0;
    }


    function spawnSpear() {
        if (gameState !== "playing") return;
        
        // ELITE ACCURACY: Higher velocity and tighter tracking
        const dx = (david.x + 15) - goliath.x;
        const dy = (david.y + 20) - (goliath.y + 40);
        
        spears.push({
            x: goliath.x, 
            y: goliath.y + 40,
            vx: (dx / 45), // Faster travel speed
            vy: (dy / 45) - 1, // Compensate for slight gravity drop
            active: true
        });
    }


    function drawBackground() {
        let skyGrd = ctx.createLinearGradient(0, 0, 0, GROUND_Y);
        skyGrd.addColorStop(0, "#0f2027"); skyGrd.addColorStop(1, "#2c5364");
        ctx.fillStyle = skyGrd; ctx.fillRect(0, 0, canvas.width, GROUND_Y);
    }


    function update() {
        if (gameState !== "playing") return;


        if (isCharging && chargeLevel < maxCharge) chargeLevel += 2; // Faster charge for elite mode


        // Goliath Movement (Strafing)
        goliath.x += goliath.direction * goliath.speed;
        if (goliath.x > 700 || goliath.x < 450) goliath.direction *= -1;


        // Goliath AI
        goliath.attackTimer++;
        if (goliath.attackTimer > 60) goliath.isWindingUp = true;
        if (goliath.attackTimer > 90) { // Very fast attack rate
            spawnSpear();
            goliath.attackTimer = 0;
            goliath.isWindingUp = false;
        }


        // Stone Physics
        if (stone.active) {
            stone.x += stone.vx; stone.y += stone.vy; stone.vy += GRAVITY;
            if (stone.x > goliath.x && stone.x < goliath.x + goliath.width &&
                stone.y > goliath.y && stone.y < goliath.y + goliath.height) {
                
                // Headshot detection (Only the top 15% of Goliath)
                if (stone.y < goliath.y + 20) {
                    goliath.lives--;
                    stone.active = false;
                    updateUI();
                    if (goliath.lives <= 0) {
                        gameState = "won";
                        msg.innerText = "THE GIANT HAS FALLEN!";
                    }
                } else { stone.active = false; }
            }
            if (stone.x > canvas.width || stone.y > GROUND_Y) stone.active = false;
        }


        // Spear Physics
        spears.forEach(s => {
            if (!s.active) return;
            s.x += s.vx; s.y += s.vy; s.vy += (GRAVITY * 0.3);
            if (s.x < david.x + david.width && s.x > david.x &&
                s.y > david.y && s.y < david.y + david.height) {
                david.lives--;
                s.active = false;
                updateUI();
                if (david.lives <= 0) {
                    gameState = "lost";
                    msg.innerText = "GOLIATH TRIUMPHS";
                }
            }
            if (s.x < 0 || s.y > GROUND_Y) s.active = false;
        });
    }


    function updateUI() {
        let dh = ""; for(let i=0; i<david.lives; i++) dh += "❤️";
        dHeartsDiv.innerText = dh || "💀";
        let gh = ""; for(let i=0; i<goliath.lives; i++) gh += "💛";
        gHeartsDiv.innerText = gh || "💀";
    }


    function draw() {
        drawBackground();
        ctx.fillStyle = "#3e2723"; ctx.fillRect(0, GROUND_Y, canvas.width, 50); // Ground


        // David
        ctx.fillStyle = "#2196f3"; ctx.fillRect(david.x, david.y, david.width, david.height);


        // Goliath
        if (gameState !== "won") {
            ctx.fillStyle = goliath.isWindingUp ? "#f44336" : "#455a64";
            ctx.fillRect(goliath.x, goliath.y, goliath.width, goliath.height);
            ctx.fillStyle = "#ffeb3b"; // Forehead (Smaller hit box)
            ctx.fillRect(goliath.x, goliath.y, goliath.width, 20);
        } else {
            ctx.save(); ctx.translate(goliath.x + 30, goliath.y + 110); ctx.rotate(Math.PI/2);
            ctx.fillStyle = "#455a64"; ctx.fillRect(-110, -30, 120, 60); ctx.restore();
        }


        // Projectiles
        if (stone.active) {
            ctx.fillStyle = "#fff"; ctx.beginPath(); ctx.arc(stone.x, stone.y, 6, 0, Math.PI*2); ctx.fill();
        }
        ctx.fillStyle = "#cfd8dc";
        spears.forEach(s => { if (s.active) ctx.fillRect(s.x, s.y, 30, 3); });


        // Charge Meter
        if (isCharging) {
            ctx.fillStyle = "rgba(0,0,0,0.5)"; ctx.fillRect(david.x - 10, david.y - 30, 50, 8);
            ctx.fillStyle = `rgb(${chargeLevel*2.5}, 50, 255)`;
            ctx.fillRect(david.x - 10, david.y - 30, (chargeLevel/maxCharge)*50, 8);
        }


        if (gameState === "lost") {
            ctx.save(); ctx.translate(david.x + 15, david.y + 35); ctx.rotate(-Math.PI/2);
            ctx.fillStyle = "#2196f3"; ctx.fillRect(-35, -15, 40, 30); ctx.restore();
        }


        requestAnimationFrame(() => { update(); draw(); });
    }
    draw();
</script>
</body>
</html>
