import logging
import sqlite3
import aiohttp
import asyncio
import random
import string
from datetime import datetime
from bs4 import BeautifulSoup
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup, ReplyKeyboardMarkup, KeyboardButton
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes

# ==================== КОНФИГУРАЦИЯ ====================
BOT_TOKEN = "8535069732:AAGO6RTIBNvnXDVyFgTBeWjuAsuPvoTZ-Hs"
ADMIN_ID = 8205783729
TON_WALLET = "UQDsR6A58NnYqihGpKdrkOEhV8o7dDa5IB12Sb_0D7BJCHkH"

MIN_STARS = 50
FRAGMENT_URL = "https://fragment.com/stars/buy"
FRAGMENT_PREMIUM_URL = "https://fragment.com/telegram-premium"

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

current_fragment_prices = {}
current_premium_prices = {}

# ==================== ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ====================
def generate_random_comment(username, stars=None, period=None):
    """
    Генерирует рандомный комментарий для оплаты
    Формат: случайные символы (буквы и цифры) до 12 символов
    """
    # Генерируем случайную строку из 12 символов (буквы + цифры)
    random_part = ''.join(random.choices(string.ascii_letters + string.digits, k=12))
    return random_part

# ==================== БАЗА ДАННЫХ ====================
def init_db():
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    
    c.execute('''CREATE TABLE IF NOT EXISTS users (
        user_id INTEGER PRIMARY KEY,
        username TEXT,
        total_stars INTEGER DEFAULT 0,
        total_spent REAL DEFAULT 0,
        total_orders INTEGER DEFAULT 0,
        reg_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        last_order_date TIMESTAMP
    )''')
    
    c.execute('''CREATE TABLE IF NOT EXISTS orders (
        order_id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER,
        username TEXT,
        stars INTEGER,
        fragment_price REAL,
        your_price REAL,
        status TEXT DEFAULT 'pending',
        payment_comment TEXT,
        order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        admin_notified INTEGER DEFAULT 0,
        FOREIGN KEY(user_id) REFERENCES users(user_id)
    )''')
    
    conn.commit()
    conn.close()

def add_user(user_id, username):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        "INSERT OR IGNORE INTO users (user_id, username) VALUES (?, ?)",
        (user_id, username)
    )
    conn.commit()
    conn.close()

def update_user_stats(user_id, stars, price):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        """UPDATE users 
           SET total_stars = total_stars + ?, 
               total_spent = total_spent + ?, 
               total_orders = total_orders + 1,
               last_order_date = CURRENT_TIMESTAMP 
           WHERE user_id = ?""",
        (stars, price, user_id)
    )
    conn.commit()
    conn.close()

def get_user_profile(user_id):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        "SELECT username, total_stars, total_spent, total_orders, reg_date FROM users WHERE user_id = ?",
        (user_id,)
    )
    result = c.fetchone()
    conn.close()
    return result

def create_order(user_id, username, stars, fragment_price, your_price, comment):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        """INSERT INTO orders 
           (user_id, username, stars, fragment_price, your_price, payment_comment) 
           VALUES (?, ?, ?, ?, ?, ?)""",
        (user_id, username, stars, fragment_price, your_price, comment)
    )
    order_id = c.lastrowid
    conn.commit()
    conn.close()
    return order_id

def get_orders_for_admin(status='pending'):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        """SELECT order_id, user_id, username, stars, your_price, payment_comment, order_date 
           FROM orders 
           WHERE status = ? AND admin_notified = 0
           ORDER BY order_date DESC""",
        (status,)
    )
    orders = c.fetchall()
    conn.close()
    return orders

def mark_order_as_notified(order_id):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        "UPDATE orders SET admin_notified = 1 WHERE order_id = ?",
        (order_id,)
    )
    conn.commit()
    conn.close()

def update_order_status(order_id, status):
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute(
        "UPDATE orders SET status = ? WHERE order_id = ?",
        (status, order_id)
    )
    conn.commit()
    conn.close()

