import telebot
from telebot import types

BOT_TOKEN = '7879706443:AAE_DAsXrIhnZSeYV-1o55AAQ9A7Kli6m8M'
ADMIN_CHAT_ID = 8048425240  # Замените на свой chat_id администратора
ADMIN_USERNAME = '@DromRMD'  # Ваш Telegram username для связи

bot = telebot.TeleBot(BOT_TOKEN)

# Хранилище данных
user_data = {}
referrals = {}
user_bonuses = {}
user_orders_count = {}
reviews = []

# Цены на Brawl Stars
brawl_stars_prices = {
    "🥉 Бронза I–III": 100,
    "🥈 Серебро I–III": 200,
    "🥇 Золото I–III": 300,
    "💎 Алмаз I–III": 400,
    "🔥 Мифик I–III": 500,
    "👑 Легенда I–III": 600,
    "🎖️ Мастер": 700,
}

# Цены на Standoff 2
standoff_prices = {
    "🥉 Бронза I–IV": 100,
    "🥈 Серебро I–IV": 200,
    "🥇 Золото I–IV": 300,
    "🔥 Феникс": 400,
    "⚔️ Рейнджер": 500,
    "🏆 Чемпион": 600,
    "🎖️ Мастер": 700,
    "💼 Элита": 800,
    "👑 Легенда": 900,
}

payment_card = "4400 4302 0757 1545"

# Главное меню
def main_menu():
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("🛒 Услуги и цены", "📝 Заказать буст")
    markup.add("📞 Связь с нами", "🗣️ Отзывы")
    markup.add("❓ FAQ", "🏆 Топ покупателей")
    return markup

# Старт и рефералы
@bot.message_handler(commands=['start'])
def start_command(message):
    chat_id = message.chat.id
    args = message.text.split()
    if len(args) > 1 and args[1].isdigit() and args[1] != str(chat_id):
        referrals[chat_id] = int(args[1])
    bot.send_message(chat_id, "🎮 Привет! Добро пожаловать в BoostMaster — твой персональный магазин по бусту аккаунтов и рангов в Brawl Stars и Standoff 2! 🚀\n\nИспользуйте меню ниже для навигации. 👇", reply_markup=main_menu())

# Услуги и цены
@bot.message_handler(func=lambda m: m.text == "🛒 Услуги и цены")
def services_command(message):
    brawl_prices = "\n".join([f"- {rank}: {price}₽" for rank, price in brawl_stars_prices.items()])
    standoff_prices_text = "\n".join([f"- {rank}: {price}₽" for rank, price in standoff_prices.items()])
    text = (f"📋 Наши услуги и цены:\n\n"
            f"🟦 Brawl Stars:\n{brawl_prices}\n\n"
            f"🟥 Standoff 2:\n{standoff_prices_text}")
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

# Связь с нами
@bot.message_handler(func=lambda m: m.text == "📞 Связь с нами")
def contact_command(message):
    bot.send_message(message.chat.id, f"📞 Связаться с нами можно через Telegram: {ADMIN_USERNAME} 📲", reply_markup=main_menu())

# Отзывы
@bot.message_handler(func=lambda m: m.text == "🗣️ Отзывы")
def reviews_command(message):
    if not reviews:
        bot.send_message(message.chat.id, "📢 Отзывов пока нет. Будьте первым, кто оставит отзыв! ✍️", reply_markup=main_menu())
    else:
        text = "🗣️ Отзывы наших клиентов:\n\n"
        for i, rev in enumerate(reviews[-5:], 1):
            text += f"{i}. {rev['user']}:\n«{rev['text']}»\n\n"
        bot.send_message(message.chat.id, text, reply_markup=main_menu())
    # Предложить оставить отзыв
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("✍️ Оставить отзыв", "🔙 Назад")
    bot.send_message(message.chat.id, "Хотите оставить отзыв? 📝", reply_markup=markup)
    bot.register_next_step_handler(message, review_step)

def review_step(message):
    if message.text == "✍️ Оставить отзыв":
        bot.send_message(message.chat.id, "📝 Напишите ваш отзыв:", reply_markup=types.ReplyKeyboardMarkup(resize_keyboard=True).add("🔙 Назад"))
        bot.register_next_step_handler(message, save_review)
    else:
        bot.send_message(message.chat.id, "⬅️ Возврат в главное меню.", reply_markup=main_menu())

def save_review(message):
    if message.text == "🔙 Назад":
        bot.send_message(message.chat.id, "⬅️ Возврат в главное меню.", reply_markup=main_menu())
        return
    user = message.from_user.username or message.from_user.first_name or "Пользователь"
    reviews.append({"user": user, "text": message.text})
    bot.send_message(message.chat.id, "Спасибо за ваш отзыв! 😊", reply_markup=main_menu())

# FAQ
@bot.message_handler(func=lambda m: m.text == "❓ FAQ")
def faq_command(message):
    faq_text = (
        "❓ Часто задаваемые вопросы:\n\n"
        "1. Как заказать буст? 🛒\n"
        "- Выберите '📝 Заказать буст' и следуйте инструкциям.\n\n"
        "2. Как оплатить? 💳\n"
        f"- Оплатить можно на карту: `{payment_card}`.\n\n"
        "3. Можно ли получить скидку? 🎁\n"
        "- Да, у нас есть реферальная система с бонусами.\n\n"
        "4. Сколько времени занимает буст? ⏳\n"
        "- Время зависит от выбранного ранга и игры.\n\n"
        "Если остались вопросы, напишите нам через '📞 Связь с нами'."
    )
    bot.send_message(message.chat.id, faq_text, parse_mode='Markdown', reply_markup=main_menu())

