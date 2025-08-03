import asyncio
import requests
from aiogram import Bot, Dispatcher, types, F
from aiogram.types import Message, KeyboardButton, ReplyKeyboardMarkup
from aiogram.filters import Command


BOT_TOKEN = "7649996689:AAHMgJgL94OPMWHXilQ5ysAEEjIEz0VO3uM"
WEATHER_API_KEY = "ccf9d27f5bf645960e60a081e27f885e"

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()


start_keyboard = ReplyKeyboardMarkup(
    keyboard=[
        [KeyboardButton(text="🌍 Ob-havo (shahar nomi bilan)")],
        [KeyboardButton(text="📍 Mening joylashuvim", request_location=True)],
    ],
    resize_keyboard=True
)


@dp.message(Command("start"))
async def start(message: Message):
    await message.answer(
        "Salom! Men ob-havo ma’lumotlarini ko‘rsataman.\n\nShahar nomini yozing yoki 📍 joylashuvingizni yuboring.",
        reply_markup=start_keyboard
    )


@dp.message(F.location)
async def weather_by_location(message: Message):
    lat = message.location.latitude
    lon = message.location.longitude
    url = f"http://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={WEATHER_API_KEY}&units=metric&lang=ru"
    await send_weather(message, url)


@dp.message(F.text)
async def weather_by_city(message: Message):
    city = message.text.strip()
    if city.startswith("/") or city.startswith("📍") or city.startswith("🌍"):
        return
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={WEATHER_API_KEY}&units=metric&lang=ru"
    await send_weather(message, url)


async def send_weather(message: Message, url: str):
    try:
        response = requests.get(url)
        data = response.json()

        if data.get("cod") != 200:
            await message.answer("❌ Ob-havo topilmadi. Iltimos, boshqa shaharni sinab ko‘ring.")
            return

        city = data["name"]
        temp = data["main"]["temp"]
        desc = data["weather"][0]["description"]
        humidity = data["main"]["humidity"]
        wind = data["wind"]["speed"]

        reply = (
            f"📍 *{city}* uchun ob-havo:\n"
            f"🌡 Temperatura: {temp}°C\n"
            f"☁ Holati: {desc}\n"
            f"💧 Namlik: {humidity}%\n"
            f"💨 Shamol: {wind} m/s"
        )

        await message.answer(reply, parse_mode="Markdown")

    except Exception as e:
        await message.answer("⚠ Xatolik yuz berdi. Iltimos, keyinroq urinib ko‘ring.")

        
async def main():
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())

#tg/ o1_sanco
