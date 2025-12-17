import logging
import asyncio
from typing import List, Dict, Optional
from datetime import datetime
import json
import csv
from telegram import Update, User, Chat, Message
from telegram.ext import (
    Application,
    CommandHandler,
    ContextTypes,
    MessageHandler,
    filters
)
from telegram.constants import ParseMode, ChatType

# Настройка логирования
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

class GroupMessageExporter:
    def __init__(self):
        self.user_data = {}
        
    async def start(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /start"""
        user = update.effective_user
        welcome_text = (
            f"👋 Привет, {user.first_name}!\n\n"
            "🤖 Я бот для выгрузки сообщений из публичных групп.\n\n"
            "📋 **Доступные команды:**\n"
            "/start - Начать работу\n"
            "/help - Помощь\n"
            "/export_messages @username - Экспорт сообщений пользователя\n"
            "/search_messages @username ключевое_слово - Поиск сообщений\n"
            "/set_limit число - Установить лимит сообщений (по умолчанию 100)\n"
            "/export_formats - Показать форматы экспорта\n\n"
            "⚠️ **Важно:** Бот должен быть участником группы и иметь права на чтение сообщений."
        )
        
        await update.message.reply_text(welcome_text, parse_mode=ParseMode.MARKDOWN)
    
    async def help_command(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик команды /help"""
        help_text = (
            "📚 **Помощь по использованию бота:**\n\n"
            "1. Добавьте бота в публичную группу/канал\n"
            "2. Дайте боту права на чтение сообщений\n"
            "3. Используйте команды:\n\n"
            "📤 **Экспорт сообщений пользователя:**\n"
            "`/export_messages @username`\n"
            "Или: `/export_messages user_id`\n\n"
            "🔍 **Поиск сообщений:**\n"
            "`/search_messages @username ключевое слово`\n\n"
            "⚙️ **Настройки:**\n"
            "`/set_limit 500` - установить лимит сообщений\n"
            "`/export_formats` - выбрать формат экспорта\n\n"
            "📝 **Форматы экспорта:** JSON, CSV, TXT\n\n"
            "⚠️ **Ограничения:**\n"
            "- Бот работает только в публичных группах\n"
            "- Максимальный лимит: 1000 сообщений за запрос\n"
            "- Бот не может получать сообщения из приватных чатов без разрешения"
        )
        await update.message.reply_text(help_text, parse_mode=ParseMode.MARKDOWN)
    
    async def export_messages(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Экспорт сообщений пользователя"""
        if not context.args:
            await update.message.reply_text(
                "❌ Укажите username или ID пользователя.\n"
                "Пример: `/export_messages @username`",
                parse_mode=ParseMode.MARKDOWN
            )
            return
        
        user_identifier = context.args[0]
        limit = int(context.user_data.get('message_limit', 100))
        
        if limit > 1000:
            limit = 1000
        
        await update.message.reply_text(
            f"🔄 Начинаю поиск сообщений для {user_identifier}...\n"
            f"Лимит: {limit} сообщений\n"
            "Это может занять некоторое время..."
        )
        
        # Получаем информацию о пользователе
        try:
            target_user = await self._get_user_info(context, user_identifier)
            if not target_user:
                await update.message.reply_text("❌ Пользователь не найден")
                return
        except Exception as e:
            await update.message.reply_text(f"❌ Ошибка: {str(e)}")
            return
        
        # Получаем сообщения из всех групп
        messages = await self._collect_user_messages(context, target_user.id, limit)
        
        if not messages:
            await update.message.reply_text("❌ Сообщения не найдены")
            return
        
        # Экспорт в выбранные форматы
        await self._export_data(update, messages, target_user)
    
    async def _get_user_info(self, context: ContextTypes.DEFAULT_TYPE, identifier: str):
        """Получение информации о пользователе"""
        try:
            # Если это username (начинается с @)
            if identifier.startswith('@'):
                # Пытаемся найти пользователя по username
                user = await context.bot.get_chat(identifier)
                return user
            # Если это числовой ID
            elif identifier.isdigit():
                user = await context.bot.get_chat(int(identifier))
                return user
        except Exception as e:
            logger.error(f"Error getting user info: {e}")
            return None
    
    async def _collect_user_messages(self, context: ContextTypes.DEFAULT_TYPE, 
                                   user_id: int, limit: int) -> List[Dict]:
        """Сбор сообщений пользователя из групп"""
        messages = []
        
        # В реальном боте здесь нужно получить список групп,
        # где присутствует бот, и проверить каждую
        
        # Это демо-версия - в реальности нужно использовать
        # Telegram API для получения сообщений из конкретных чатов
        
        # Заглушка для демонстрации
        # В реальном проекте здесь будет логика обхода групп
        
        return messages
    
    async def _export_data(self, update: Update, messages: List[Dict], user: User):
        """Экспорт данных в разные форматы"""
        if not messages:
            return
        
        # Подготовка данных
        export_data = []
        for msg in messages:
            export_data.append({
                'date': msg.get('date', ''),
                'chat_title': msg.get('chat_title', ''),
                'chat_id': msg.get('chat_id', ''),
                'message_id': msg.get('message_id', ''),
                'text': msg.get('text', '')[:500],  # Ограничиваем длину
                'has_media': msg.get('has_media', False)
            })
        
        # Экспорт в JSON
        json_data = json.dumps(export_data, ensure_ascii=False, indent=2, default=str)
        
        # Экспорт в CSV
        csv_data = self._convert_to_csv(export_data)
        
        # Экспорт в TXT
        txt_data = self._convert_to_txt(export_data, user)
        
        # Отправка файлов пользователю
        await self._send_export_files(update, json_data, csv_data, txt_data, user.username or user.id)
    
    def _convert_to_csv(self, data: List[Dict]) -> str:
        """Конвертация данных в CSV"""
        if not data:
            return ""
        
        import io
        output = io.StringIO()
        
        # Определяем все возможные ключи
        fieldnames = set()
        for item in data:
            fieldnames.update(item.keys())
        
        writer = csv.DictWriter(output, fieldnames=sorted(fieldnames))
        writer.writeheader()
        writer.writerows(data)
        
        return output.getvalue()
    
    def _convert_to_txt(self, data: List[Dict], user: User) -> str:
        """Конвертация данных в TXT"""
        txt_lines = []
        txt_lines.append(f"Экспорт сообщений пользователя: {user.username or user.first_name or user.id}")
        txt_lines.append(f"Дата экспорта: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        txt_lines.append(f"Количество сообщений: {len(data)}")
        txt_lines.append("=" * 50)
        
        for i, msg in enumerate(data, 1):
            txt_lines.append(f"\n{i}. Дата: {msg.get('date', 'N/A')}")
            txt_lines.append(f"   Чат: {msg.get('chat_title', 'N/A')}")
            txt_lines.append(f"   Текст: {msg.get('text', '')}")
            txt_lines.append(f"   Медиа: {'Да' if msg.get('has_media') else 'Нет'}")
            txt_lines.append("-" * 30)
        
        return "\n".join(txt_lines)
    
    async def _send_export_files(self, update: Update, json_data: str, 
                                csv_data: str, txt_data: str, username: str):
        """Отправка экспортированных файлов"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        
        try:
            # Отправляем JSON
            if json_data:
                json_filename = f"messages_{username}_{timestamp}.json"
                await update.message.reply_document(
                    document=json_data.encode('utf-8'),
                    filename=json_filename,
                    caption=f"📊 JSON экспорт ({len(json_data)} байт)"
                )
            
            # Отправляем CSV
            if csv_data:
                csv_filename = f"messages_{username}_{timestamp}.csv"
                await update.message.reply_document(
                    document=csv_data.encode('utf-8'),
                    filename=csv_filename,
                    caption=f"📈 CSV экспорт"
                )
            
            # Отправляем TXT
            if txt_data:
                txt_filename = f"messages_{username}_{timestamp}.txt"
                await update.message.reply_document(
                    document=txt_data.encode('utf-8'),
                    filename=txt_filename,
                    caption=f"📝 Текстовый экспорт"
                )
            
            await update.message.reply_text(
                f"✅ Экспорт завершен!\n"
                f"📁 Файлы отправлены в форматах: JSON, CSV, TXT\n\n"
                f"💡 Совет: Используйте /search_messages для поиска по ключевым словам"
            )
            
        except Exception as e:
            logger.error(f"Error sending files: {e}")
            await update.message.reply_text(f"❌ Ошибка при отправке файлов: {str(e)}")
    
    async def search_messages(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Поиск сообщений по ключевым словам"""
        if len(context.args) < 2:
            await update.message.reply_text(
                "❌ Укажите username и ключевое слово для поиска.\n"
                "Пример: `/search_messages @username ключевое_слово`",
                parse_mode=ParseMode.MARKDOWN
            )
            return
        
        user_identifier = context.args[0]
        search_query = " ".join(context.args[1:])
        
        await update.message.reply_text(
            f"🔍 Ищу сообщения {user_identifier} с ключевым словом: '{search_query}'...\n"
            "Это может занять некоторое время..."
        )
        
        # Здесь будет реализация поиска
        # В демо-версии просто возвращаем заглушку
        
        await update.message.reply_text(
            f"📊 Поиск завершен.\n"
            f"Найдено сообщений: 0\n\n"
            f"💡 Для полного экспорта используйте `/export_messages {user_identifier}`",
            parse_mode=ParseMode.MARKDOWN
        )
    
    async def set_limit(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Установка лимита сообщений"""
        if not context.args or not context.args[0].isdigit():
            await update.message.reply_text(
                "❌ Укажите число для лимита.\n"
                "Пример: `/set_limit 500`",
                parse_mode=ParseMode.MARKDOWN
            )
            return
        
        limit = int(context.args[0])
        
        if limit > 1000:
            limit = 1000
            await update.message.reply_text("⚠️ Максимальный лимит установлен: 1000 сообщений")
        elif limit < 10:
            limit = 10
            await update.message.reply_text("⚠️ Минимальный лимит установлен: 10 сообщений")
        else:
            context.user_data['message_limit'] = limit
            await update.message.reply_text(f"✅ Лимит сообщений установлен: {limit}")
    
    async def export_formats(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Показ доступных форматов экспорта"""
        formats_text = (
            "📁 **Доступные форматы экспорта:**\n\n"
            "1. **JSON** - структурированный формат для программной обработки\n"
            "2. **CSV** - табличный формат для Excel/Google Sheets\n"
            "3. **TXT** - текстовый формат для чтения\n\n"
            "📊 **Что экспортируется:**\n"
            "- Текст сообщения\n"
            "- Дата и время\n"
            "- Название чата\n"
            "- ID сообщения\n"
            "- Наличие медиафайлов\n\n"
            "⚙️ Все форматы создаются автоматически при экспорте."
        )
        
        await update.message.reply_text(formats_text, parse_mode=ParseMode.MARKDOWN)
    
    async def error_handler(self, update: Update, context: ContextTypes.DEFAULT_TYPE):
        """Обработчик ошибок"""
        logger.error(f"Update {update} caused error {context.error}")
        
        try:
            await update.message.reply_text(
                "❌ Произошла ошибка при обработке запроса.\n"
                "Попробуйте еще раз или обратитесь к администратору."
            )
        except:
            pass

def main():
    """Основная функция запуска бота"""
    
    # Вставьте ваш токен бота
    TOKEN = "YOUR_BOT_TOKEN_HERE"
    
    # Создаем приложение
    application = Application.builder().token(TOKEN).build()
    
    # Создаем экземпляр экспортера
    exporter = GroupMessageExporter()
    
    # Регистрируем обработчики команд
    application.add_handler(CommandHandler("start", exporter.start))
    application.add_handler(CommandHandler("help", exporter.help_command))
    application.add_handler(CommandHandler("export_messages", exporter.export_messages))
    application.add_handler(CommandHandler("search_messages", exporter.search_messages))
    application.add_handler(CommandHandler("set_limit", exporter.set_limit))
    application.add_handler(CommandHandler("export_formats", exporter.export_formats))
    
    # Обработчик ошибок
    application.add_error_handler(exporter.error_handler)
    
    # Запускаем бота
    print("🤖 Бот запущен...")
    print("🔄 Используйте Ctrl+C для остановки")
    
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == '__main__':
    main()