# Топ покупателей
@bot.message_handler(func=lambda m: m.text == "🏆 Топ покупателей")
def top_buyers_command(message):
    if not user_orders_count:
        bot.send_message(message.chat.id, "Пока нет заказов. Будьте первым! 🥇", reply_markup=main_menu())
        return
    # Сортируем по убыванию заказов
    sorted_users = sorted(user_orders_count.items(), key=lambda x: x[1], reverse=True)
    text = "🏆 Топ покупателей:\n\n"
    for i, (user_id, count) in enumerate(sorted_users[:10], 1):
        try:
            user = bot.get_chat(user_id)
            username = f"@{user.username}" if user.username else user.first_name or f"User {user_id}"
        except Exception:
            username = f"Пользователь {user_id}"
        text += f"{i}. {username} — {count} заказ(ов) 🎉\n"
    bot.send_message(message.chat.id, text, reply_markup=main_menu())

# Заказать буст — выбор игры
@bot.message_handler(func=lambda m: m.text == "📝 Заказать буст")
def order_command(message):
    chat_id = message.chat.id
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("🎮 Brawl Stars", "🔫 Standoff 2")
    markup.add("🔙 Назад")
    bot.send_message(chat_id, "🎮 Выберите игру для буста:", reply_markup=markup)
    bot.register_next_step_handler(message, process_game)

def process_game(message):
    chat_id = message.chat.id
    game = message.text.replace("🎮 ", "").replace("🔫 ", "")
    if message.text == "🔙 Назад":
        bot.send_message(chat_id, "⬅️ Возврат в главное меню.", reply_markup=main_menu())
        return
    if game not in ["Brawl Stars", "Standoff 2"]:
        bot.send_message(chat_id, "❗ Пожалуйста, выберите игру из списка.", reply_markup=main_menu())
        return
    user_data[chat_id] = {"game": game}
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    if game == "Brawl Stars":
        markup.add("🥉 Бронза I–III", "🥈 Серебро I–III", "🥇 Золото I–III")
        markup.add("💎 Алмаз I–III", "🔥 Мифик I–III", "👑 Легенда I–III", "🎖️ Мастер")
    else:
        markup.add("🥉 Бронза I–IV", "🥈 Серебро I–IV", "🥇 Золото I–IV")
        markup.add("🔥 Феникс", "⚔️ Рейнджер", "🏆 Чемпион", "🎖️ Мастер", "💼 Элит", "👑 Легенда")
    markup.add("🔙 Назад")
    bot.send_message(chat_id, "💬 Какой ранг хотите?", reply_markup=markup)
    bot.register_next_step_handler(message, process_rank)

def process_rank(message):
    chat_id = message.chat.id
    rank = message.text
    if rank == "🔙 Назад":
        order_command(message)
        return
    if rank not in brawl_stars_prices and rank not in standoff_prices:
        bot.send_message(chat_id, "❗ Пожалуйста, выберите ранг из списка.", reply_markup=main_menu())
        return
    user_data[chat_id]['rank'] = rank
    bot.send_message(chat_id, "👤 Введите ваш ник или ID в игре:", reply_markup=types.ReplyKeyboardMarkup(resize_keyboard=True).add("🔙 Назад"))
    bot.register_next_step_handler(message, process_nick)

def process_nick(message):
    chat_id = message.chat.id
    nick = message.text
    if nick == "🔙 Назад":
        process_rank(message)
        return
    user_data[chat_id]['nick'] = nick
    bot.send_message(chat_id, "📧 Введите вашу почту для связи:", reply_markup=types.ReplyKeyboardMarkup(resize_keyboard=True).add("🔙 Назад"))
    bot.register_next_step_handler(message, process_email)

def process_email(message):
    chat_id = message.chat.id
    email = message.text
    if email == "🔙 Назад":
        process_nick(message)
        return
    user_data[chat_id]['email'] = email
    data = user_data[chat_id]
    # Рассчёт цены
    if data['game'] == "Brawl Stars":
        base_price = brawl_stars_prices.get(data['rank'], 0)
    else:
        base_price = standoff_prices.get(data['rank'], 0)

    # Бонус за реферала 5%
    user_ref = referrals.get(chat_id)
    bonus_amount = 0
    final_price = base_price
    if user_ref:
        bonus_amount = int(base_price * 0.05)
        final_price = base_price - bonus_amount
        user_bonuses[chat_id] = bonus_amount

    # Отправляем итоговую информацию и карту оплаты
    payment_msg = (f"🎮 Заказ на буст:\n"
                   f"🎲 Игра: {data['game']}\n"
                   f"🏅 Ранг: {data['rank']}\n"
                   f"👤 Ник/ID: {data['nick']}\n"
                   f"📧 Почта: {data['email']}\n\n"
                   f"💰 Итоговая цена: {final_price}₽\n"
                   f"🎁 Скидка за реферала: {bonus_amount}₽\n\n"
                   f"💳 Для оплаты используйте карту:\n`{payment_card}`")

    bot.send_message(chat_id, payment_msg, parse_mode="Markdown", reply_markup=main_menu())

    # Обновляем количество заказов
    user_orders_count[chat_id] = user_orders_count.get(chat_id, 0) + 1

    # Уведомление администратору с ником и username
    user_mention = f"@{message.from_user.username}" if message.from_user.username else message.from_user.first_name
    admin_msg = (f"🆘 Новый заказ:\n"
                 f"Пользователь: {user_mention}\n"
                 f"Игра: {data['game']}\n"
                 f"Ранг: {data['rank']}\n"
                 f"Ник/ID: {data['nick']}\n"
                 f"Почта: {data['email']}\n"
                 f"Цена: {final_price}₽")
    bot.send_message(ADMIN_CHAT_ID, admin_msg)

bot.infinity_polling()
