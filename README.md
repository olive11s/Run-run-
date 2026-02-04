# Run-run-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Seattle Dash - Giselle ❤️</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; touch-action: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #192a56; font-family: -apple-system, sans-serif; transition: background 1s; }

        #game-area { position: relative; width: 100%; height: 100%; background: #2f3640; overflow: hidden; transition: background 1s; }
        .line { position: absolute; top: 0; bottom: 0; width: 4px; background: rgba(255,255,255,0.3); left: 33.3%; }
        .line-2 { left: 66.6%; }

        #player {
            position: absolute; bottom: 12%; width: 65px; height: 65px;
            font-size: 55px; display: flex; justify-content: center; align-items: center;
            transition: left 0.15s ease-out; z-index: 100; transform: translateX(-50%);
        }
        
        .invincible { filter: drop-shadow(0 0 20px gold); }
        .heart-trail { position: absolute; font-size: 20px; pointer-events: none; z-index: 90; animation: fadeUp 0.6s forwards; }
        @keyframes fadeUp { 0% { opacity: 1; transform: translate(-50%, 0) scale(1); } 100% { opacity: 0; transform: translate(-50%, -100px) scale(1.5); } }

        #hud { position: absolute; top: 55px; width: 100%; display: flex; flex-direction: column; align-items: center; color: white; font-weight: bold; z-index: 200; text-shadow: 0 2px 4px #000; }
        .progress-bar { width: 75%; height: 12px; background: rgba(255,255,255,0.2); border-radius: 6px; margin-top: 8px; overflow: hidden; border: 1px solid rgba(255,255,255,0.3); }
        #progress-fill { width: 0%; height: 100%; background: #00d2d3; transition: width 0.3s; }

        .entity { position: absolute; font-size: 45px; transform: translateX(-50%); z-index: 50; }
        .screen { position: fixed; inset: 0; z-index: 500; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; background: rgba(0,0,0,0.92); color: white; padding: 30px; }
        .btn { padding: 18px 45px; border-radius: 50px; border: none; background: #00d2d3; color: white; font-weight: bold; font-size: 1.2rem; box-shadow: 0 4px 15px rgba(0,0,0,0.4); }
    </style>
</head>
<body>

<div id="game-area">
    <div class="line"></div>
    <div class="line-2"></div>
    <div id="player">🏃‍♀️</div>
</div>

<div id="hud">
    <div>Tickets: <span id="tickets">0</span>/15</div>
    <div class="progress-bar"><div id="progress-fill"></div></div>
    <div style="font-size: 11px; margin-top: 5px; letter-spacing: 1px;">ROAD TO SEATTLE 🏔️</div>
</div>

<div id="start-screen" class="screen">
    <h1 style="color: #00d2d3; font-size: 2rem;">The Road to Seattle! ✈️</h1>
    <p style="margin: 15px 0;">Dodge study stress and grab 15 tickets!</p>
    <button onclick="startGame()" class="btn">START VACATION 🎶</button>
</div>

<div id="win-screen" class="screen" style="display:none; background: #fff; color: #333;">
    <h1 style="color: #00d2d3;">Welcome to Seattle! 🏔️</h1>
    <p style="margin: 20px 0;">Time for citizenM!<br>I love you, Giselle! ❤️</p>
</div>

<audio id="travelMusic" loop><source src="https://www.bensound.com/bensound-music/bensound-sunny.mp3" type="audio/mpeg"></audio>

<script>
    let currentLane = 1, lanes = [16.6, 50, 83.3];
    let tickets = 0, isInvincible = false, gameActive = false, frame = 0;
    const music = document.getElementById('travelMusic');

    function startGame() {
        document.getElementById('start-screen').style.display = 'none';
        gameActive = true;
        music.play().catch(() => {});
        spawnLoop();
    }

    let startX = 0;
    document.addEventListener('touchstart', (e) => startX = e.touches[0].clientX);
    document.addEventListener('touchend', (e) => {
        if (!gameActive) return;
        let diff = e.changedTouches[0].clientX - startX;
        if (Math.abs(diff) > 20) {
            if (diff > 0 && currentLane < 2) currentLane++;
            else if (diff < 0 && currentLane > 0) currentLane--;
            document.getElementById('player').style.left = lanes[currentLane] + '%';
        }
    });

    function spawnLoop() {
        if (!gameActive) return;
        frame++;
        createEntity(frame < 4); 
        setTimeout(spawnLoop, Math.max(700, 1100 - (tickets * 35)));
    }

    function createEntity(safeStart) {
        const entity = document.createElement('div');
        entity.className = 'entity';
        const lane = Math.floor(Math.random() * 3);
        const rand = Math.random();
        let type = 'good';
        if (!safeStart && rand > 0.94) type = 'power';
        else if (!safeStart && rand > 0.60) type = 'bad';

        entity.innerHTML = type === 'good' ? '✈️' : (type === 'power' ? '🔑' : '📚');
        entity.dataset.type = type;
        entity.style.left = lanes[lane] + '%';
        entity.style.top = '-60px';
        document.getElementById('game-area').appendChild(entity);

        let top = -60;
        const move = setInterval(() => {
            if (!gameActive) { clearInterval(move); return; }
            top += (isInvincible ? 12 : 7 + (tickets * 0.2));
            entity.style.top = top + 'px';

            const pRect = document.getElementById('player').getBoundingClientRect();
            const eRect = entity.getBoundingClientRect();

            if (pRect.top < eRect.bottom - 12 && pRect.bottom > eRect.top + 12 && 
                pRect.left < eRect.right - 12 && pRect.right > eRect.left + 12) {
                handleCollision(entity, type, move);
            }

            if (top > window.innerHeight) { clearInterval(move); entity.remove(); }
        }, 16);
    }

    function handleCollision(entity, type, interval) {
        if (type === 'good') {
            tickets++;
            document.getElementById('tickets').innerText = tickets;
            updateAtmosphere();
            if (tickets >= 15) win();
        } else if (type === 'power') {
            activatePowerUp();
        } else if (type === 'bad' && !isInvincible) {
            gameActive = false;
            alert("Study stress caught up! Let's try again.");
            location.reload();
        }
        clearInterval(interval);
        entity.remove();
    }

    function updateAtmosphere() {
        const progress = tickets / 15;
        document.getElementById('progress-fill').style.width = (progress * 100) + '%';
        const area = document.getElementById('game-area');
        if (progress > 0.7) area.style.background = "#00d2d3";
        else if (progress > 0.3) area.style.background = "#f0932b";
    }

    function activatePowerUp() {
        isInvincible = true;
        const p = document.getElementById('player');
        p.innerHTML = '✈️';
        p.classList.add('invincible');
        
        const trailInterval = setInterval(() => {
            if (!isInvincible) { clearInterval(trailInterval); return; }
            const heart = document.createElement('div');
            heart.className = 'heart-trail';
            heart.innerHTML = '❤️';
            heart.style.left = p.style.left;
            heart.style.bottom = '12%';
            document.getElementById('game-area').appendChild(heart);
            setTimeout(() => heart.remove(), 600);
        }, 100);

        setTimeout(() => {
            isInvincible = false;
            p.innerHTML = '🏃‍♀️';
            p.classList.remove('invincible');
        }, 4000);
    }

    function win() {
        gameActive = false;
        document.getElementById('win-screen').style.display = 'flex';
    }

    document.getElementById('player').style.left = lanes[currentLane] + '%';
</script>
</body>
</html>
