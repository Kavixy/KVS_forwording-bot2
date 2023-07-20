from telegram import Update, ParseMode
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, ConversationHandler, CallbackContext

BOT_TOKEN = '6040598338:AAFCBaJVTF8hSajKr7G43pfvW1UY6BPHFN8'

user_config = {-1001983173210}

SETUP_SOURCE_CHANNELS, SETUP_DESTINATION_CHANNELS = range(2)

def start(update: Update, _: CallbackContext) -> int:
    update.message.reply_text(
        "Hi! I am your auto-forwarding bot. Use /set_source to add source channels and /set_destination to add destination channels."
    )
    return ConversationHandler.END

def set_source_channels(update: Update, _: CallbackContext) -> int:
    update.message.reply_text(
        "Send me the chat ID of the source channels one by one. Type /done when you're finished."
    )
    return SETUP_SOURCE_CHANNELS

def set_destination_channels(update: Update, _: CallbackContext) -> int:
    update.message.reply_text(
        "Send me the chat ID of the destination channels one by one. Type /done when you're finished."
    )
    return SETUP_DESTINATION_CHANNELS

def add_source_channel(update: Update, _: CallbackContext) -> int:
    chat_id = update.message.text
    if chat_id == "/done":
        return ConversationHandler.END

    user_config.setdefault(update.message.chat_id, {"source_channels": [], "destination_channels": []})
    user_config[update.message.chat_id]["source_channels"].append(int(chat_id))
    update.message.reply_text(f"Source channel {chat_id} added. Send another chat ID or type /done.")
    return SETUP_SOURCE_CHANNELS

def add_destination_channel(update: Update, _: CallbackContext) -> int:
    chat_id = update.message.text
    if chat_id == "/done":
        return ConversationHandler.END

    user_config[update.message.chat_id]["destination_channels"].append(int(chat_id))
    update.message.reply_text(f"Destination channel {chat_id} added. Send another chat ID or type /done.")
    return SETUP_DESTINATION_CHANNELS

def auto_forward(update: Update, _: CallbackContext):
    source_channels = user_config.get(update.message.chat_id, {}).get("source_channels", [])
    destination_channels = user_config.get(update.message.chat_id, {}).get("destination_channels", [])
    if not source_channels or not destination_channels:
        update.message.reply_text("Please set up source and destination channels using /set_source and /set_destination commands.")
        return

    sender = update.message.from_user
    message_text = update.message.text

    for dest_channel_id in destination_channels:
        for source_channel_id in source_channels:
            try:
                update.message.bot.send_message(dest_channel_id, f"{sender.username} said:\n{message_text}", parse_mode=ParseMode.MARKDOWN)
            except Exception as e:
                print(f"Error forwarding message: {e}")

def main():
    updater = Updater(BOT_TOKEN)
    dispatcher = updater.dispatcher

    conv_handler = ConversationHandler(
        entry_points=[CommandHandler('start', start)],
        states={
            SETUP_SOURCE_CHANNELS: [CommandHandler('set_source', set_source_channels)],
            SETUP_DESTINATION_CHANNELS: [CommandHandler('set_destination', set_destination_channels)],
        },
        fallbacks=[],
    )
    dispatcher.add_handler(conv_handler)

    add_source_handler = MessageHandler(Filters.text & ~Filters.command, add_source_channel)
    add_destination_handler = MessageHandler(Filters.text & ~Filters.command, add_destination_channel)

    dispatcher.add_handler(add_source_handler)
    dispatcher.add_handler(add_destination_handler)
    dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, auto_forward))

    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
