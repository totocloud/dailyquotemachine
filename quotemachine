import epd2in13_V4 as display  # For 2.13inch screen, import V3 if that's your model
from PIL import Image, ImageDraw, ImageFont
import time
import datetime
import textwrap
import json
import logging

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Constants
FONT_PATH_QUOTE = '/usr/share/fonts/truetype/JosefinSans/JosefinSans-ExtraLight.ttf'
FONT_PATH_AUTHOR = '/usr/share/fonts/truetype/JosefinSans/JosefinSans-ExtraLight.ttf'
QUOTE_FONT_SIZE = 18
AUTHOR_FONT_SIZE = 16
ERROR_FONT_SIZE = 16
DISPLAY_WIDTH, DISPLAY_HEIGHT = 250, 122  # Update with your actual display dimensions
QUOTE_WRAP_WIDTH = 28
ERROR_WRAP_WIDTH = 32
DISPLAY_ROTATION = 180
SPECIFIC_TIME = datetime.time(7, 20, 0)  # Adjust to your desired time

# Initialize the display
epd = display.EPD()
epd.init()

def get_daily_quote(quote_file='quotes.json'):
    """Fetch the daily quote from a JSON file."""
    try:
        with open(quote_file, 'r') as file:
            quotes = json.load(file)
        if not quotes:
            raise Exception("No quotes found in the file.")
        
        today = datetime.date.today()
        quote_index = today.toordinal() % len(quotes)
        return quotes[quote_index]
    except Exception as e:
        logging.error(f"Failed to get the daily quote: {e}")
        raise

def load_font(font_path, size):
    """Load a TTF font."""
    try:
        return ImageFont.truetype(font_path, size)
    except IOError as e:
        logging.error(f"Failed to load font: {e}")
        raise

def display_text(lines, y_start, draw, font):
    """Display wrapped text lines on the image."""
    y = y_start
    for line in lines:
        draw.text((5, y), line, font=font, fill=0)
        y += font.size + 2  # Adjust line height if needed
    return y

def display_quote(quote):
    """Display the daily quote on the e-ink screen."""
    image = Image.new('1', (DISPLAY_WIDTH, DISPLAY_HEIGHT), 255)
    draw = ImageDraw.Draw(image)

    font_quote = load_font(FONT_PATH_QUOTE, QUOTE_FONT_SIZE)
    font_author = load_font(FONT_PATH_AUTHOR, AUTHOR_FONT_SIZE)

    quote_lines = textwrap.wrap(quote["quote"], width=QUOTE_WRAP_WIDTH)
    y = display_text(quote_lines, 10, draw, font_quote)

    draw.text((5, y + 10), f"- {quote['author']}", font=font_author, fill=0)

    image = image.rotate(DISPLAY_ROTATION, expand=1)
    epd.display(epd.getbuffer(image))

def display_error(e):
    """Display an error message on the e-ink screen."""
    logging.error(f"Showing error message on display: {e}")
    epd.Clear()

    image = Image.new('1', (DISPLAY_WIDTH, DISPLAY_HEIGHT), 255)
    draw = ImageDraw.Draw(image)

    font_error = load_font(FONT_PATH_QUOTE, ERROR_FONT_SIZE)
    error_lines = textwrap.wrap(str(e) + ", try again in 60s.", width=ERROR_WRAP_WIDTH)
    display_text(error_lines, 10, draw, font_error)

    image = image.rotate(DISPLAY_ROTATION, expand=1)
    epd.display(epd.getbuffer(image))

def calculate_sleep_time():
    """Calculate the time difference until the specific wake-up time."""
    current_time = datetime.datetime.now()
    tomorrow = datetime.date.today() + datetime.timedelta(days=1)
    next_wake_time = datetime.datetime.combine(tomorrow, SPECIFIC_TIME)
    time_difference = (next_wake_time - current_time).total_seconds()
    logging.info(f"Sleeping until {next_wake_time}")
    return time_difference

def check_and_call():
    """Clear the display, show the new quote, and handle sleep."""
    try:
        epd.Clear()
        quote = get_daily_quote()
        display_quote(quote)
        epd.sleep()
        time.sleep(calculate_sleep_time())
    except Exception as e:
        display_error(e)
        time.sleep(60)  # Retry after 60 seconds in case of error

# Main loop
if __name__ == "__main__":
    while True:
        check_and_call()
