# Telegram-Bot
import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

# Включение логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

logger = logging.getLogger(__name__)

# Список товаров (временное хранилище в памяти)
products = []

# Функция для старта бота
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("""Привет! Я ваш бот для управления товарами. Вот доступные команды:
/spisok - показать список товаров
/add - добавить новый товар
/update - обновить данные товара
/choose - выбрать товар по названию""")

# Функция для вывода списка товаров
async def spisok(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if not products:
        await update.message.reply_text("Список товаров пуст.")
    else:
        message = "Список товаров:\n\n"
        for product in products:
            message += f"ID: {product['id']}, Название: {product['name']}, Статус: {product['status']}, Дата: {product['date']}, Процент брака: {product['defect_rate']}%\n"
        await update.message.reply_text(message)

# Функция для добавления товара
async def add(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("Пожалуйста, введите описание товара в формате: ID, Название, Статус, Дата (ДД.MM.ГГГГ), Процент брака.")
    context.user_data['adding'] = True

# Обработчик сообщения для добавления нового товара
async def process_addition(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if 'adding' in context.user_data:
        try:
            id_str, name, status, date, defect_rate_str = map(lambda x: x.strip(), update.message.text.split(','))
            defect_rate = int(defect_rate_str)

            product = {
                'id': id_str,
                'name': name,
                'status': status,
                'date': date,
                'defect_rate': defect_rate
            }
            products.append(product)

            await update.message.reply_text(f"Товар добавлен:\nID: {product['id']}, Название: {product['name']}, Статус: {product['status']}, Дата: {product['date']}, Процент брака: {product['defect_rate']}%")
            del context.user_data['adding']  # Удаляем ключ, чтобы не продолжать добавление
            await spisok(update, context)  # Показать обновленный список товаров
        except Exception:
            await update.message.reply_text("Ошибка формата. Пожалуйста, попробуйте снова.")
    else:
        await update.message.reply_text("Пожалуйста, используйте команду /add для добавления нового товара.")

# Функция для обновления товара
async def update_product(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("Пожалуйста, введите ID товара, который хотите обновить.")
    context.user_data['updating'] = True

# Обработчик для обновления товара
async def process_update(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if 'updating' in context.user_data:
        product_id = update.message.text.strip()
        product = next((p for p in products if p['id'] == product_id), None)

        if product:
            await update.message.reply_text(f"Обновляем товар {product['name']}.\nПожалуйста, введите новый статус, дату и процент брака в формате:\nСтатус, Дата (ДД.MM.ГГГГ), Процент брака.")
            context.user_data['current_product'] = product  # Сохраняем текущий товар для обновления
            del context.user_data['updating']  # Удаляем ключ
        else:
            await update.message.reply_text("Товар с таким ID не найден.")
    else:
        await update.message.reply_text("Пожалуйста, используйте команду /update для обновления товара.")

# Обработчик обновления данных товара
async def process_update_data(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if 'current_product' in context.user_data:
        product = context.user_data['current_product']
        try:
            status_str, date_str, defect_rate_str = map(lambda x: x.strip(), update.message.text.split(','))
            defect_rate = int(defect_rate_str)

            # Обновляем данные товара
            product['status'] = status_str
            product['date'] = date_str
            product['defect_rate'] = defect_rate

            await update.message.reply_text(f"Товар обновлен:\nID: {product['id']}, Название: {product['name']}, Статус: {product['status']}, Дата: {product['date']}, Процент брака: {product['defect_rate']}%")
            del context.user_data['current_product']  # Удаляем ключ после обновления
            await spisok(update, context)  # Показать обновленный список товаров
        except Exception:
            await update.message.reply_text("Ошибка формата. Пожалуйста, попробуйте снова.")
    else:
        await update.message.reply_text("Чтобы обновить, сначала используйте команду /update.")

# Функция для выбора товара по названию
async def choose(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("Пожалуйста, введите название товара.")
    context.user_data['choosing'] = True

# Обработчик для выбора товара
async def process_choice(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if 'choosing' in context.user_data:
        product_name = update.message.text.strip()
        product = next((p for p in products if p['name'].lower() == product_name.lower()), None)

        if product:
            await update.message.reply_text(f"Ваш товар:\nID: {product['id']}, Название: {product['name']}, Статус: {product['status']}, Дата: {product['date']}, Процент брака: {product['defect_rate']}%")
        else:
            await update.message.reply_text("Товар с таким названием не найден.")
        del context.user_data['choosing']  # Удаляем ключ
    else:
        await update.message.reply_text("Пожалуйста, используйте команду /choose для выбора товара.")

# Основная функция
def main() -> None:
    # Создаем приложение
    application = Application.builder().token("7729422654:AAFJb5d7m94NpiOfSux7hEjv1R1o04GDecE").build()

# Регистрация обработчиков
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("spisok", spisok))
    application.add_handler(CommandHandler("add", add))
    application.add_handler(CommandHandler("update", update_product))
    application.add_handler(CommandHandler("choose", choose))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, process_addition))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, process_update))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, process_update_data))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, process_choice))

    # Запуск бота
    application.run_polling()

if __name__ == '__main__':
    import asyncio
    asyncio.run(main())
