# Bot_Alert

import discord
import asyncio
from datetime import datetime, time, timedelta

TOKEN = "MTM0NjAzNDIzODYxNjc2ODYxMg.GCkJL7.2YVIj4rIGBTegmglcVepqT_Gf9kpN-ady32a0A"  # ใส่ Token บอทของคุณ
CHANNEL_ID = 1331102196514816031  # ใส่ ID ของช่องที่ต้องการให้แจ้งเตือน

intents = discord.Intents.default()
client = discord.Client(intents=intents)

# กำหนดเวลาที่ต้องการให้แจ้งเตือน
ALERT_TIMES = [
    time(8, 30),
    time(12, 0),
    time(13, 0),
    time(17, 25)
]

async def send_alert():
    await client.wait_until_ready()
    channel = client.get_channel(CHANNEL_ID)

    while not client.is_closed():
        now = datetime.now().time()
        next_alert = None

        # หาเวลาถัดไปที่ต้องแจ้งเตือน
        for alert_time in ALERT_TIMES:
            if now < alert_time:
                next_alert = alert_time
                break

        # ถ้าเวลาตอนนี้เกินเวลาทั้งหมดแล้ว ให้รอข้ามไปวันถัดไป
        if not next_alert:
            next_alert = ALERT_TIMES[0]
            wait_seconds = (datetime.combine(datetime.today() + timedelta(days=1), next_alert) - datetime.now()).seconds
        else:
            wait_seconds = (datetime.combine(datetime.today(), next_alert) - datetime.now()).seconds

        print(f"⏳ รอ {wait_seconds} วินาที จนถึง {next_alert.strftime('%H:%M')} เพื่อส่งแจ้งเตือน")
        await asyncio.sleep(wait_seconds)

        # ส่งข้อความแจ้งเตือน
        await channel.send(f"📌 **อย่าลืมส่งตำแหน่งของคุณ!**\n⏰ เวลา: {next_alert.strftime('%H:%M')} น.")

@client.event
async def on_ready():
    print(f'✅ บอทล็อกอินแล้ว: {client.user}')
    client.loop.create_task(send_alert())

client.run(TOKEN)
