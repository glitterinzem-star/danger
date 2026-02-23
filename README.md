<!DOCTYPE html>
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
            background-color: #1e1e1e; /* Темно-серый фон */
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }

        /* Главный контейнер */
        .card {
            max-width: 700px;
            width: 100%;
            background-color: #2d2d2d; /* Серый фон карточки */
            border-radius: 32px;
            padding: 2rem 2rem 2.5rem 2rem;
            box-shadow: 0 15px 30px rgba(0, 0, 0, 0.5);
            border: 1px solid #3a3a3a;
        }

        /* Шапка с кнопкой профиля */
        .header {
            display: flex;
            justify-content: flex-end;
            margin-bottom: 2rem;
        }

        .profile-button {
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            background-color: #3a3a3a;
            color: #e0e0e0;
            text-decoration: none;
            font-weight: 500;
            padding: 0.6rem 1.4rem;
            border-radius: 40px;
            font-size: 1rem;
            border: 1px solid #4f4f4f;
            transition: background-color 0.2s ease, border-color 0.2s ease;
        }

        .profile-button:hover {
            background-color: #4a4a4a;
            border-color: #6a6a6a;
            color: #ffffff;
        }

        /* Иконка (простой эмодзи, можно заменить на SVG) */
        .profile-icon {
            font-size: 1.2rem;
        }

        /* Заголовок */
        .title {
            color: #ffffff;
            font-size: 2rem;
            font-weight: 600;
            letter-spacing: -0.02em;
            margin-bottom: 0.5rem;
            border-left: 4px solid #777;
            padding-left: 1rem;
        }

        .subtitle {
            color: #b0b0b0;
            margin-bottom: 2.5rem;
            font-size: 1rem;
            border-left: 4px solid #777;
            padding-left: 1rem;
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
            background-color: #252525;
            border-radius: 20px;
            padding: 1.2rem 1.5rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border: 1px solid #3d3d3d;
            transition: transform 0.1s ease, border-color 0.2s ease;
        }

        .item:hover {
            border-color: #6b6b6b;
            transform: scale(1.01);
            background-color: #2a2a2a;
        }

        .item-left {
            display: flex;
            align-items: baseline;
            gap: 0.8rem;
        }

        .item-name {
            color: #ffffff;
            font-size: 1.3rem;
            font-weight: 500;
        }

        .item-badge {
            background-color: #3b3b3b;
            color: #c0c0c0;
            font-size: 0.8rem;
            padding: 0.2rem 0.8rem;
            border-radius: 30px;
            border: 1px solid #555;
        }

        .item-price {
            color: #d4d4d4;
            font-weight: 600;
            font-size: 1.3rem;
            background-color: #1f1f1f;
            padding: 0.4rem 1rem;
            border-radius: 40px;
            border: 1px solid #4a4a4a;
        }

        .stars-emoji {
            color: #ffd966;
            margin-right: 0.2rem;
            font-size: 1.1rem;
        }

        /* Дополнительная информация */
        .note {
            text-align: center;
            color: #8f8f8f;
            font-size: 0.9rem;
            margin-top: 2rem;
            padding-top: 1rem;
            border-top: 1px dashed #404040;
        }

        /* Адаптация для мобильных */
        @media (max-width: 480px) {
            .card {
                padding: 1.5rem;
            }
            .item {
                flex-direction: column;
                align-items: flex-start;
                gap: 0.8rem;
            }
            .item-price {
                align-self: flex-end;
            }
            .title {
                font-size: 1.6rem;
            }
        }

        /* Убираем всё лишнее, только суть */
        .no-extra {
            /* Класс для демонстрации отсутствия лишних элементов */
        }
    </style>
</head>
<body>
    <div class="card">
        <!-- Кнопка профиля с ссылкой на Telegram -->
        <div class="header">
            <a href="https://t.me/anon_t_me" target="_blank" rel="noopener noreferrer" class="profile-button">
                <span class="profile-icon">👤</span>
                Профиль в Telegram
            </a>
        </div>

        <!-- Заголовок -->
        <h1 class="title">Премиум стикеры</h1>
        <div class="subtitle">Цифровой товар • Telegram Stars</div>

        <!-- Сам прайс-лист -->
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

        <!-- Небольшой текст (чисто для информации, но можно убрать если совсем "без лишних добавлений") 
             Я оставил минимум, но если хотите полностью пустой низ - удалите этот блок -->
        <div class="note">
            * Для заказа пишите в Telegram
        </div>
        <!-- Абсолютно ничего лишнего: нет счетчиков, нет картинок, нет выпадающих меню -->
    </div>

    <!-- Фон сайта уже серый (body), внутри только необходимые элементы -->
</body>
</html>