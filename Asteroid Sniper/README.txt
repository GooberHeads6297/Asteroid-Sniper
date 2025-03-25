INSTRUCTIONS!!!!

1. Use A and D to move from side to side.

2. Use the M key to shoot.

3.THAT'S IT JUST PLAY THE GAME NOWWWWWWWW!!!!

reverted code: 

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const backgroundMusic = document.getElementById('backgroundMusic');
const menuMusic = document.getElementById('menuMusic');
const mainMenu = document.getElementById('mainMenu');
const startButton = document.getElementById('startButton');
const scoreElement = document.getElementById('score');
const highscoreElement = document.getElementById('highscore');

let highscore = localStorage.getItem('highscore') || 0;
highscoreElement.textContent = highscore;

function resizeCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    menuMusic.play();
}

window.addEventListener('resize', resizeCanvas);
resizeCanvas();

let player, bullets, enemies, enemySpeed, score, gameOver, paused, particles, backgroundCircles;

function initializeGame() {
    player = {
        x: canvas.width / 2,
        y: canvas.height - 80, // Moved the player up for a flying effect
        width: 50,
        height: 50,
        color: 'blue',
        speed: 10,
        dx: 0
    };

    bullets = [];
    enemies = [];
    particles = [];
    backgroundCircles = [];
    enemySpeed = 2;
    score = 0;
    gameOver = false;
    paused = false;

    for (let i = 0; i < 100; i++) {
        backgroundCircles.push({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            radius: Math.random() * 2 + 1,
            speed: Math.random() * 0.5 + 0.5
        });
    }
}

function drawPlayer() {
    ctx.fillStyle = player.color;
    ctx.fillRect(player.x, player.y, player.width, player.height);
}

function drawBullets() {
    ctx.fillStyle = 'red';
    bullets.forEach(bullet => {
        ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
        bullet.y -= bullet.speed;
    });
}

function drawEnemies() {
    ctx.fillStyle = 'gray';
    enemies.forEach(enemy => {
        ctx.beginPath();
        ctx.arc(enemy.x, enemy.y, enemy.radius, 0, Math.PI * 2);
        ctx.fill();
        enemy.y += enemySpeed;
    });
}

function drawParticles() {
    particles.forEach((particle, index) => {
        ctx.fillStyle = 'gray';
        ctx.beginPath();
        ctx.arc(particle.x, particle.y, particle.radius, 0, Math.PI * 2);
        ctx.fill();
        particle.x += particle.dx;
        particle.y += particle.dy;
        particle.life -= 1;
        if (particle.life <= 0) {
            particles.splice(index, 1);
        }
    });
}

function drawBackgroundCircles() {
    ctx.fillStyle = 'white';
    backgroundCircles.forEach(circle => {
        ctx.beginPath();
        ctx.arc(circle.x, circle.y, circle.radius, 0, Math.PI * 2);
        ctx.fill();
        circle.y += circle.speed;
        if (circle.y > canvas.height) {
            circle.y = 0;
            circle.x = Math.random() * canvas.width;
        }
    });
}

function drawScore() {
    ctx.fillStyle = 'white';
    ctx.font = '20px Arial';
    ctx.fillText('Score: ' + score, 10, 20);
}

function update() {
    if (gameOver || paused) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawBackgroundCircles();
    drawPlayer();
    drawBullets();
    drawEnemies();
    drawParticles();
    drawScore();
    checkCollisions();
    player.x += player.dx;

    // Invisible barrier: Prevent player from going off the canvas
    if (player.x < 0) {
        player.x = 0; // Prevent moving left off the canvas
    } else if (player.x + player.width > canvas.width) {
        player.x = canvas.width - player.width; // Prevent moving right off the canvas
    }

    requestAnimationFrame(update);
}

document.addEventListener('keydown', (e) => {
    if (e.key === 'a') {
        player.dx = -player.speed;
    } else if (e.key === 'd') {
        player.dx = player.speed;
    } else if (e.key === ' ') { // Spacebar to shoot
        bullets.push({ 
            x: player.x + player.width / 2 - 2.5, 
            y: player.y, 
            width: 5, 
            height: 10, 
            speed: 15 
        });
    } else if (e.key === 'Escape') {
        paused = !paused;
        if (!paused) {
            update();
        }
    }
});

document.addEventListener('keyup', (e) => {
    if (e.key === 'a' || e.key === 'd') {
        player.dx = 0;
    }
});

function spawnEnemies() {
    setInterval(() => {
        if (!gameOver && !paused) {
            let x = Math.random() * (canvas.width - 50);
            enemies.push({ x: x, y: 0, radius: 25 });
        }
    }, 1000);
}

function createParticles(x, y) {
    for (let i = 0; i < 5; i++) { // Changed to 5 particles
        particles.push({
            x: x,
            y: y,
            radius: 5,
            dx: (Math.random() - 0.5) * 2,
            dy: (Math.random() - 0.5) * 2,
            life: 50
        });
    }
}

function checkCollisions() {
    bullets.forEach((bullet, bulletIndex) => {
        enemies.forEach((enemy, enemyIndex) => {
            let dist = Math.hypot(bullet.x - enemy.x, bullet.y - enemy.y);
            if (dist < enemy.radius) {
                createParticles(enemy.x, enemy.y);
                enemies.splice(enemyIndex, 1);
                bullets.splice(bulletIndex, 1);
                score++;
            }
        });
    });

    enemies.forEach((enemy, enemyIndex) => {
        let dist = Math.hypot(player.x + player.width / 2 - enemy.x, player.y + player.height / 2 - enemy.y);
        if (dist < enemy.radius + player.width / 2) {
            gameOver = true;
            alert('Game Over');
            if (score > highscore) {
                highscore = score;
                localStorage.setItem('highscore', highscore);
                highscoreElement.textContent = highscore;
            }
            backgroundMusic.pause();
            menuMusic.play();
            mainMenu.style.display = 'flex';
            canvas.style.display = 'none';
        }
    });
}

startButton.addEventListener('click', () => {
    mainMenu.style.display = 'none';
    canvas.style.display = 'block';
    menuMusic.pause();
    backgroundMusic.play();
    initializeGame();
    update();
    spawnEnemies();
});