# ==================== ЦЕНЫ ====================
def get_fallback_prices():
    # Резервные цены если не получится получить с Fragment
    return {
        50: 0.5,   # 50 звёзд = 0.5 TON
        100: 0.9   # 100 звёзд = 0.9 TON
    }

def get_fallback_premium_prices():
    # Резервные цены для Telegram Premium если не получится получить с Fragment
    return {
        "3_month": 7.3,
        "6_month": 9.65,
        "12_month": 17.4
    }

def calculate_price_for_stars(stars_count):
    global current_fragment_prices
    
    if stars_count < MIN_STARS:
        return None
    
    # Получаем базовую цену для 100 звёзд
    if current_fragment_prices and 100 in current_fragment_prices:
        base_100_price = current_fragment_prices[100]
    else:
        # Используем резервную цену для 100 звёзд
        base_100_price = get_fallback_prices()[100]
    
    # Получаем цену для 50 звёзд
    if current_fragment_prices and 50 in current_fragment_prices:
        base_50_price = current_fragment_prices[50]
    else:
        # Используем резервную цену для 50 звёзд
        base_50_price = get_fallback_prices()[50]
    
    # Рассчитываем цену в зависимости от количества звёзд
    if stars_count == 50:
        fragment_price = base_50_price
    elif stars_count == 100:
        fragment_price = base_100_price
    else:
        # Для других количеств: умножаем цену 100 звёзд на множитель
        multiplier = stars_count / 100
        fragment_price = round(base_100_price * multiplier, 3)
    
    # Ваша цена такая же как цена на фрагменте (без наценки)
    your_price = round(fragment_price, 3)
    
    return {
        'stars': stars_count,
        'fragment_price': round(fragment_price, 3),
        'your_price': your_price,
        'markup': 0  # Нет наценки
    }

def calculate_price_for_premium(period):
    global current_premium_prices
    
    if period == "3_month":
        if current_premium_prices and "3_month" in current_premium_prices:
            price = current_premium_prices["3_month"]
        else:
            price = get_fallback_premium_prices()["3_month"]
    elif period == "6_month":
        if current_premium_prices and "6_month" in current_premium_prices:
            price = current_premium_prices["6_month"]
        else:
            price = get_fallback_premium_prices()["6_month"]
    elif period == "12_month":
        if current_premium_prices and "12_month" in current_premium_prices:
            price = current_premium_prices["12_month"]
        else:
            price = get_fallback_premium_prices()["12_month"]
    else:
        return None
    
    return {
        'period': period,
        'price': price,
        'your_price': price,
        'markup': 0
    }

async def fetch_fragment_prices():
    global current_fragment_prices
    
    headers = {
        'User-Agent': 'Mozilla/5.0'
    }
    
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(FRAGMENT_URL, headers=headers, timeout=10) as response:
                if response.status == 200:
                    html = await response.text()
                    soup = BeautifulSoup(html, 'html.parser')
                    
                    price_table = soup.find('table')
                    if price_table:
                        prices = {}
                        rows = price_table.find_all('tr')
                        
                        for row in rows[1:]:
                            cols = row.find_all('td')
                            if len(cols) >= 2:
                                try:
                                    stars_text = cols[0].text.strip()
                                    price_text = cols[1].text.strip()
                                    
                                    stars = int(''.join(filter(str.isdigit, stars_text)))
                                    price_text_clean = price_text.replace('TON', '').strip()
                                    price = float(price_text_clean)
                                    
                                    prices[stars] = price
                                except:
                                    continue
                        
                        if prices:
                            current_fragment_prices = prices
                            return True
        
        # Если не удалось получить цены с Fragment, используем резервные
        current_fragment_prices = get_fallback_prices()
        return True
        
    except Exception as e:
        logger.error(f"Ошибка: {e}")
        # При ошибке используем резервные цены
        current_fragment_prices = get_fallback_prices()
        return True

