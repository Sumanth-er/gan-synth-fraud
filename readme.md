import requests
from bs4 import BeautifulSoup
import time

BASE_URL = "https://pypi.org/search/?q=model+context+protocol+&page={}"
TOTAL_PAGES = 3  # You can increase this
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
}
DELAY = 1  # seconds

results_list = []

for page in range(1, TOTAL_PAGES + 1):
    url = BASE_URL.format(page)
    print(f"\nScraping: {url}")
    res = requests.get(url, headers=HEADERS)

    if res.status_code != 200:
        print(f"Error: Status {res.status_code}")
        break

    soup = BeautifulSoup(res.text, 'html.parser')
    packages = soup.find_all("a", class_="package-snippet")

    for pkg in packages:
        name = pkg.find("span", class_="package-snippet__name").text.strip()
        version = pkg.find("span", class_="package-snippet__version").text.strip()
        desc_tag = pkg.find("p", class_="package-snippet__description")
        description = desc_tag.text.strip() if desc_tag else ""
        link = "https://pypi.org" + pkg.get("href")

        results_list.append({
            "name": name,
            "version": version,
            "description": description,
            "link": link
        })

        print(f" - {name} ({version}): {description}")

    time.sleep(DELAY)

# Save results to file
with open("pypi_packages_detailed.txt", "w", encoding="utf-8") as f:
    for item in results_list:
        f.write(f"{item['name']} ({item['version']})\n")
        f.write(f"  Description: {item['description']}\n")
        f.write(f"  Link: {item['link']}\n\n")

print(f"\n✅ Scraped {len(results_list)} packages.")