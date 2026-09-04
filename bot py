import requests
import json
import time

BOT_TOKEN = "8717325788:AAEir96MaRhstNlAPtXe2AWjjoXV2ZSdar4"
EXTERNAL_API_URL = "https://core.telegram.org/bots/api"

BASE_URL = "https://api.telegram.org/bot" + BOT_TOKEN

keyboard = {
    "keyboard": [
        ["📱 Phone Lookup"]
    ],
    "resize_keyboard": True
}


def send_message(chat_id, text, reply_markup=None):
    url = BASE_URL + "/sendMessage"

    data = {
        "chat_id": chat_id,
        "text": text
    }

    if reply_markup:
        data["reply_markup"] = json.dumps(reply_markup)

    try:
        requests.post(url, data=data, timeout=20)
    except requests.RequestException:
        pass


def get_updates(offset):
    url = BASE_URL + "/getUpdates"

    params = {
        "offset": offset,
        "timeout": 30
    }

    try:
        response = requests.get(url, params=params, timeout=35)
        return response.json()
    except (requests.RequestException, ValueError):
        return None


def phone_lookup(chat_id, phone_number):
    if len(phone_number) != 10 or not phone_number.isdigit():
        send_message(
            chat_id,
            "❌ Invalid input.\n\nPlease send a valid 10-digit numeric mobile number."
        )
        return

    if not EXTERNAL_API_URL:
        send_message(
            chat_id,
            "⚠️ External API URL is not configured."
        )
        return

    try:
        # Adjust the parameter name if your external API expects something different.
        response = requests.get(
            EXTERNAL_API_URL,
            params={"phone": phone_number},
            timeout=20
        )

        response.raise_for_status()

        # Convert API response into JSON.
        api_data = response.json()

        # Pretty-format the JSON.
        formatted_json = json.dumps(
            api_data,
            indent=2,
            ensure_ascii=False
        )

        # Telegram messages have a size limit, so avoid sending an
        # excessively large response as one message.
        if len(formatted_json) > 3900:
            formatted_json = formatted_json[:3900] + "\n..."

        send_message(
            chat_id,
            "<pre>" + formatted_json + "</pre>"
        )

    except (requests.RequestException, ValueError) as error:
        send_message(
            chat_id,
            "❌ Could not retrieve the lookup result.\n"
            "Please try again later."
        )


def main():
    offset = 0

    print("Bot is running...")

    while True:
        updates = get_updates(offset)

        if not updates:
            time.sleep(2)
            continue

        if not updates.get("ok"):
            time.sleep(2)
            continue

        for update in updates.get("result", []):
            offset = update["update_id"] + 1

            message = update.get("message")

            if not message:
                continue

            chat = message.get("chat", {})
            chat_id = chat.get("id")

            text = message.get("text", "")

            if not chat_id:
                continue

            if text == "/start":
                welcome_message = (
                    "👋 Welcome!\n\n"
                    "Choose an option from the keyboard below."
                )

                send_message(
                    chat_id,
                    welcome_message,
                    keyboard
                )

            elif text == "📱 Phone Lookup":
                send_message(
                    chat_id,
                    "📞 Send 10 digit mobile number:"
                )

            elif text.isdigit():
                phone_lookup(chat_id, text)

            else:
                send_message(
                    chat_id,
                    "❌ Invalid input.\n\n"
                    "Please use the button below or send a 10-digit "
                    "numeric mobile number.",
                    keyboard
                )


if __name__ == "__main__":
    main()
