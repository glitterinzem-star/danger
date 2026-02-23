<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Прайс-лист | Премиум стикеры</title>
    <style>
        /* Сброс отступов и базовые настройки */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
            background: linear-gradient(145deg, #1b8f9e 0%, #1565C0 100%); /* Голубой градиент */
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 1rem;
            position: relative;
            overflow-x: hidden;
        }

        /* Анимированные волны на фоне */
        body::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: radial-gradient(circle at 20% 50%, rgba(255,255,255,0.1) 0%, transparent 50%);
            pointer-events: none;
        }

        body::after {
            content: '';
            position: absolute;
            width: 200%;
            height: 200%;
            top: -50%;
            left: -50%;
            background: radial-gradient(circle, rgba(255,255,255,0.15) 0%, transparent 30%);
            animation: wave 20s infinite linear;
            pointer-events: none;
        }

        @keyframes wave {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Главный контейнер */
        .card {
            max-width: 700px;
            width: 100%;
            background: rgba(255, 255, 255, 0.1); /* Полупрозрачный фон */
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border-radius: 32px;
            padding: 2rem 2rem 2.5rem 2rem;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3),
                       0 0 0 1px rgba(255, 255, 255, 0.2) inset;
            border: 1px solid rgba(255, 255, 255, 0.3);
            position: relative;
            z-index: 2;
            animation: float 6s ease-in-out infinite;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-10px); }
        }

        /* Шапка с кнопкой профиля - центрируем */
        .header {
            display: flex;
            justify-content: center;
            margin-bottom: 2.5rem;
        }

        .profile-button {
            display: inline-flex;
            align-items: center;
            gap: 0.7rem;
            background: rgba(255, 255, 255, 0.2);
            color: white;
            text-decoration: none;
            font-weight: 600;
            padding: 0.8rem 2rem;
            border-radius: 60px;
            font-size: 1.1rem;
            border: 1px solid rgba(255, 255, 255, 0.4);
            transition: all 0.3s ease;
            backdrop-filter: blur(5px);
            -webkit-backdrop-filter: blur(5px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
            letter-spacing: 0.3px;
            position: relative;
            overflow: hidden;
        }

        .profile-button::before {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 0;
            height: 0;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.3);
            transform: translate(-50%, -50%);
            transition: width 0.6s, height 0.6s;
        }

        .profile-button:hover::before {
            width: 300px;
            height: 300px;
        }

        .profile-button:hover {
            background: rgba(255, 255, 255, 0.3);
            border-color: rgba(255, 255, 255, 0.8);
            transform: scale(1.05);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
        }

        .profile-button:active {
            transform: scale(0.98);
        }

        /* Иконка */
        .profile-icon {
            font-size: 1.3rem;
            filter: drop-shadow(0 2px 2px rgba(0,0,0,0.2));
            position: relative;
            z-index: 2;
        }

        /* Заголовок */
        .title {
            color: #ffffff;
            font-size: 2rem;
            font-weight: 700;
            letter-spacing: -0.02em;
            margin-bottom: 0.5rem;
            border-left: 4px solid rgba(255, 255, 255, 0.7);
            padding-left: 1rem;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .subtitle {
            color: rgba(255, 255, 255, 0.9);
            margin-bottom: 2.5rem;
            font-size: 1rem;
            border-left: 4px solid rgba(255, 255, 255, 0.5);
            padding-left: 1rem;
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
        }

        /* Сетка прайс-листа */
        .pricelist {
            display: flex;
            flex-direction: column;
            gap: 1rem;
            margin-bottom: 2rem;
        }

        /* Карточка товара */
        .item {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(8px);
            -webkit-backdrop-filter: blur(8px);
            border-radius: 20px;
            padding: 1.2rem 1.5rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border: 1px solid rgba(255, 255, 255, 0.3);
            transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            position: relative;
            overflow: hidden;
        }

        /* Эффект сияния при наведении */
        .item::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(
                90deg,
                transparent,
                rgba(255, 255, 255, 0.2),
                transparent
            );
            transition: left 0.7s;
        }

        .item:hover::before {
            left: 100%;
        }

        .item:hover {
            border-color: rgba(255, 255, 255, 0.8);
            transform: scale(1.02) translateY(-3px);
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.3);
            background: rgba(255, 255, 255, 0.15);
        }

        .item-left {
            display: flex;
            align-items: baseline;
            gap: 0.8rem;
            position: relative;
            z-index: 2;
        }

        .item-name {
            color: #ffffff;
            font-size: 1.3rem;
            font-weight: 600;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.2);
        }

        .item-badge {
            background: rgba(255, 255, 255, 0.25);
            color: white;
            font-size: 0.8rem;
            padding: 0.2rem 0.8rem;
            border-radius: 30px;
            border: 1px solid rgba(255, 255, 255, 0.4);
            backdrop-filter: blur(4px);
            font-weight: 500;
        }

        .item-price {
            color: #ffffff;
            font-weight: 700;
            font-size: 1.3rem;
            background: rgba(0, 0, 0, 0.25);
            padding: 0.4rem 1rem;
            border-radius: 40px;
            border: 1px solid rgba(255, 255, 255, 0.4);
            backdrop-filter: blur(4px);
            display: flex;
            align-items: center;
            gap: 4px;
            position: relative;
            z-index: 2;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
            transition: all 0.3s ease;
        }

        .item:hover .item-price {
            background: rgba(0, 0, 0, 0.35);
            border-color: rgba(255, 255, 255, 0.8);
            transform: scale(1.05);
        }

        .stars-emoji {
            color: #ffeb3b;
            margin-right: 0.2rem;
            font-size: 1.2rem;
            filter: drop-shadow(0 0 5px rgba(255, 235, 59, 0.5));
        }

        /* Дополнительная информация */
        .note {
            text-align: center;
            color: rgba(255, 255, 255, 0.9);
            font-size: 0.95rem;
            margin-top: 2rem;
            padding-top: 1.2rem;
            border-top: 1px dashed rgba(255, 255, 255, 0.4);
            font-weight: 400;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
            letter-spacing: 0.3px;
        }

        /* Адаптация для мобильных */
        @media (max-width: 480px) {
            .card {
                padding: 1.5rem;
            }
            .item {
                flex-direction: column;
                align-items: flex-start;
                gap: 1rem;
            }
            .item-price {
                align-self: flex-end;
            }
            .title {
                font-size: 1.6rem;
            }
            .profile-button {
                padding: 0.7rem 1.5rem;
                font-size: 1rem;
            }
        }
    </style>
