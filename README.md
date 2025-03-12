# snake-game1
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>宇宙暗黑科技风 - 贪吃蛇游戏</title>
    <style>
        /* 全局样式 */
        body {
            margin: 0;
            padding: 0;
            font-family: 'Arial', sans-serif;
            background-color: #0a0a0a;
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            position: relative;
        }

        /* 粒子背景容器 */
        #particles-js {
            position: absolute;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
            z-index: 1;
        }

        /* 内容容器 */
        .container {
            text-align: center;
            z-index: 2;
            position: relative;
        }

        /* 标题样式 */
        h1 {
            font-size: 4rem;
            margin-bottom: 20px;
            background: linear-gradient(45deg, #00ffcc, #0077ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            animation: glow 2s infinite alternate;
        }

        /* 段落样式 */
        p {
            font-size: 1.2rem;
            color: #cccccc;
            margin-bottom: 30px;
        }

        /* 按钮样式 */
        .btn {
            background: linear-gradient(45deg, #00ffcc, #0077ff);
            border: none;
            padding: 15px 30px;
            font-size: 1rem;
            color: #0a0a0a;
            border-radius: 5px;
            cursor: pointer;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .btn:hover {
            transform: scale(1.1);
            box-shadow: 0 0 20px rgba(0, 255, 204, 0.7);
        }

        /* 贪吃蛇游戏画布 */
        #snakeCanvas {
            background-color: rgba(0, 0, 0, 0.5); /* 半透明背景 */
            border: 2px solid rgba(0, 255, 204, 0.5); /* 半透明边框 */
            position: relative;
            z-index: 3;
        }

        /* 标题发光动画 */
        @keyframes glow {
            0% { text-shadow: 0 0 10px #00ffcc, 0 0 20px #0077ff; }
            100% { text-shadow: 0 0 20px #00ffcc, 0 0 40px #0077ff; }
        }

        /* 爱心样式 */
        .heart {
            position: absolute;
            font-size: 24px;
            color: #ff0077;
            animation: float 1s ease-out forwards;
            pointer-events: none;
        }

        /* 爱心浮动动画 */
        @keyframes float {
            0% { transform: translateY(0) scale(1); opacity: 1; }
            100% { transform: translateY(-100px) scale(0.5); opacity: 0; }
        }
    </style>
</head>
<body>
<!-- 粒子背景 -->
<div id="particles-js"></div>

<!-- 内容区域 -->
<div class="container">
    <h1>暗黑科技</h1>
    <p>这是一个充满无聊至极的游戏。</p >
    <p>于2025年3月12日生产，贪吃蛇</p >
    <button class="btn">搜索更多</button>
</div>

<!-- 贪吃蛇游戏画布 -->
<canvas id="snakeCanvas" width="400" height="400"></canvas>

<!-- 引入粒子动画库 -->
<script src="https://cdn.jsdelivr.net/particles.js/2.0.0/particles.min.js"></script>
<script>
    // 初始化粒子背景
    particlesJS('particles-js', {
        particles: {
            number: { value: 80 },
            color: { value: '#00ffcc' },
            shape: { type: 'circle' },
            opacity: { value: 0.5, random: true },
            size: { value: 3, random: true },
            line_linked: { enable: true, distance: 150, color: '#0077ff', opacity: 0.4, width: 1 },
            move: { enable: true, speed: 3 }
        },
        interactivity: {
            detect_on: 'canvas',
            events: {
                onhover: { enable: true, mode: 'repulse' },
                onclick: { enable: true, mode: 'push' }
            }
        }
    });

    // 点击页面出现爱心
    document.addEventListener('click', function (event) {
        const heart = document.createElement('div');
        heart.classList.add('heart');
        heart.innerHTML = '❤️';
        heart.style.left = `${event.clientX}px`;
        heart.style.top = `${event.clientY}px`;
        document.body.appendChild(heart);

        // 动画结束后移除爱心
        setTimeout(() => {
            heart.remove();
        }, 1000);
    });

    // 贪吃蛇游戏逻辑
    const snakeCanvas = document.getElementById('snakeCanvas');
    const ctx = snakeCanvas.getContext('2d');

    const gridSize = 20; // 网格大小
    const tileCount = snakeCanvas.width / gridSize; // 每行/列的格子数

    let snake = [{ x: 10, y: 10 }]; // 蛇的初始位置
    let direction = { x: 0, y: 0 }; // 蛇的移动方向
    let food = { x: 5, y: 5 }; // 食物的初始位置
    let score = 0;

    // 绘制游戏元素
    function drawGame() {
        // 清空画布
        ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
        ctx.fillRect(0, 0, snakeCanvas.width, snakeCanvas.height);

        // 绘制蛇
        ctx.fillStyle = '#00ffcc';
        snake.forEach(segment => {
            ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
        });

        // 绘制食物
        ctx.fillStyle = '#ff0077';
        ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize, gridSize);

        // 显示分数
        ctx.fillStyle = '#fff';
        ctx.font = '16px Arial';
        ctx.fillText(`分数: ${score}`, 10, 20);
    }

    // 更新游戏状态
    function updateGame() {
        // 移动蛇
        const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

        // 检测是否吃到食物
        if (head.x === food.x && head.y === food.y) {
            score++;
            food = { x: Math.floor(Math.random() * tileCount), y: Math.floor(Math.random() * tileCount) };
        } else {
            snake.pop(); // 如果没有吃到食物，移除蛇尾
        }

        // 添加新头部
        snake.unshift(head);

        // 检测是否撞墙或撞到自己
        if (
            head.x < 0 || head.x >= tileCount ||
            head.y < 0 || head.y >= tileCount ||
            snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)
        ) {
            alert(`屌毛你输了，分数是：${score}`);
            resetGame();
        }
    }

    // 重置游戏
    function resetGame() {
        snake = [{ x: 10, y: 10 }];
        direction = { x: 0, y: 0 };
        food = { x: 5, y: 5 };
        score = 0;
    }

    // 监听键盘事件
    document.addEventListener('keydown', e => {
        switch (e.key) {
            case 'ArrowUp':
                if (direction.y === 0) direction = { x: 0, y: -1 };
                break;
            case 'ArrowDown':
                if (direction.y === 0) direction = { x: 0, y: 1 };
                break;
            case 'ArrowLeft':
                if (direction.x === 0) direction = { x: -1, y: 0 };
                break;
            case 'ArrowRight':
                if (direction.x === 0) direction = { x: 1, y: 0 };
                break;
        }
    });

    // 游戏循环
    function gameLoop() {
        updateGame();
        drawGame();
        setTimeout(gameLoop, 200); // 调整速度：200ms更新一次
    }

    // 启动游戏
    gameLoop();
</script>
</body>
</html>
