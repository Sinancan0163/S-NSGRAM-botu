import requests
import time
from flask import Flask, request
import threading
import json

BOT_TOKEN = "8456855175:AAGKjqabe7PE5x6i__IHmwD6PeTaYiQLnxI"
ADMIN_CHAT_ID = "7714028503"

app = Flask(__name__)

users = {}
banned_users = []
badges = {}

def send_message(chat_id, text):
    url = f"https://api.telegram.org/bot{BOT_TOKEN}/sendMessage"
    data = {"chat_id": chat_id, "text": text, "parse_mode": "HTML"}
    try:
        requests.post(url, json=data, timeout=10)
    except:
        pass

@app.route('/')
def home():
    return "SİNSGRAM Bot Aktif! 🤖"

@app.route(f'/webhook/{BOT_TOKEN}', methods=['POST'])
def webhook():
    try:
        update = request.get_json()
        if 'message' in update:
            handle_message(update['message'])
        return 'OK', 200
    except:
        return 'Error', 500

def handle_message(message):
    chat_id = message.get('chat', {}).get('id')
    text = message.get('text', '')
    username = message.get('from', {}).get('username', 'bilinmeyen')
    
    if not text or not chat_id:
        return
    
    if text == '/start':
        send_message(chat_id, "🛡️ SİNSGRAM Bot Aktif!\n/ban @kullanici - Banla\n/verify @kullanici - Mavi tik ver")
    
    elif text.startswith('/ban '):
        target = text.replace('/ban ', '').replace('@', '').strip()
        if target:
            banned_users.append(target)
            send_message(chat_id, f"✅ @{target} BANLANDI!")
        else:
            send_message(chat_id, "❌ Kullanım: /ban @kullanici")
    
    elif text.startswith('/unban '):
        target = text.replace('/unban ', '').replace('@', '').strip()
        if target in banned_users:
            banned_users.remove(target)
            send_message(chat_id, f"✅ @{target} BANI KALDIRILDI!")
        else:
            send_message(chat_id, f"⚠️ @{target} banlı değil!")
    
    elif text.startswith('/verify '):
        target = text.replace('/verify ', '').replace('@', '').strip()
        if target:
            badges[target] = 'mavi'
            send_message(chat_id, f"🔵 @{target} MAVİ TİK VERİLDİ!")
        else:
            send_message(chat_id, "❌ Kullanım: /verify @kullanici")
    
    elif text == '/stats':
        send_message(chat_id, f"📊 İSTATİSTİKLER\n👥 Kullanıcı: {len(users)}\n⛔ Banlı: {len(banned_users)}\n🏷️ Rozetli: {len(badges)}")
    
    else:
        send_message(chat_id, "❌ Bilinmeyen komut. /start yaz.")

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