async def fetch_premium_prices():
    global current_premium_prices
    
    headers = {
        'User-Agent': 'Mozilla/5.0'
    }
    
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(FRAGMENT_PREMIUM_URL, headers=headers, timeout=10) as response:
                if response.status == 200:
                    html = await response.text()
                    soup = BeautifulSoup(html, 'html.parser')
                    
                    # Пытаемся найти цены на премиум
                    # Это упрощенная логика, на практике нужно адаптировать под структуру fragment.com
                    
                    # Ищем элементы с ценами
                    price_elements = soup.find_all(class_=['tm-price', 'price'])
                    
                    if price_elements:
                        prices = {}
                        # Простая логика - берем первые 3 найденные цены
                        for i, elem in enumerate(price_elements[:3]):
                            try:
                                price_text = elem.text.strip().replace('TON', '').strip()
                                price = float(price_text)
                                
                                if i == 0:
                                    prices["3_month"] = price
                                elif i == 1:
                                    prices["6_month"] = price
                                elif i == 2:
                                    prices["12_month"] = price
                            except:
                                continue
                        
                        if prices:
                            current_premium_prices = prices
                            return True
        
        # Если не удалось получить цены с Fragment, используем резервные
        current_premium_prices = get_fallback_premium_prices()
        return True
        
    except Exception as e:
        logger.error(f"Ошибка при получении цен на премиум: {e}")
        # При ошибке используем резервные цены
        current_premium_prices = get_fallback_premium_prices()
        return True

# ==================== КЛАВИАТУРЫ ====================
def get_main_keyboard():
    keyboard = [
        [KeyboardButton("⭐ Купить звёзды"), KeyboardButton("👑 Телеграмм премиум")]
    ]
    return ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

def get_stars_keyboard():
    keyboard = [
        [InlineKeyboardButton("50 звёзд", callback_data="stars_50")],
        [InlineKeyboardButton("100 звёзд", callback_data="stars_100")],
        [InlineKeyboardButton("200 звёзд", callback_data="stars_200")],
        [InlineKeyboardButton("500 звёзд", callback_data="stars_500")],
        [InlineKeyboardButton("1000 звёзд", callback_data="stars_1000")],
        [InlineKeyboardButton("Другое количество", callback_data="stars_custom")],
        [InlineKeyboardButton("🔄 Обновить цены", callback_data="refresh_prices")]
    ]
    return InlineKeyboardMarkup(keyboard)

def get_premium_keyboard():
    keyboard = [
        [InlineKeyboardButton("3 месяца", callback_data="premium_3_month")],
        [InlineKeyboardButton("6 месяцев", callback_data="premium_6_month")],
        [InlineKeyboardButton("1 год", callback_data="premium_12_month")],
        [InlineKeyboardButton("🔄 Обновить цены", callback_data="refresh_premium_prices")]
    ]
    return InlineKeyboardMarkup(keyboard)

def get_admin_keyboard(order_id):
    keyboard = [
        [
            InlineKeyboardButton("✅ Выполнено", callback_data=f"admin_complete_{order_id}"),
            InlineKeyboardButton("❌ Отменить", callback_data=f"admin_cancel_{order_id}")
        ]
    ]
    return InlineKeyboardMarkup(keyboard)

# ==================== КОМАНДЫ ====================
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    user_id = user.id
    username = user.username or f"user_{user_id}"
    
    add_user(user_id, username)
    
    welcome_text = (
        "🤖 Бот для покупки Telegram Stars\n\n"
        "✨ Самые дешёвые звёзды в Telegram!\n"
        "🎯 Минимальная покупка: 50 звезд\n\n"
        "Выберите действие:"
    )
    
    await update.message.reply_text(
        welcome_text,
        reply_markup=get_main_keyboard()
    )

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    user_id = update.effective_user.id
    
    if text == "⭐ Купить звёзды":
        await show_stars_menu(update, context)
    elif text == "👑 Телеграмм премиум":
        await show_premium_menu(update, context)
    elif text.isdigit():
        stars = int(text)
        await process_stars_selection(update, context, stars)
    else:
        await update.message.reply_text(
            "Используйте кнопки",
            reply_markup=get_main_keyboard()
        )

