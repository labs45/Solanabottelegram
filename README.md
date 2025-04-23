# Solanabottelegram
import os
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext
from solana.rpc.api import Client
from solana.keypair import Keypair
from solana.publickey import PublicKey
import json
import requests

# --- Telegram Bot Configuration ---
TELEGRAM_BOT_TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN")  # Retrieve token from environment variable

# --- Solana Configuration ---
SOLANA_RPC_URL = "https://api.devnet.solana.com"  # Use devnet for development
solana_client = Client(SOLANA_RPC_URL)

# --- Solana Functions ---
def get_solana_balance(public_key: str) -> float:
    """Gets the SOL balance of a wallet address."""
    try:
        balance = solana_client.get_balance(PublicKey(public_key))["result"]["value"]
        return balance / 10**9  # Convert lamports to SOL
    except Exception as e:
        return f"Failed to get balance: {e}"

def get_solana_price() -> float:
    """Gets the current SOL price from CoinGecko."""
    try:
        url = "https://api.coingecko.com/api/v3/simple/price?ids=solana&vs_currencies=usd"  # Change 'usd' as needed
        response = requests.get(url)
        response.raise_for_status()  # Raise an exception for bad status codes
        data = response.json()
        return data["solana"]["usd"]
    except requests.exceptions.RequestException as e:
        return f"Failed to get SOL price: {e}"
    except (KeyError, TypeError) as e:
        return f"Failed to process SOL price data: {e}"

def create_new_wallet() -> tuple[str, str]:
    """Creates a new Solana wallet (FOR DEMO PURPOSES ONLY - DO NOT USE FOR REAL FUNDS)."""
    keypair = Keypair()
    public_key = str(keypair.public_key)
    private_key = keypair.secret_key().hex()
    return public_key, private_key

# --- Telegram Bot Handlers ---
def start(update: Update, context: CallbackContext) -> None:
    """Handles the /start command."""
    update.message.reply_text("Welcome! Use commands /balance [wallet_address], /price, or /newwallet.")

def balance(update: Update, context: CallbackContext) -> None:
    """Handles the /balance command to display the balance."""
    if context.args:
        public_key = context.args[0]
        balance = get_solana_balance(public_key)
        update.message.reply_text(f"Balance for address {public_key}: {balance} SOL")
    else:
        update.message.reply_text("Please include a wallet address. Example: /balance [wallet_address]")

def price(update: Update, context: CallbackContext) -> None:
    """Handles the /price command to display the SOL price."""
    price = get_solana_price()
    update.message.reply_text(f"Current Solana price (USD): ${price}")

def newwallet(update: Update, context: CallbackContext) -> None:
    """Handles the /newwallet command (FOR DEMO PURPOSES ONLY - DO NOT USE FOR REAL FUNDS)."""
    public_key, private_key = create_new_wallet()
    update.message.reply_text(f"New Wallet Address (Public Key): {public_key}\nPrivate Key (KEEP IT SAFE!): {private_key}")
    update.message.reply_text("WARNING: This is for demonstration purposes ONLY. Do not use this private key to store real funds without proper security!")

def main() -> None:
    """Starts the bot."""
    updater = Updater(TELEGRAM_BOT_TOKEN, use_context=True)

    dispatcher = updater.dispatcher

    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("balance", balance))
    dispatcher.add_handler(CommandHandler("price", price))
    dispatcher.add_handler(CommandHandler("newwallet", newwallet))

    updater.start_polling()

    updater.idle()

if __name__ == "__main__":
    main()
