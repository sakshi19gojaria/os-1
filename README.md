import requests
from bs4 import BeautifulSoup
import threading
from queue import Queue

# Set up the initial parameters
base_url = "https://example.com"  # Replace with your starting URL
visited_urls = set()
thread_count = 4  # Number of threads

# Create a queue to hold URLs to be crawled
url_queue = Queue()

# Add the initial URL to the queue
url_queue.put(base_url)

# Lock to ensure thread-safe access to visited_urls set
visited_lock = threading.Lock()

# Function to fetch and parse URLs
def fetch_and_parse_url(url):
    global visited_urls

    try:
        response = requests.get(url, timeout=10)
    except requests.RequestException as e:
        print(f"Error fetching {url}: {e}")
        return

    if response.status_code != 200:
        print(f"Failed to fetch {url}")
        return

    soup = BeautifulSoup(response.text, 'html.parser')

    # Process the content here (e.g., extract links, data, etc.)
    # For demonstration, we'll just extract and print title of the page
    title = soup.title.string if soup.title else "No title"
    print(f"Title of {url}: {title}")

    # Extract links from the page
    for link in soup.find_all('a', href=True):
        absolute_link = link['href']
        if absolute_link.startswith('/'):  # Relative URLs
            absolute_link = base_url + absolute_link
        elif not absolute_link.startswith(base_url):  # Skip external URLs
            continue

        with visited_lock:
            if absolute_link not in visited_urls:
                visited_urls.add(absolute_link)
                url_queue.put(absolute_link)

# Function to manage the crawling process
def crawl():
    while not url_queue.empty():
        url = url_queue.get()
        fetch_and_parse_url(url)

# Create and start the threads
threads = []
for _ in range(thread_count):
    thread = threading.Thread(target=crawl)
    thread.start()
    threads.append(thread)

# Wait for all threads to complete
for thread in threads:
    thread.join()

print("Crawling completed.")
