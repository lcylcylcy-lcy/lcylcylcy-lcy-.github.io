<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2D彈道射擊</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }
        
        body {
            font-family: 'Arial', sans-serif;
            background: #000;
            overflow: hidden;
            width: 100%;
            height: 100vh;
            color: white;
            margin: 0;
            touch-action: manipulation;
        }
        
        #game-container {
            position: relative;
            width: 100%;
            height: 100%;
            overflow: hidden;
        }
        
        #player {
            position: absolute;
            width: 60px;
            height: 60px;
            bottom: 50px;
            left: 50%;
            transform: translateX(-50%);
            background: radial-gradient(circle at center, #2ecc71, #27ae60);
            border-radius: 50%;
            z-index: 10;
            transition: left 0.1s ease-out;
            box-shadow: 0 0 20px #2ecc71;
            font-size: 30px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        
        .enemy {
            position: absolute;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            z-index: 5;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 30px;
            text-shadow: 0 0 5px black;
        }
        
        .enemy.normal {
            background: radial-gradient(circle at center, #e74c3c, #c0392b);
        }
        
        .enemy.fast {
            background: radial-gradient(circle at center, #3498db, #2980b9);
        }
        
        .enemy.tank {
            background: radial-gradient(circle at center, #7f8c8d, #34495e);
        }
        
        .enemy.boss {
            width: 100px;
            height: 100px;
            background: radial-gradient(circle at center, #9b59b6, #8e44ad);
            font-size: 50px;
            animation: bossPulse 1s infinite alternate;
        }
        
        @keyframes bossPulse {
            0% { transform: scale(1); }
            100% { transform: scale(1.1); }
        }
        
        .bullet {
            position: absolute;
            width: 6px;
            height: 15px;
            background: linear-gradient(to bottom, #f1c40f, #f39c12);
            border-radius: 3px;
            z-index: 8;
            box-shadow: 0 0 10px #f1c40f;
        }
        
        .upgrade-door {
            position: absolute;
            width: 80px;
            height: 30px;
            border-radius: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            font-size: 14px;
            z-index: 7;
            box-shadow: 0 5px 15px rgba(0,0,0,0.5);
            transition: all 0.2s;
        }
        
        .door-fire-rate {
            background: linear-gradient(135deg, #3498db, #2980b9);
            border: 2px solid #2c3e50;
        }
        
        .door-damage {
            background: linear-gradient(135deg, #e74c3c, #c0392b);
            border: 2px solid #7f8c8d;
        }
        
        .door-bullet {
            background: linear-gradient(135deg, #2ecc71, #27ae60);
            border: 2px solid #16a085;
        }
        
        #hud {
            position: absolute;
            top: 10px;
            left: 0;
            width: 100%;
            padding: 10px;
            display: flex;
            justify-content: space-around;
            z-index: 20;
            background: rgba(0, 0, 0, 0.5);
        }
        
        .hud-item {
            padding: 8px 15px;
            border-radius: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .hud-value {
            font-size: 18px;
            font-weight: bold;
            color: #f1c40f;
        }
        
        .hud-label {
            font-size: 12px;
            color: #ecf0f1;
        }
        
        #health-bar {
            position: absolute;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 5px;
            background: rgba(0, 0, 0, 0.5);
            z-index: 10;
        }
        
        #health-fill {
            height: 100%;
            background: linear-gradient(90deg, #2ecc71, #27ae60);
            width: 100%;
            transition: width 0.3s;
        }
        
        .explosion {
            position: absolute;
            width: 50px;
            height: 50px;
            background: radial-gradient(circle, #f39c12 0%, #e67e22 70%, transparent 100%);
            border-radius: 50%;
            z-index: 9;
            animation: explode 0.5s forwards;
        }
        
        @keyframes explode {
            0% { transform: scale(0.5); opacity: 1; }
            100% { transform: scale(2); opacity: 0; }
        }
        
        #game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 30;
        }
        
        #game-over h2 {
            color: #e74c3c;
            font-size: 36px;
            margin-bottom: 20px;
            text-shadow: 0 0 10px #e74c3c;
        }
        
        #restart-btn {
            padding: 12px 30px;
            background: linear-gradient(135deg, #2ecc71, #27ae60);
            border: none;
            border-radius: 25px;
            color: white;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 20px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.5);
        }
        
        .floating-text {
            position: absolute;
            font-weight: bold;
            font-size: 18px;
            z-index: 6;
            animation: floatUp 1s forwards;
            text-shadow: 0 0 5px currentColor;
        }
        
        @keyframes floatUp {
            0% { transform: translateY(0); opacity: 1; }
            100% { transform: translateY(-50px); opacity: 0; }
        }
        
        #background {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(to bottom, #1a1a2e, #16213e);
            z-index: -1;
        }
        
        .star {
            position: absolute;
            width: 2px;
            height: 2px;
            background: white;
            border-radius: 50%;
            animation: twinkle 2s infinite alternate;
        }
        
        @keyframes twinkle {
            0% { opacity: 0.2; }
            100% { opacity: 1; }
        }
        
        .controls-info {
            position: absolute;
            bottom: 10px;
            left: 0;
            width: 100%;
            text-align: center;
            font-size: 12px;
            color: #7f8c8d;
            z-index: 15;
        }
        
        #boss-health-bar {
            position: absolute;
            top: 60px;
            left: 10%;
            width: 80%;
            height: 10px;
            background: rgba(0, 0, 0, 0.5);
            z-index: 20;
            display: none;
        }
        
        #boss-health-fill {
            height: 100%;
            background: linear-gradient(90deg, #e74c3c, #c0392b);
            width: 100%;
            transition: width 0.3s;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="background"></div>
        <div id="hud">
            <div class="hud-item">
                <div class="hud-value" id="score">0</div>
                <div class="hud-label">分數</div>
            </div>
            <div class="hud-item">
                <div class="hud-value" id="level">1</div>
                <div class="hud-label">關卡</div>
            </div>
            <div class="hud-item">
                <div class="hud-value" id="fire-rate">3.0</div>
                <div class="hud-label">攻速/秒</div>
            </div>
            <div class="hud-item">
                <div class="hud-value" id="damage">15</div>
                <div class="hud-label">傷害</div>
            </div>
            <div class="hud-item">
                <div class="hud-value" id="bullet-lines">1</div>
                <div class="hud-label">彈道行數</div>
            </div>
        </div>
        
        <div id="player">🚀</div>
        <div id="health-bar"><div id="health-fill"></div></div>
        <div id="boss-health-bar"><div id="boss-health-fill"></div></div>
        
        <div class="controls-info">
            使用 ← → 鍵或觸摸屏幕移動 | 自動射擊
        </div>
        
        <div id="game-over">
            <h2>遊戲結束</h2>
            <div class="hud-item">
                <div class="hud-value" id="final-score">0</div>
                <div class="hud-label">最終分數</div>
            </div>
            <div class="hud-item">
                <div class="hud-value" id="final-level">1</div>
                <div class="hud-label">達成關卡</div>
            </div>
            <button id="restart-btn">重新開始</button>
        </div>
    </div>

    <script>
        // 遊戲狀態
        const game = {
            score: 0,
            level: 1,
            playerHealth: 100,
            maxHealth: 100,
            baseFireRate: 3.0,
            fireRateMultiplier: 1,
            damage: 15,
            isGameOver: false,
            playerX: 50,
            enemies: [],
            bullets: [],
            upgrades: [],
            explosions: [],
            floatingTexts: [],
            stars: [],
            enemySpawnRate: 1500,
            upgradeSpawnRate: 8000,
            lastEnemySpawn: 0,
            lastUpgradeSpawn: 0,
            lastFireTime: 0,
            touchX: null,
            bulletLines: 1,
            bulletLevel: 1,
            upgradeActive: false,
            keys: {
                left: false,
                right: false
            },
            playerSpeed: 5,
            bossActive: false,
            bossHealth: 0,
            maxBossHealth: 0,
            bossMoveDirection: 1,
            bossMoveSpeed: 2
        };
        
        // DOM元素
        const player = document.getElementById('player');
        const gameContainer = document.getElementById('game-container');
        const background = document.getElementById('background');
        const scoreDisplay = document.getElementById('score');
        const levelDisplay = document.getElementById('level');
        const fireRateDisplay = document.getElementById('fire-rate');
        const damageDisplay = document.getElementById('damage');
        const bulletLinesDisplay = document.getElementById('bullet-lines');
        const healthFill = document.getElementById('health-fill');
        const gameOverScreen = document.getElementById('game-over');
        const finalScoreDisplay = document.getElementById('final-score');
        const finalLevelDisplay = document.getElementById('final-level');
        const restartBtn = document.getElementById('restart-btn');
        const bossHealthBar = document.getElementById('boss-health-bar');
        const bossHealthFill = document.getElementById('boss-health-fill');
        
        // 敵人類型和emoji
        const enemyTypes = [
            { type: 'normal', emoji: '👾', speed: 2, health: 20, color: 'normal', spawnRate: 0.6 },
            { type: 'fast', emoji: '👽', speed: 4, health: 15, color: 'fast', spawnRate: 0.3 },
            { type: 'tank', emoji: '🤖', speed: 1.5, health: 40, color: 'tank', spawnRate: 0.1 }
        ];
        
        // BOSS類型
        const bossTypes = [
            { emoji: '👹', healthMultiplier: 10, speed: 2 },
            { emoji: '👻', healthMultiplier: 8, speed: 3 },
            { emoji: '🐉', healthMultiplier: 15, speed: 1.5 }
        ];
        
        // 遊戲初始化
        function initGame() {
            game.score = 0;
            game.level = 1;
            game.playerHealth = 100;
            game.baseFireRate = 3.0;
            game.fireRateMultiplier = 1;
            game.damage = 15;
            game.isGameOver = false;
            game.playerX = 50;
            game.enemies = [];
            game.bullets = [];
            game.upgrades = [];
            game.explosions = [];
            game.floatingTexts = [];
            game.enemySpawnRate = 1500;
            game.upgradeSpawnRate = 8000;
            game.lastEnemySpawn = 0;
            game.lastUpgradeSpawn = 0;
            game.lastFireTime = 0;
            game.touchX = null;
            game.bulletLines = 1;
            game.bulletLevel = 1;
            game.upgradeActive = false;
            game.keys = { left: false, right: false };
            game.bossActive = false;
            game.bossHealth = 0;
            
            updateHUD();
            gameOverScreen.style.display = 'none';
            bossHealthBar.style.display = 'none';
            
            // 清除所有遊戲元素
            document.querySelectorAll('.enemy, .bullet, .upgrade-door, .explosion, .floating-text, .star').forEach(el => {
                if (el.parentNode) el.parentNode.removeChild(el);
            });
            
            // 重置玩家位置
            updatePlayerPosition();
            
            // 創建星空背景
            createStars();
        }
        
        // 創建星空背景
        function createStars() {
            for (let i = 0; i < 100; i++) {
                const star = document.createElement('div');
                star.className = 'star';
                
                const x = Math.random() * 100;
                const y = Math.random() * 100;
                const size = Math.random() * 2 + 1;
                const delay = Math.random() * 2;
                
                star.style.left = `${x}%`;
                star.style.top = `${y}%`;
                star.style.width = `${size}px`;
                star.style.height = `${size}px`;
                star.style.animationDelay = `${delay}s`;
                
                background.appendChild(star);
                game.stars.push(star);
            }
        }
        
        // 更新HUD
        function updateHUD() {
            scoreDisplay.textContent = game.score;
            levelDisplay.textContent = game.level;
            fireRateDisplay.textContent = (game.baseFireRate * game.fireRateMultiplier).toFixed(1);
            damageDisplay.textContent = game.damage;
            bulletLinesDisplay.textContent = game.bulletLines;
            healthFill.style.width = `${(game.playerHealth / game.maxHealth) * 100}%`;
            
            if (game.bossActive) {
                bossHealthFill.style.width = `${(game.bossHealth / game.maxBossHealth) * 100}%`;
            }
        }
        
        // 更新玩家位置
        function updatePlayerPosition() {
            const containerWidth = gameContainer.clientWidth;
            const playerWidth = player.clientWidth;
            const maxX = containerWidth - playerWidth;
            
            let x = (game.playerX / 100) * containerWidth;
            x = Math.max(0, Math.min(maxX, x));
            player.style.left = `${x}px`;
        }
        
        // 處理鍵盤輸入
        function handleKeyboard() {
            if (game.keys.left) {
                game.playerX -= game.playerSpeed;
            }
            if (game.keys.right) {
                game.playerX += game.playerSpeed;
            }
            game.playerX = Math.max(0, Math.min(100, game.playerX));
            updatePlayerPosition();
        }
        
        // 生成敵人
        function spawnEnemy() {
            // 每5關生成BOSS
            if (game.level % 5 === 0 && !game.bossActive && game.enemies.length === 0) {
                spawnBoss();
                return;
            }
            
            // 根據權重隨機選擇敵人類型
            const rand = Math.random();
            let selectedType = enemyTypes[0];
            let cumulativeRate = 0;
            
            for (const type of enemyTypes) {
                cumulativeRate += type.spawnRate;
                if (rand < cumulativeRate) {
                    selectedType = type;
                    break;
                }
            }
            
            const enemy = document.createElement('div');
            enemy.className = `enemy ${selectedType.color}`;
            enemy.textContent = selectedType.emoji;
            
            const containerWidth = gameContainer.clientWidth;
            const x = Math.random() * (containerWidth - 50);
            
            enemy.style.left = `${x}px`;
            enemy.style.top = '-50px';
            gameContainer.appendChild(enemy);
            
            const health = selectedType.health + game.level * 5;
            const speed = selectedType.speed + game.level * 0.1;
            
            game.enemies.push({
                element: enemy,
                x: x,
                y: -50,
                speed: speed,
                health: health,
                maxHealth: health,
                type: selectedType.type,
                emoji: selectedType.emoji,
                id: Date.now() + Math.random()
            });
        }
        
        // 生成BOSS
        function spawnBoss() {
            game.bossActive = true;
            const bossType = bossTypes[Math.floor(Math.random() * bossTypes.length)];
            
            const boss = document.createElement('div');
            boss.className = 'enemy boss';
            boss.textContent = bossType.emoji;
            
            const containerWidth = gameContainer.clientWidth;
            const x = containerWidth / 2 - 50;
            
            boss.style.left = `${x}px`;
            boss.style.top = '-100px';
            gameContainer.appendChild(boss);
            
            game.maxBossHealth = 200 + (game.level * 20 * bossType.healthMultiplier);
            game.bossHealth = game.maxBossHealth;
            
            bossHealthBar.style.display = 'block';
            updateHUD();
            
            game.enemies.push({
                element: boss,
                x: x,
                y: -100,
                speed: bossType.speed,
                health: game.maxBossHealth,
                maxHealth: game.maxBossHealth,
                type: 'boss',
                emoji: bossType.emoji,
                id: Date.now() + Math.random(),
                isBoss: true
            });
            
            createFloatingText(`BOSS 出現!`, containerWidth / 2, 100, 0, '#e74c3c');
        }
        
        // 生成升級門
        function spawnUpgrade() {
            clearAllUpgrades();
            
            const containerWidth = gameContainer.clientWidth;
            const doorWidth = 80;
            const gap = (containerWidth - doorWidth * 3) / 4;
            
            // 左側攻速門
            const fireRateDoor = document.createElement('div');
            fireRateDoor.className = 'upgrade-door door-fire-rate';
            fireRateDoor.textContent = '攻速×2';
            fireRateDoor.dataset.type = 'fireRate';
            
            const leftX = gap;
            fireRateDoor.style.left = `${leftX}px`;
            fireRateDoor.style.top = '-30px';
            gameContainer.appendChild(fireRateDoor);
            
            game.upgrades.push({
                element: fireRateDoor,
                x: leftX,
                y: -30,
                speed: 3,
                type: 'fireRate',
                id: Date.now() + Math.random()
            });
            
            // 中間彈道門
            const bulletDoor = document.createElement('div');
            bulletDoor.className = 'upgrade-door door-bullet';
            bulletDoor.textContent = '彈道+1';
            bulletDoor.dataset.type = 'bullet';
            
            const centerX = gap * 2 + doorWidth;
            bulletDoor.style.left = `${centerX}px`;
            bulletDoor.style.top = '-30px';
            gameContainer.appendChild(bulletDoor);
            
            game.upgrades.push({
                element: bulletDoor,
                x: centerX,
                y: -30,
                speed: 3,
                type: 'bullet',
                id: Date.now() + Math.random()
            });
            
            // 右側傷害門
            const damageDoor = document.createElement('div');
            damageDoor.className = 'upgrade-door door-damage';
            damageDoor.textContent = '傷害+10';
            damageDoor.dataset.type = 'damage';
            
            const rightX = gap * 3 + doorWidth * 2;
            damageDoor.style.left = `${rightX}px`;
            damageDoor.style.top = '-30px';
            gameContainer.appendChild(damageDoor);
            
            game.upgrades.push({
                element: damageDoor,
                x: rightX,
                y: -30,
                speed: 3,
                type: 'damage',
                id: Date.now() + Math.random()
            });
            
            game.upgradeActive = false;
        }
        
        // 清除所有升級門
        function clearAllUpgrades() {
            const upgrades = [...game.upgrades];
            upgrades.forEach(upgrade => {
                if (upgrade.element && upgrade.element.parentNode) {
                    upgrade.element.parentNode.removeChild(upgrade.element);
                }
            });
            game.upgrades = [];
        }
        
        // 玩家射擊
        function fireBullet() {
            const now = Date.now();
            const actualFireRate = game.baseFireRate * game.fireRateMultiplier;
            
            if (now - game.lastFireTime < 1000 / actualFireRate) return;
            game.lastFireTime = now;
            
            const playerRect = player.getBoundingClientRect();
            const centerX = playerRect.left + playerRect.width / 2;
            const centerY = playerRect.top;
            
            // 根據彈道行數發射子彈
            for (let line = 0; line < game.bulletLines; line++) {
                // 計算每行的偏移量
                const lineOffset = (line - (game.bulletLines - 1) / 2) * 30; // 均勻分布
                createBullet(centerX + lineOffset, centerY);
            }
        }
        
        // 創建子彈
        function createBullet(x, y) {
            const bullet = document.createElement('div');
            bullet.className = 'bullet';
            
            bullet.style.left = `${x - 3}px`;
            bullet.style.top = `${y}px`;
            gameContainer.appendChild(bullet);
            
            game.bullets.push({
                element: bullet,
                x: x,
                y: y,
                speed: 8,
                damage: game.damage,
                id: Date.now() + Math.random()
            });
        }
        
        // 創建爆炸效果
        function createExplosion(x, y) {
            const explosion = document.createElement('div');
            explosion.className = 'explosion';
            explosion.style.left = `${x - 25}px`;
            explosion.style.top = `${y - 25}px`;
            gameContainer.appendChild(explosion);
            
            game.explosions.push({
                element: explosion,
                time: Date.now(),
                id: Date.now() + Math.random()
            });
        }
        
        // 創建浮動文字
        function createFloatingText(text, x, y, color = '#f1c40f') {
            const floatingText = document.createElement('div');
            floatingText.className = 'floating-text';
            floatingText.textContent = text;
            floatingText.style.left = `${x}px`;
            floatingText.style.top = `${y}px`;
            floatingText.style.color = color;
            gameContainer.appendChild(floatingText);
            
            game.floatingTexts.push({
                element: floatingText,
                time: Date.now(),
                id: Date.now() + Math.random()
            });
        }
        
        // 處理升級
        function processUpgrade(upgrade) {
            if (game.upgradeActive) return;
            game.upgradeActive = true;
            
            let upgradeText = '';
            let upgradeColor = '#3498db';
            
            if (upgrade.type === 'fireRate') {
                game.fireRateMultiplier *= 2;
                upgradeText = '攻速×2!';
                upgradeColor = '#3498db';
            } else if (upgrade.type === 'damage') {
                game.damage += 10;
                upgradeText = '傷害+10!';
                upgradeColor = '#e74c3c';
            } else if (upgrade.type === 'bullet') {
                game.bulletLines += 1; // 增加彈道行數
                upgradeText = `彈道 ${game.bulletLines}行!`;
                upgradeColor = '#2ecc71';
            }
            
            const playerRect = player.getBoundingClientRect();
            createFloatingText(upgradeText, playerRect.left + playerRect.width / 2, playerRect.top, upgradeColor);
            updateHUD();
            
            // 安全移除升級門
            if (upgrade.element && upgrade.element.parentNode) {
                upgrade.element.parentNode.removeChild(upgrade.element);
            }
            
            // 從數組中移除
            game.upgrades = game.upgrades.filter(u => u.id !== upgrade.id);
            
            // 移除其他升級門
            clearAllUpgrades();
            
            game.upgradeActive = false;
        }
        
        // 遊戲主循環
        function gameLoop(timestamp) {
            if (game.isGameOver) return;
            
            // 處理鍵盤輸入
            handleKeyboard();
            
            // 自動射擊
            fireBullet();
            
            // 生成敵人
            if (timestamp - game.lastEnemySpawn > game.enemySpawnRate && !game.bossActive) {
                spawnEnemy();
                game.lastEnemySpawn = timestamp;
                
                // 隨關卡提高生成率
                game.enemySpawnRate = Math.max(300, 1500 - game.level * 100);
            }
            
            // 生成升級門
            if (timestamp - game.lastUpgradeSpawn > game.upgradeSpawnRate && game.upgrades.length === 0 && !game.bossActive) {
                spawnUpgrade();
                game.lastUpgradeSpawn = timestamp;
            }
            
            // 更新敵人位置
            for (let i = game.enemies.length - 1; i >= 0; i--) {
                const enemy = game.enemies[i];
                
                // BOSS特殊移動模式
                if (enemy.isBoss) {
                    enemy.x += game.bossMoveDirection * game.bossMoveSpeed;
                    
                    // BOSS左右移動
                    if (enemy.x <= 0 || enemy.x >= gameContainer.clientWidth - 100) {
                        game.bossMoveDirection *= -1;
                    }
                    
                    // BOSS偶爾向下移動
                    if (Math.random() < 0.01 && enemy.y < gameContainer.clientHeight * 0.3) {
                        enemy.y += 20;
                    }
                } else {
                    // 普通敵人直線向下
                    enemy.y += enemy.speed;
                }
                
                enemy.element.style.left = `${enemy.x}px`;
                enemy.element.style.top = `${enemy.y}px`;
                
                // 檢查敵人是否超出屏幕底部
                if (enemy.y > gameContainer.clientHeight) {
                    if (enemy.element.parentNode) {
                        enemy.element.parentNode.removeChild(enemy.element);
                    }
                    game.enemies.splice(i, 1);
                    
                    if (enemy.isBoss) {
                        game.bossActive = false;
                        bossHealthBar.style.display = 'none';
                    }
                    continue;
                }
                
                // 檢查敵人與玩家碰撞
                const playerRect = player.getBoundingClientRect();
                const enemyRect = enemy.element.getBoundingClientRect();
                
                if (!(
                    playerRect.right < enemyRect.left || 
                    playerRect.left > enemyRect.right || 
                    playerRect.bottom < enemyRect.top || 
                    playerRect.top > enemyRect.bottom
                )) {
                    // 玩家被擊中
                    const damage = enemy.isBoss ? 20 : 10;
                    game.playerHealth -= damage;
                    updateHUD();
                    
                    createExplosion(enemyRect.left + enemyRect.width / 2, enemyRect.top + enemyRect.height / 2);
                    createFloatingText(`-${damage}`, playerRect.left + playerRect.width / 2, playerRect.top, '#e74c3c');
                    
                    if (!enemy.isBoss) {
                        if (enemy.element.parentNode) {
                            enemy.element.parentNode.removeChild(enemy.element);
                        }
                        game.enemies.splice(i, 1);
                    }
                    
                    if (game.playerHealth <= 0) {
                        gameOver();
                    }
                }
            }
            
            // 更新子彈位置
            for (let i = game.bullets.length - 1; i >= 0; i--) {
                const bullet = game.bullets[i];
                
                // 直線向上移動
                bullet.y -= bullet.speed;
                bullet.element.style.top = `${bullet.y}px`;
                
                // 檢查子彈是否超出屏幕
                if (bullet.y < -15) {
                    if (bullet.element.parentNode) {
                        bullet.element.parentNode.removeChild(bullet.element);
                    }
                    game.bullets.splice(i, 1);
                    continue;
                }
                
                // 檢查子彈與敵人碰撞
                for (let j = game.enemies.length - 1; j >= 0; j--) {
                    const enemy = game.enemies[j];
                    
                    const bulletRect = bullet.element.getBoundingClientRect();
                    const enemyRect = enemy.element.getBoundingClientRect();
                    
                    if (!(
                        bulletRect.right < enemyRect.left || 
                        bulletRect.left > enemyRect.right || 
                        bulletRect.bottom < enemyRect.top || 
                        bulletRect.top > enemyRect.bottom
                    )) {
                        // 擊中敵人
                        enemy.health -= bullet.damage;
                        
                        if (enemy.health <= 0) {
                            createExplosion(enemyRect.left + enemyRect.width / 2, enemyRect.top + enemyRect.height / 2);
                            const points = enemy.isBoss ? 100 * game.level : 10 * game.level;
                            createFloatingText(`+${points}`, enemyRect.left + enemyRect.width / 2, enemyRect.top, '#2ecc71');
                            
                            if (enemy.element.parentNode) {
                                enemy.element.parentNode.removeChild(enemy.element);
                            }
                            game.enemies.splice(j, 1);
                            game.score += points;
                            
                            if (enemy.isBoss) {
                                game.bossActive = false;
                                bossHealthBar.style.display = 'none';
                                createFloatingText('BOSS 擊敗!', gameContainer.clientWidth / 2, 100, '#f39c12');
                            }
                        } else {
                            createFloatingText(`-${bullet.damage}`, enemyRect.left + enemyRect.width / 2, enemyRect.top);
                            
                            if (enemy.isBoss) {
                                game.bossHealth = enemy.health;
                                updateHUD();
                            }
                        }
                        
                        if (bullet.element.parentNode) {
                            bullet.element.parentNode.removeChild(bullet.element);
                        }
                        game.bullets.splice(i, 1);
                        updateHUD();
                        break;
                    }
                }
            }
            
            // 更新升級門位置
            for (let i = game.upgrades.length - 1; i >= 0; i--) {
                const upgrade = game.upgrades[i];
                upgrade.y += upgrade.speed;
                upgrade.element.style.top = `${upgrade.y}px`;
                
                // 檢查升級門是否超出屏幕底部
                if (upgrade.y > gameContainer.clientHeight) {
                    if (upgrade.element.parentNode) {
                        upgrade.element.parentNode.removeChild(upgrade.element);
                    }
                    game.upgrades.splice(i, 1);
                    continue;
                }
                
                // 檢查升級門與玩家碰撞
                const playerRect = player.getBoundingClientRect();
                const upgradeRect = upgrade.element.getBoundingClientRect();
                
                if (!(
                    playerRect.right < upgradeRect.left || 
                    playerRect.left > upgradeRect.right || 
                    playerRect.bottom < upgradeRect.top || 
                    playerRect.top > upgradeRect.bottom
                )) {
                    // 處理升級
                    processUpgrade(upgrade);
                    break;
                }
            }
            
            // 更新爆炸效果
            for (let i = game.explosions.length - 1; i >= 0; i--) {
                if (Date.now() - game.explosions[i].time > 500) {
                    if (game.explosions[i].element.parentNode) {
                        game.explosions[i].element.parentNode.removeChild(game.explosions[i].element);
                    }
                    game.explosions.splice(i, 1);
                }
            }
            
            // 更新浮動文字
            for (let i = game.floatingTexts.length - 1; i >= 0; i--) {
                if (Date.now() - game.floatingTexts[i].time > 1000) {
                    if (game.floatingTexts[i].element.parentNode) {
                        game.floatingTexts[i].element.parentNode.removeChild(game.floatingTexts[i].element);
                    }
                    game.floatingTexts.splice(i, 1);
                }
            }
            
            // 檢查升級關卡
            if (game.score >= game.level * 100 && !game.bossActive) {
                game.level++;
                game.playerHealth = Math.min(game.maxHealth, game.playerHealth + 30);
                createFloatingText(`關卡 ${game.level}!`, player.getBoundingClientRect().left + player.clientWidth / 2, 
                                 player.getBoundingClientRect().top, '#f39c12');
                updateHUD();
            }
            
            requestAnimationFrame(gameLoop);
        }
        
        // 遊戲結束
        function gameOver() {
            game.isGameOver = true;
            finalScoreDisplay.textContent = game.score;
            finalLevelDisplay.textContent = game.level;
            gameOverScreen.style.display = 'flex';
        }
        
        // 觸控事件處理
        gameContainer.addEventListener('touchstart', (e) => {
            e.preventDefault();
            game.touchX = e.touches[0].clientX;
            updatePlayerPositionFromTouch();
        });
        
        gameContainer.addEventListener('touchmove', (e) => {
            e.preventDefault();
            if (e.touches.length > 0) {
                game.touchX = e.touches[0].clientX;
                updatePlayerPositionFromTouch();
            }
        });
        
        gameContainer.addEventListener('touchend', (e) => {
            e.preventDefault();
            game.touchX = null;
        });
        
        // 根據觸控位置更新玩家位置
        function updatePlayerPositionFromTouch() {
            if (game.touchX !== null) {
                const containerWidth = gameContainer.clientWidth;
                game.playerX = (game.touchX / containerWidth) * 100;
                game.playerX = Math.max(0, Math.min(100, game.playerX));
                updatePlayerPosition();
            }
        }
        
        // 鍵盤事件處理
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') game.keys.left = true;
            if (e.key === 'ArrowRight') game.keys.right = true;
        });
        
        window.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft') game.keys.left = false;
            if (e.key === 'ArrowRight') game.keys.right = false;
        });
        
        restartBtn.addEventListener('click', () => {
            initGame();
            requestAnimationFrame(gameLoop);
        });
        
        // 開始遊戲
        initGame();
        requestAnimationFrame(gameLoop);
    </script>
</body>
</html>
