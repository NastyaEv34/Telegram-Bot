# Telegram-Bot
import logging
from telegram import Update, ForceReply
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext

# Включаем логирование
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)

logger = logging.getLogger(__name__)

# Простой хранилище данных для товаров
products = {1: {'name': 'Товар 1', 'status': 'непроверенный'},
            2: {'name': 'Товар 2', 'status': 'непроверенный'}}

# Команда для старта
def start(update: Update, context: CallbackContext) -> None:
    user = update.effective_user
    update.message.reply_html(
        rf"Привет, {user.mention_html()}! Я Телеграм-бот для проверки товаров. Используйте команду /check для проверки товара.",
        reply_markup=ForceReply(selective=True),
    )

# Команда для проверки товара
def check_product(update: Update, context: CallbackContext) -> None:
    update.message.reply_text("Выберите номер товара (1 или 2), чтобы проверить его:")

# Обработка выбора товара
def select_product(update: Update, context: CallbackContext) -> None:
    product_id = int(update.message.text)
    if product_id in products:
        context.user_data['selected_product'] = product_id
        update.message.reply_text(f"Вы выбрали: {products[product_id]['name']}. Введите 'проверенный' для подтверждения или 'непроверенный' для отказа.")
    else:
        update.message.reply_text("Неверный номер товара. Пожалуйста, выберите 1 или 2.")

# Отметить товар как проверенный
def mark_verified(update: Update, context: CallbackContext) -> None:
    product_id = context.user_data.get('selected_product')
    if product_id is not None:
        products[product_id]['status'] = 'проверенный'
        update.message.reply_text(f"{products[product_id]['name']} отмечен как 'проверенный'.")
    else:
        update.message.reply_text("Нет выбранного товара.")

# Отметить товар как непроверенный
def mark_unverified(update: Update, context: CallbackContext) -> None:
    product_id = context.user_data.get('selected_product')
    if product_id is not None:
        products[product_id]['status'] = 'непроверенный'
        update.message.reply_text(f"{products[product_id]['name']} отмечен как 'непроверенный'.")
    else:
        update.message.reply_text("Нет выбранного товара.")

# Получить нормальные показатели для товара
def normal_indicators(update: Update, context: CallbackContext) -> None:
    product_id = context.user_data.get('selected_product')
    if product_id is not None:
        update.message.reply_text(f"Нормальные показатели для товара {products[product_id]['name']}: ...") # Здесь можно добавить информацию о нормальных показателях
    else:
        update.message.reply_text("Нет выбранного товара.")

# Генерировать отчет
def generate_report(update: Update, context: CallbackContext) -> None:
    verified_count = sum(1 for product in products.values() if product['status'] == 'проверенный')
    unverified_count = sum(1 for product in products.values() if product['status'] == 'непроверенный')
    report = f"Отчет о проверке товаров:\nПроверенные товары: {verified_count}\nНепроверенные товары: {unverified_count}"
    update.message.reply_text(report)

# Статус товара
def product_status(update: Update, context: CallbackContext) -> None:
    status_report = "\n".join([f"{product['name']}: {product['status']}" for product in products.values()])
    update.message.reply_text(f"Список товаров и их статусы:\n{status_report}")

# Удаление товара
def delete_product(update: Update, context: CallbackContext) -> None:
    # Пример удаления
    product_id = 1 # Заменить на выбор пользователя
    if product_id in products:
        del products[product_id]
        update.message.reply_text(f"Товар с ID {product_id} был удален из базы данных.")
    else:
        update.message.reply_text("Товар не найден.")

# Добавление нового товара
def add_product(update: Update, context: CallbackContext) -> None:
    product_id = 3 # Пример ID нового товара
    products[product_id] = {'name': 'Товар 3', 'status': 'непроверенный'}
    update.message.reply_text(f"Товар {products[product_id]['name']} был успешно добавлен.")

def main() -> None:
    # Создаем бота с токеном
    updater = Updater("YOUR_TELEGRAM_BOT_TOKEN")

    # Получаем диспетчер для регистрации обработчиков
    dispatcher = updater.dispatcher

    # Регистрация обработчиков команд
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("check", check_product))
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, select_product))

    # Регистрация обработчиков для действий с товарами
    dispatcher.add_handler(MessageHandler(Filters.regex('^проверенный$'), mark_verified))
    dispatcher.add_handler(MessageHandler(Filters.regex('^непроверенный$'), mark_unverified))
    dispatcher.add_handler(CommandHandler("indicators", normal_indicators))
    dispatcher.add_handler(CommandHandler("report", generate_report))
    dispatcher.add_handler(CommandHandler("status", product_status))
    dispatcher.add_handler(CommandHandler("delete", delete_product))
    dispatcher.add_handler(CommandHandler("add", add_product))

    # Запуск бота
    updater.start_polling()

    # Бот будет работать до тех пор, пока вы не прервёте его
    updater.idle()

if __name__ == '__main__':
    main()
