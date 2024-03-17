# Music-0099 import os
import youtube_dl
from telegram.ext import Updater, CommandHandler
from pydub import AudioSegment
from pydub.playback import play

# Replace 'your_telegram_bot_token' with your actual bot token
TOKEN = "your_telegram_bot_token"

# Create an updater object
updater = Updater(token=TOKEN, use_context=True)
dispatcher = updater.dispatcher

# Define the start command handler
def start(update, context):
    context.bot.send_message(chat_id=update.effective_chat.id, text="I'm a music bot! Send me the name of a song to play.")

# Define the play command handler
def play_song(update, context):
    query = ' '.join(context.args)
    ydl_opts = {
        'format': 'bestaudio/best',
        'postprocessors': [{
            'key': 'FFmpegExtractAudio',
            'preferredcodec': 'mp3',
            'preferredquality': '192',
        }],
        'outtmpl': 'songs/%(title)s.%(ext)s',
    }

    with youtube_dl.YoutubeDL(ydl_opts) as ydl:
        info = ydl.extract_info(f"ytsearch1:{query}", download=False)
        song_url = info['entries'][0]['url']
        context.bot.send_message(chat_id=update.effective_chat.id, text=f"Now playing: {info['entries'][0]['title']}")
        os.system(f"ffmpeg -i $(youtube-dl -f bestaudio -g '{song_url}') -acodec pcm_s16le -ar 44100 -ac 2 -f wav - | ffmpeg -i pipe: -ac 2 -f mp3 pipe: > songs/temp.mp3")
        song = AudioSegment.from_mp3("songs/temp.mp3")
        play(song)

# Add command handlers to the dispatcher
start_handler = CommandHandler('start', start)
play_handler = CommandHandler('play', play_song)
dispatcher.add_handler(start_handler)
dispatcher.add_handler(play_handler)

# Start the bot
updater.start_polling()
updater.idle()
