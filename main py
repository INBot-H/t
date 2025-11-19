import os
import base64
import requests
from flask import Flask, request

app = Flask(__name__)

BOT_TOKEN = os.getenv("BOT_TOKEN")
IMGBB_KEY = os.getenv("IMGBB_KEY")
TELEGRAM_API = f"https://api.telegram.org/bot{BOT_TOKEN}"


@app.route("/")
def home():
    return "Image Hosting Bot Running!", 200


@app.route(f"/webhook/{os.getenv('BOT_TOKEN')}", methods=["POST"])
def webhook():
    data = request.get_json()

    if not data or "message" not in data:
        return "OK", 200

    msg = data["message"]
    chat_id = msg["chat"]["id"]

    # ------------- START COMMAND -------------
    text = msg.get("text", "")
    first_name = msg["from"].get("first_name", "User")

    if text == "/start":
        welcome_message = (
            f"👋 Hello *{first_name}!* \n\n"
             "📸 *Welcome to the Professional Image Hosting Bot!*\n\n"
             "Send me any photo and I'll host it on ImgBB and return a direct link.\n\n"
            
             "Send your first image now!"
          
        )

        requests.post(f"{TELEGRAM_API}/sendMessage", json={
            "chat_id": chat_id,
            "text": welcome_message,
            "parse_mode": "Markdown"
        })

        return "OK", 200

    # ------------- IF USER SENDS PHOTO -------------
    if "photo" in msg:
        file_id = msg["photo"][-1]["file_id"]

        # Get file path
        file_info = requests.get(f"{TELEGRAM_API}/getFile?file_id={file_id}").json()
        file_path = file_info["result"]["file_path"]

        # Download file
        file_url = f"https://api.telegram.org/file/bot{BOT_TOKEN}/{file_path}"
        img_bytes = requests.get(file_url).content

        # Encode to base64
        b64_image = base64.b64encode(img_bytes).decode()

        # Upload to ImgBB
        r = requests.post(
            "https://api.imgbb.com/1/upload",
            data={"key": IMGBB_KEY, "image": b64_image}
        )

        if r.status_code != 200:
            requests.post(f"{TELEGRAM_API}/sendMessage", json={
                "chat_id": chat_id,
                "text": "❌ Upload failed!"
            })
            return "OK", 200

        url = r.json()["data"]["url"]

        # Send hosted link
        requests.post(f"{TELEGRAM_API}/sendMessage", json={
            "chat_id": chat_id,
            "text": f"✅ *Uploaded Successfully!*\n\n📸 Link:\n{url}",
            "parse_mode": "Markdown"
        })

    else:
        requests.post(f"{TELEGRAM_API}/sendMessage", json={
            "chat_id": chat_id,
            "text": "Send an image to host it!"
        })

    return "OK", 200


def set_webhook():
    domain = os.getenv("RENDER_EXTERNAL_HOSTNAME")
    if not domain:
        print("Waiting for Render hostname...")
        return

    webhook_url = f"https://{domain}/webhook/{BOT_TOKEN}"
    requests.get(f"{TELEGRAM_API}/setWebhook?url={webhook_url}")

    print("Webhook Set:", webhook_url)


if __name__ == "__main__":
    set_webhook()
    port = int(os.environ.get("PORT", 10000))
    app.run(host="0.0.0.0", port=port)
