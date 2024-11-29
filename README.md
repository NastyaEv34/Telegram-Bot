import logging
from telegram import Update, ForceReply
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

# Настроим логирование
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)
logger = logging.getLogger(__name__)

# Простой хранилище данных для товаров
products = {
    1: {'name': 'Товар 1', 'status': 'непроверенный'},
    2: {'name': 'Товар 2', 'status': 'непроверенный'}
}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    user = update.effective_user
    await update.message.reply_html(
        rf"Привет, {user.mention_html()}! Я Телеграм-бот для проверки товаров. Используйте команду /check для проверки товара.",
        reply_markup=ForceReply(selective=True),
    )

async def check_product(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("Выберите номер товара (1 или 2), чтобы проверить его:")

async def select_product(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    try:
        product_id = int(update.message.text)
        if product_id in products:
            context.user_data['selected_product'] = product_id
            await update.message.reply_text(f"Вы выбрали: {products[product_id]['name']}. Введите 'проверенный' для подтверждения или 'непроверенный' для отказа.")
        else:
            await update.message.reply_text("Неверный номер товара. Пожалуйста, выберите 1 или 2.")
    except ValueError:
        await update.message.reply_text("Пожалуйста, введите корректный номер товара.")

async def mark_verified(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    product_id = context.user_data.get('selected_product')
    if product_id is not None:
        products[product_id]['status'] = 'проверенный'
        await update.message.reply_text(f"{products[product_id]['name']} отмечен как 'проверенный'.")
    else:
        await update.message.reply_text("Нет выбранного товара.")

async def mark_unverified(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    product_id = context.user_data.get('selected_product')
    if product_id is not None:
        products[product_id]['status'] = 'непроверенный'
        await update.message.reply_text(f"{products[product_id]['name']} отмечен как 'непроверенный'.")
    else:
        await update.message.reply_text("Нет выбранного товара.")

# Остальные функции остаются без изменений, только добавьте await перед методами отправки сообщений.

async def main() -> None:
    app = ApplicationBuilder().token("YOUR_TELEGRAM_BOT_TOKEN").build()

    # Регистрация обработчиков команд
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("check", check_product))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, select_product))
    app.add_handler(MessageHandler(filters.Regex('^проверенный$'), mark_verified))
    app.add_handler(MessageHandler(filters.Regex('^непроверенный$'), mark_unverified))

    # Запуск бота
    await app.run_polling()

if __name__ == '__main__':
    import asyncio
    asyncio.run(main())

    # Бот будет работать до тех пор, пока вы не прервёте его
    updater.idle()

if __name__ == '__main__':
    main()