async def show_stars_menu(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await fetch_fragment_prices()
    
    menu_text = (
        f"Выберите количество звёзд:\n\n"
        f"Минимум: 50 звёзд\n\n"
        f"Или введите своё количество:"
    )
    
    if update.message:
        await update.message.reply_text(
            menu_text,
            reply_markup=get_stars_keyboard()
        )
    else:
        await update.callback_query.message.reply_text(
            menu_text,
            reply_markup=get_stars_keyboard()
        )

async def show_premium_menu(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await fetch_premium_prices()
    
    # Получаем текущие цены
    price_3month = calculate_price_for_premium("3_month")
    price_6month = calculate_price_for_premium("6_month")
    price_12month = calculate_price_for_premium("12_month")
    
    menu_text = (
        f"👑 Telegram Premium\n\n"
        f"Выберите срок подписки:\n\n"
    )
    
    if update.message:
        await update.message.reply_text(
            menu_text,
            reply_markup=get_premium_keyboard()
        )
    else:
        await update.callback_query.message.reply_text(
            menu_text,
            reply_markup=get_premium_keyboard()
        )

async def handle_callback_query(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    
    data = query.data
    user_id = query.from_user.id
    
    if data.startswith("stars_"):
        if data == "stars_custom":
            await query.message.reply_text(
                "Введите количество звёзд:\nМинимум: 50 звёзд"
            )
        elif data == "refresh_prices":
            await query.message.reply_text("Обновляю цены...")
            await fetch_fragment_prices()
            await show_stars_menu(update, context)
        else:
            stars = int(data.split("_")[1])
            await process_stars_selection(update, context, stars)
    
    elif data.startswith("premium_"):
        if data == "refresh_premium_prices":
            await query.message.reply_text("Обновляю цены на премиум...")
            await fetch_premium_prices()
            await show_premium_menu(update, context)
        else:
            period = data.split("_")[1] + "_" + data.split("_")[2]
            await process_premium_selection(update, context, period)
    
    elif data.startswith("admin_"):
        await handle_admin_action(update, context, data)

async def process_stars_selection(update: Update, context: ContextTypes.DEFAULT_TYPE, stars: int):
    if stars < MIN_STARS:
        if update.callback_query:
            await update.callback_query.message.reply_text(
                f"Минимальная покупка {MIN_STARS} звёзд",
                reply_markup=get_stars_keyboard()
            )
        else:
            await update.message.reply_text(
                f"Минимальная покупка {MIN_STARS} звёзд",
                reply_markup=get_stars_keyboard()
            )
        return
    
    user = update.effective_user
    user_id = user.id
    username = user.username or f"user_{user_id}"
    
    price_data = calculate_price_for_stars(stars)
    if not price_data:
        return
    
    # Генерируем рандомный комментарий для оплаты
    payment_comment = generate_random_comment(username, stars=stars)
    
    order_id = create_order(
        user_id, username, stars,
        price_data['fragment_price'],
        price_data['your_price'],
        payment_comment
    )
    
    # Форматируем текст с отдельными блоками для копирования
    payment_text = (
        f"Заказ #{order_id} создан!\n\n"
        f"Покупатель: @{username}\n"
        f"Звёзд: {stars}\n"
        f"Сумма: {price_data['your_price']} TON\n\n"
        f"Для оплаты отправьте точную сумму:\n"
    )
    
    # Блок с суммой для копирования (с фиксированным шрифтом)
    amount_block = (
        f"<code>{price_data['your_price']} TON</code>"
    )
    
    # Блок с адресом кошелька для копирования
    wallet_block = (
        f"\nНа адрес:\n"
        f"<code>{TON_WALLET}</code>\n\n"
    )
    
    comment_block = (
        f"КОММЕНТАРИЙ (Memo):\n"
        f"<code>{payment_comment}</code>\n\n"
    )
    
    instructions = (
        f"Важно:\n"
        f"• Отправьте точную сумму\n"
        f"• Обязательно укажите комментарий (Memo)\n"
        f"• Звёзды в течение 15 минут\n"
        f"• После 30 минут не оплаты заказ будет отменён"
    )
    
    #Объединяем все части
    full_message = payment_text + amount_block + wallet_block + comment_block + instructions
    
    if update.callback_query:
        await update.callback_query.message.reply_text(
            full_message,
            parse_mode='HTML'  # Включаем HTML парсинг для тега <code>
        )
    else:
        await update.message.reply_text(
            full_message,
            parse_mode='HTML'  # Включаем HTML парсинг для тега <code>
        )
    
    await notify_admin(context, order_id, user_id, username, stars, price_data['your_price'], payment_comment)

async def process_premium_selection(update: Update, context: ContextTypes.DEFAULT_TYPE, period: str):
    user = update.effective_user
    user_id = user.id
    username = user.username or f"user_{user_id}"
    
    price_data = calculate_price_for_premium(period)
    if not price_data:
        if update.callback_query:
            await update.callback_query.message.reply_text(
                "Ошибка при получении цены. Попробуйте позже.",
                reply_markup=get_premium_keyboard()
            )
        return
    
    # Определяем текст периода
    if period == "3_month":
        period_text = "3 месяца"
        stars_for_order = 0  # Для премиум заказов можно использовать stars=0
    elif period == "6_month":
        period_text = "6 месяцев"
        stars_for_order = 0
    elif period == "12_month":
        period_text = "1 год"
        stars_for_order = 0
    else:
        period_text = period
        stars_for_order = 0
    
    # Генерируем рандомный комментарий для оплаты
    payment_comment = generate_random_comment(username, period=period)
    
    # Создаем заказ для премиума
    order_id = create_order(
        user_id, username, stars_for_order,
        price_data['price'],
        price_data['your_price'],
        payment_comment
    )
    
    # Форматируем текст с отдельными блоками для копирования
    payment_text = (
        f"Заказ #{order_id} на Telegram Premium создан!\n\n"
        f"Покупатель: @{username}\n"
        f"Срок: {period_text}\n"
        f"Сумма: {price_data['your_price']} TON\n\n"
        f"Для оплаты отправьте точную сумму:\n"
    )
    
    # Блок с суммой для копирования (с фиксированным шрифтом)
    amount_block = (
        f"<code>{price_data['your_price']} TON</code>"
    )
    
    # Блок с адресом кошелька для копирования
    wallet_block = (
        f"\nНа адрес:\n"
        f"<code>{TON_WALLET}</code>\n\n"
    )
    
    comment_block = (
        f"КОММЕНТАРИЙ (Memo):\n"
        f"<code>{payment_comment}</code>\n\n"
    )
    
    instructions = (
        f"Важно:\n"
        f"• Отправьте точную сумму\n"
        f"• Обязательно укажите комментарий (Memo)\n"
        f"• Подписка активируется в течение 15 минут\n"
        f"• После 30 минут не оплаты заказ будет отменён"
    )
    
    # Объединяем все части
    full_message = payment_text + amount_block + wallet_block + comment_block + instructions
    
    if update.callback_query:
        await update.callback_query.message.reply_text(
            full_message,
            parse_mode='HTML'  # Включаем HTML парсинг для тега <code>
        )
    else:
        await update.message.reply_text(
            full_message,
            parse_mode='HTML'  # Включаем HTML парсинг для тега <code>
        )
    
    # Уведомляем администратора о новом заказе на премиум
    await notify_admin_premium(context, order_id, user_id, username, period_text, price_data['your_price'], payment_comment)

async def notify_admin(context, order_id, user_id, username, stars, price, comment):
    admin_message = (
        f"НОВЫЙ ЗАКАЗ #{order_id}\n\n"
        f"Тип: Telegram Stars\n"
        f"Покупатель: @{username}\n"
        f"ID: {user_id}\n"
        f"Звёзд: {stars}\n"
        f"Сумма: {price} TON\n"
        f"Комментарий (Memo): {comment}\n"
        f"Время: {datetime.now().strftime('%H:%M %d.%m.%Y')}"
    )
    
    try:
        await context.bot.send_message(
            chat_id=ADMIN_ID,
            text=admin_message,
            reply_markup=get_admin_keyboard(order_id)
        )
        mark_order_as_notified(order_id)
    except Exception as e:
        logger.error(f"Ошибка: {e}")

async def notify_admin_premium(context, order_id, user_id, username, period, price, comment):
    admin_message = (
        f"НОВЫЙ ЗАКАЗ #{order_id}\n\n"
        f"Тип: Telegram Premium\n"
        f"Покупатель: @{username}\n"
        f"ID: {user_id}\n"
        f"Срок: {period}\n"
        f"Сумма: {price} TON\n"
        f"Комментарий (Memo): {comment}\n"
        f"Время: {datetime.now().strftime('%H:%M %d.%m.%Y')}"
    )
    
    try:
        await context.bot.send_message(
            chat_id=ADMIN_ID,
            text=admin_message,
            reply_markup=get_admin_keyboard(order_id)
        )
        mark_order_as_notified(order_id)
    except Exception as e:
        logger.error(f"Ошибка: {e}")

async def notify_user(context, user_id, order_id, action):
    """Отправляет уведомление пользователю об изменении статуса заказа"""
    if action == "completed":
        message = f"✅ Заказ #{order_id} был выполнен!\n\nСпасибо за покупку! Посоветуйте нас друзьям 🙏"
    elif action == "cancelled":
        message = f"❌ Заказ #{order_id} был отменён администратором."
    else:
        return
    
    try:
        await context.bot.send_message(
            chat_id=user_id,
            text=message
        )
    except Exception as e:
        logger.error(f"Не удалось отправить уведомление пользователю {user_id}: {e}")

async def handle_admin_action(update: Update, context: ContextTypes.DEFAULT_TYPE, data: str):
    query = update.callback_query
    user_id = query.from_user.id
    
    if user_id != ADMIN_ID:
        return
    
    action, order_id = data.split("_")[1], int(data.split("_")[2])
    
    # Получаем информацию о заказе чтобы узнать user_id покупателя
    conn = sqlite3.connect('stars_bot.db')
    c = conn.cursor()
    c.execute("SELECT user_id FROM orders WHERE order_id = ?", (order_id,))
    result = c.fetchone()
    conn.close()
    
    if not result:
        await query.answer("Заказ не найден!")
        return
    
    buyer_user_id = result[0]
    
    if action == "complete":
        update_order_status(order_id, "completed")
        await query.message.edit_text(
            f"Заказ #{order_id} выполнен",
            reply_markup=None
        )
        # Уведомляем покупателя
        await notify_user(context, buyer_user_id, order_id, "completed")
        
    elif action == "cancel":
        update_order_status(order_id, "cancelled")
        await query.message.edit_text(
            f"Заказ #{order_id} отменен",
            reply_markup=None
        )
        # Уведомляем покупателя
        await notify_user(context, buyer_user_id, order_id, "cancelled")

# ==================== ЗАПУСК ====================
def main():
    init_db()
    
    application = Application.builder().token(BOT_TOKEN).build()
    
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CallbackQueryHandler(handle_callback_query))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    logger.info("Бот запускается...")
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == '__main__':
    main()