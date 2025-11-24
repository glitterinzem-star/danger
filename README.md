<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>министров</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background-color: #000;
            color: #00ffff;
            font-family: 'Courier New', monospace;
            overflow-x: hidden;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            padding: 20px;
        }

        .container {
            text-align: center;
            z-index: 10;
            position: relative;
            width: 100%;
            max-width: 800px;
            padding: 0 15px;
        }

        .neon-text {
            font-size: 5rem;
            font-weight: bold;
            color: #00ffff;
            text-shadow:
                0 0 10px #00ffff,
                0 0 20px #00ffff,
                0 0 30px #00ffff,
                0 0 40px #0080ff,
                0 0 70px #0080ff,
                0 0 80px #0080ff;
            margin-bottom: 2rem;
            letter-spacing: 5px;
            /* Упрощенная анимация для лучшей производительности */
            animation: neon-flicker 4s infinite alternate;
            will-change: text-shadow;
        }

        @keyframes neon-flicker {
            0%, 18%, 22%, 25%, 53%, 57%, 100% {
                text-shadow: 
                    0 0 10px #00ffff,
                    0 0 20px #00ffff,
                    0 0 30px #00ffff,
                    0 0 40px #0080ff,
                    0 0 70px #0080ff,
                    0 0 80px #0080ff;
            }
            20%, 24%, 55% {
                text-shadow: 
                    0 0 5px #00ffff,
                    0 0 10px #00ffff,
                    0 0 15px #00ffff,
                    0 0 20px #0080ff,
                    0 0 35px #0080ff,
                    0 0 40px #0080ff;
            }
        }

        .info-section {
            background: rgba(0, 255, 255, 0.1);
            border: 1px solid #00ffff;
            padding: 2rem;
            margin: 1rem 0;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.3);
            width: 100%;
            /* Улучшение производительности */
            transform: translateZ(0);
            backface-visibility: hidden;
        }

        .separator {
            height: 2px;
            background: linear-gradient(90deg, transparent, #00ffff, transparent);
            margin: 1.5rem 0;
        }

        .links {
            margin-top: 2rem;
            display: flex;
            flex-direction: column;
            gap: 1rem;
            align-items: center;
            width: 100%;
        }

        .link {
            color: #00ffff;
            text-decoration: none;
            padding: 0.8rem 2rem;
            border: 1px solid #00ffff;
            border-radius: 5px;
            transition: all 0.3s ease;
            text-shadow: 0 0 10px #00ffff;
            width: 250px;
            display: block;
            /* Улучшение производительности */
            transform: translateZ(0);
        }

        .link:hover {
            background: #00ffff;
            color: #000;
            box-shadow: 0 0 20px #00ffff;
            transform: scale(1.05);
        }

        .telegram-links {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            margin: 1rem 0;
        }

        .telegram-link {
            color: #00ffff;
            text-decoration: none;
            padding: 0.5rem 1rem;
            border: 1px solid #00ffff;
            border-radius: 3px;
            transition: all 0.3s ease;
            font-size: 0.9rem;
            /* Улучшение производительности */
            transform: translateZ(0);
        }

        .telegram-link:hover {
            background: #00ffff;
            color: #000;
            box-shadow: 0 0 10px #00ffff;
        }

        .owner-button {
            margin-top: 1.5rem;
            padding: 0.8rem 2rem;
            background: rgba(0, 255, 255, 0.1);
            border: 1px solid #00ffff;
            border-radius: 5px;
            color: #00ffff;
            text-decoration: none;
            transition: all 0.3s ease;
            display: inline-block;
            font-size: 1rem;
            margin: 0.5rem;
            /* Улучшение производительности */
            transform: translateZ(0);
        }

        .owner-button:hover {
            background: #00ffff;
            color: #000;
            box-shadow: 0 0 20px #00ffff;
            transform: scale(1.05);
        }

        .owner-link {
            color: #00ffff;
            text-decoration: none;
        }

        .owner-link:hover {
            text-decoration: underline;
        }

        .channel-text {
            font-size: 1.1em;
            font-weight: bold;
            margin-bottom: 0.3rem;
        }

        .chat-text {
            font-size: 1em;
            opacity: 0.9;
        }

        /* Упрощенный эффект глитча для лучшей производительности */
        .glitch-effect {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, transparent 49%, rgba(0, 255, 255, 0.05) 50%, transparent 51%);
            background-size: 20px 20px;
            animation: glitch 2s infinite;
            pointer-events: none;
            opacity: 0.05;
            /* Улучшение производительности */
            will-change: transform;
        }

        @keyframes glitch {
            0% { transform: translateX(0); }
            50% { transform: translateX(-5px); }
            100% { transform: translateX(0); }
        }

        /* Упрощенная сканирующая линия */
        .scan-line {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 2px;
            background: linear-gradient(90deg, transparent, #00ffff, transparent);
            animation: scan 4s linear infinite;
            box-shadow: 0 0 10px #00ffff;
            /* Улучшение производительности */
            will-change: top;
        }

        @keyframes scan {
            0% { top: 0; }
            100% { top: 100%; }
        }

        /* Медиа-запросы для адаптивности */
        @media (max-width: 768px) {
            .neon-text {
                font-size: 3.5rem;
                letter-spacing: 3px;
                margin-bottom: 1.5rem;
            }
            
            .info-section {
                padding: 1.5rem;
            }
            
            .link {
                width: 220px;
                padding: 0.7rem 1.5rem;
            }
            
            .owner-button {
                padding: 0.7rem 1.5rem;
            }
        }

        @media (max-width: 480px) {
            .neon-text {
                font-size: 2.5rem;
                letter-spacing: 2px;
                margin-bottom: 1rem;
            }
            
            .info-section {
                padding: 1rem;
            }
            
            .link {
                width: 200px;
                padding: 0.6rem 1.2rem;
                font-size: 0.9rem;
            }
            
            .owner-button {
                padding: 0.6rem 1.2rem;
                font-size: 0.9rem;
            }
            
            .telegram-link {
                font-size: 0.8rem;
            }
            
            .channel-text, .chat-text {
                font-size: 0.9rem;
            }
        }

        @media (max-width: 320px) {
            .neon-text {
                font-size: 2rem;
            }
            
            .link {
                width: 180px;
            }
        }
    </style>
</head>
<body>
    <div class="scan-line"></div>
    <div class="glitch-effect"></div>
    
    <div class="container">
        <div class="neon-text">danger place</div>
        
        <div class="info-section">
            <p>привет я министров</p>
            <div class="separator"></div>
            <div class="separator"></div>
            
            <div class="telegram-links">
                <a href="https://t.me/dangerplace01" class="telegram-link" target="_blank"> CHANNEL › danger place</a>
            </div>
            
            <div class="separator"></div>
            <p>я специалист в сфере осинт там соц инженер и занимаюсь активным поиском информации о человека по соц сетям и зацепкам</p>
        </div>

        <div class="links">
            <a href="https://t.me/dangerplace01" class="link" target="_blank">JOIN CHANNEL</a>
        </div>

        <div class="owner-buttons">
            <a href="http://t.me/oexess" class="owner-button" target="_blank">OWNER</a>
        </div>
    </div>

    <script>
        // Упрощенный JavaScript для лучшей производительности
        
        // Эффект ввода текста (выполняется только один раз)
        window.addEventListener('load', () => {
            const texts = document.querySelectorAll('.info-section p, .channel-text, .chat-text');
            texts.forEach((text, index) => {
                const originalText = text.textContent;
                text.textContent = '';
                let i = 0;
                
                setTimeout(() => {
                    const typeWriter = setInterval(() => {
                        if (i < originalText.length) {
                            text.textContent += originalText.charAt(i);
                            i++;
                        } else {
                            clearInterval(typeWriter);
                        }
                    }, 50);
                }, index * 500);
            });
        });

        // Упрощенный эффект мерцания с использованием requestAnimationFrame
        let lastTime = 0;
        function flickerNeon(currentTime) {
            if (currentTime - lastTime > 100) {
                const neon = document.querySelector('.neon-text');
                const intensity = 0.8 + Math.random() * 0.4;
                neon.style.textShadow = `
                    0 0 ${10 * intensity}px #00ffff,
                    0 0 ${20 * intensity}px #00ffff,
                    0 0 ${30 * intensity}px #00ffff,
                    0 0 ${40 * intensity}px #0080ff,
                    0 0 ${70 * intensity}px #0080ff,
                    0 0 ${80 * intensity}px #0080ff
                `;
                lastTime = currentTime;
            }
            requestAnimationFrame(flickerNeon);
        }
        requestAnimationFrame(flickerNeon);

        // Упрощенный эффект мерцания элементов с использованием requestAnimationFrame
        let lastFlickerTime = 0;
        function flickerElements(currentTime) {
            if (currentTime - lastFlickerTime > 300) {
                const elements = document.querySelectorAll('.link, .telegram-link, .owner-button');
                elements.forEach(element => {
                    if (Math.random() > 0.7) {
                        element.style.opacity = '0.7';
                        setTimeout(() => {
                            element.style.opacity = '1';
                        }, 100);
                    }
                });
                lastFlickerTime = currentTime;
            }
            requestAnimationFrame(flickerElements);
        }
        requestAnimationFrame(flickerElements);
    </script>
</body>
</html>