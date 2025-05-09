## File Structure
```
telegram-chatbot/
├── telegram_chatbot.py        # Main bot implementation
├── requirements.txt          # Python dependencies
├── README.md                 # Project overview & setup instructions
└── .gitignore                # Files & dirs to ignore in Git
```

---

## telegram_chatbot.py
```python
import os
import logging
from telegram import Update
from telegram.ext import ApplicationBuilder, ContextTypes, CommandHandler, MessageHandler, filters
import openai

def setup_logging():
    logging.basicConfig(
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        level=logging.INFO
    )

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Send a welcome message when the /start command is issued."""
    await update.message.reply_text(
        "Hello! I'm your chat bot. Send me a message and I'll reply!"
    )

async def chat(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Handle incoming messages and respond via OpenAI ChatCompletion."""
    user_message = update.message.text
    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": user_message}],
            max_tokens=150,
            n=1,
            temperature=0.7,
        )
        reply_text = response.choices[0].message.content.strip()
    except Exception as e:
        logging.error(f"OpenAI API error: {e}")
        reply_text = "Sorry, I couldn't process that."

    await update.message.reply_text(reply_text)

async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    """Send a help message when the /help command is issued."""
    await update.message.reply_text(
        '/start - Start the bot\n'
        '/help - Show this help message\n'
        'Just send any message to chat with me.'
    )


def main() -> None:
    setup_logging()

    telegram_token = os.getenv('TELEGRAM_TOKEN')
    openai.api_key = os.getenv('OPENAI_API_KEY')

    if not telegram_token or not openai.api_key:
        logging.error("Missing TELEGRAM_TOKEN or OPENAI_API_KEY environment variable.")
        return

    app = ApplicationBuilder().token(telegram_token).build()
    app.add_handler(CommandHandler('start', start))
    app.add_handler(CommandHandler('help', help_command))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, chat))

    logging.info("Bot started...")
    app.run_polling()

if __name__ == '__main__':
    main()
```

---

## requirements.txt
```
python-telegram-bot==20.3
openai>=0.27.0
```

---

## .gitignore
```
venv/
__pycache__/
*.pyc
.env
```

---

## README.md
```markdown
