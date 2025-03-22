import asyncio
import os
import sqlite3
from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command

TOKEN = os.getenv("TOKEN")  # Берём токен из Railway

bot = Bot(token=TOKEN)
dp = Dispatcher()

# Подключаем базу данных
conn = sqlite3.connect("queue.db")
cursor = conn.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS queue (id INTEGER PRIMARY KEY, name TEXT)")
conn.commit()

# Команда /start
@dp.message(Command("start"))
async def start(message: types.Message):
    await message.answer("Привет! Отправь имя, чтобы добавить в очередь.")

# Добавление имени
@dp.message()
async def add_to_queue(message: types.Message):
    name = message.text.strip()
    cursor.execute("INSERT INTO queue (name) VALUES (?)", (name,))
    conn.commit()
    await message.answer(f"{name} добавлен в очередь!")

# Просмотр очереди
@dp.message(Command("queue"))
async def show_queue(message: types.Message):
    cursor.execute("SELECT name FROM queue")
    names = cursor.fetchall()
    if names:
        queue_list = "\n".join([name[0] for name in names])
        await message.answer(f"Очередь:\n{queue_list}")
    else:
        await message.answer("Очередь пуста.")

# Удаление первого в очереди
@dp.message(Command("next"))
async def remove_from_queue(message: types.Message):
    cursor.execute("SELECT id, name FROM queue ORDER BY id LIMIT 1")
    first = cursor.fetchone()
    if first:
        cursor.execute("DELETE FROM queue WHERE id = ?", (first[0],))
        conn.commit()
        await message.answer(f"{first[1]} удален из очереди.")
    else:
        await message.answer("Очередь пуста.")

# Запуск бота
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