</head>
<body>
    <div class="card">
        <!-- Кнопка профиля в центре -->
        <div class="header">
            <a href="https://t.me/anon_t_me" target="_blank" rel="noopener noreferrer" class="profile-button">
                <span class="profile-icon">👤</span>
                Профиль в Telegram
            </a>
        </div>

        <!-- Заголовок -->
        <h1 class="title">Премиум стикеры</h1>
        <div class="subtitle">Цифровой товар • Telegram Stars</div>

        <!-- Прайс-лист -->
        <div class="pricelist">
            <!-- 1 штука -->
            <div class="item">
                <div class="item-left">
                    <span class="item-name">1 стикер</span>
                    <span class="item-badge">набор</span>
                </div>
                <div class="item-price">
                    <span class="stars-emoji">⭐</span>25 звёзд
                </div>
            </div>

            <!-- 5 штук -->
            <div class="item">
                <div class="item-left">
                    <span class="item-name">5 стикеров</span>
                    <span class="item-badge">экономия 30⭐</span>
                </div>
                <div class="item-price">
                    <span class="stars-emoji">⭐</span>95 звёзд
                </div>
            </div>

            <!-- 10 штук -->
            <div class="item">
                <div class="item-left">
                    <span class="item-name">10 стикеров</span>
                    <span class="item-badge">пак</span>
                </div>
                <div class="item-price">
                    <span class="stars-emoji">⭐</span>150 звёзд
                </div>
            </div>
        </div>

        <!-- Информационная заметка -->
        <div class="note">
            ✦ Для заказа пишите в Telegram ✦
        </div>
    </div>
</body>
</html>