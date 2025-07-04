from playwright.sync_api import sync_playwright
import time

BASE_URL = "https://pypi.org/search/?q=model+context+protocol+&page={}"
TOTAL_PAGES = 3
HEADLESS = False  # Set True after confirming it works

def run():
    results = []

    with sync_playwright() as p:
        # ✅ Launch real Chrome browser with anti-bot settings
        browser = p.chromium.launch(
            channel="chrome",
            headless=HEADLESS,
            args=[
                "--start-maximized",
                "--no-sandbox",
                "--disable-blink-features=AutomationControlled",  # Anti-bot
                "--disable-dev-shm-usage"
            ]
        )
        context = browser.new_context(
            user_agent=(
                "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                "AppleWebKit/537.36 (KHTML, like Gecko) "
                "Chrome/115.0.0.0 Safari/537.36"
            ),
            viewport={"width": 1920, "height": 1080},
            java_script_enabled=True,
            locale="en-US"
        )
        page = context.new_page()

        for page_no in range(1, TOTAL_PAGES + 1):
            url = BASE_URL.format(page_no)
            print(f"\n🟡 Visiting: {url}")
            page.goto(url, wait_until="networkidle")
            time.sleep(2)  # Human-like pause

            items = page.query_selector_all("a.package-snippet")

            for item in items:
                name = item.query_selector("span.package-snippet__name").inner_text().strip()
                results.append(name)
                print(f"  ✅ {name}")

        browser.close()

    with open("pypi_results.txt", "w", encoding="utf-8") as f:
        for name in results:
            f.write(name + "\n")

    print(f"\n✅ Scraped {len(results)} packages successfully.")

if __name__ == "__main__":
    run()