import sys
import os
import logging
from rich.console import Console
# === Force UTF-8 Encoding for stdout/stderr (Python 3.7+) ===
if hasattr(sys.stdout, "reconfigure"):
    try:
        sys.stdout.reconfigure(encoding="utf-8")
    except Exception:
        pass
if hasattr(sys.stderr, "reconfigure"):
    try:
        sys.stderr.reconfigure(encoding="utf-8")
    except Exception:
        pass
# === Safe Console for Windows / Any Terminal ===
class SafeConsole(Console):
    def print(self, *args, **kwargs):
        safe_args = []
        for arg in args:
            if isinstance(arg, str):
                try:
                    enc = sys.stdout.encoding or "utf-8"
                    arg.encode(enc)
                    safe_args.append(arg)
                except Exception:
                    enc = sys.stdout.encoding or "utf-8"
                    safe_args.append(arg.encode(enc, errors="replace").decode(enc))
            else:
                safe_args.append(arg)
        super().print(*safe_args, **kwargs)
console = SafeConsole()
# === Logging Setup: Safe Stream Handler ===
class SafeStreamHandler(logging.StreamHandler):
    def emit(self, record):
        try:
            msg = self.format(record)
            stream = self.stream
            try:
                stream.write(msg + self.terminator)
            except UnicodeEncodeError:
                enc = getattr(stream, "encoding", None) or "utf-8"
                stream.write(msg.encode(enc, errors="replace").decode(enc) + self.terminator)
            self.flush()
        except Exception:
            self.handleError(record)
# -------------------------
# === BEGIN ORIGINAL CODE (merged) ===
# -------------------------
import requests
import random
import time
import datetime
import pytz
import re
import socket
import subprocess
import string
from http.cookies import SimpleCookie
from urllib.parse import urlparse
import threading
import json
import hashlib
import base64
from concurrent.futures import ThreadPoolExecutor
import warnings
from cryptography.fernet import Fernet
import numpy as np
from typing import Dict, List, Tuple, Optional
import configparser
from pathlib import Path
from dataclasses import dataclass
from enum import Enum
import uuid
import platform
import psutil
from rich.panel import Panel
from io import BytesIO
from PIL import Image, ImageDraw, ImageFont
import tls_client
import math
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
# Suppress warnings
warnings.filterwarnings("ignore")
# === NEW FEATURES: Enhanced Behavioral Randomness ===
def check_proxy_quality(proxy):
    """Check if proxy is high quality and not blacklisted"""
    try:
        formatted_proxy = format_proxy(proxy)

        # Check against known blacklists
        blacklists = [
            "https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/firehol_level1.netset",
            "https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/dshield.netset"
        ]

        # Extract IP from proxy string
        if "@" in proxy:
            ip = proxy.split("@")[-1].split(":")[0]
        else:
            ip = proxy.split("://")[-1].split(":")[0]

        # Check against blacklists
        for blacklist_url in blacklists:
            try:
                response = requests.get(blacklist_url, timeout=5)
                if response.status_code == 200:
                    if ip in response.text:
                        logger.warning(f"⚠️ Proxy {proxy} is blacklisted")
                        return False
            except Exception as e:
                logger.warning(f"Error checking blacklist {blacklist_url}: {e}")
                continue

        # Check proxy type (residential vs datacenter)
        try:
            ip_info = get_ip_info(formatted_proxy)
            if not ip_info or len(ip_info) < 6:
                logger.warning(f"⚠️ Could not get IP info for proxy: {proxy}")
                return False

            isp = ip_info[5]
            if isp is None:
                logger.warning(f"⚠️ ISP information is None for proxy: {proxy}")
                return False

            isp = isp.lower()

            # Datacenter indicators
            datacenter_indicators = ["datacenter", "cloud", "hosting", "server", "digital ocean", "aws", "azure"]

            if any(indicator in isp for indicator in datacenter_indicators):
                logger.warning(f"⚠️ Proxy {proxy} appears to be datacenter: {isp}")
                return False
        except Exception as e:
            logger.error(f"Error checking proxy type: {e}")
            return False

        return True
    except Exception as e:
        logger.error(f"Error checking proxy quality: {e}")
        return False
def ensure_device_consistency(ua, device_fingerprint):
    """Ensure all device parameters are perfectly consistent"""
    device_info = device_fingerprint.device_info
    device_type = device_info.get('device_type')

    # Device-specific parameters
    device_specs = {
        "iPhone": {
            "platform": "iPhone",
            "maxTouchPoints": 5,
            "hardwareConcurrency": 6,
            "deviceMemory": 4,
            "connectionType": "cellular",
            "mobile": True
        },
        "Samsung": {
            "platform": "Linux armv8l",
            "maxTouchPoints": 5,
            "hardwareConcurrency": 8,
            "deviceMemory": 8,
            "connectionType": "cellular",
            "mobile": True
        },
        "Windows": {
            "platform": "Win32",
            "maxTouchPoints": 0,
            "hardwareConcurrency": random.randint(4, 16),
            "deviceMemory": random.choice([4, 8, 16]),
            "connectionType": random.choice(["wifi", "ethernet"]),
            "mobile": False
        }
    }

    return device_specs.get(device_type, device_specs["iPhone"])
def calculate_natural_wait_time():
    """Calculate wait times that mimic human behavior more closely"""
    current_hour = datetime.datetime.now().hour

    # Different behavior patterns by time of day
    if 6 <= current_hour <= 9:  # Morning rush
        base_wait = random.randint(180, 600)  # 3-10 minutes
    elif 12 <= current_hour <= 14:  # Lunch time
        base_wait = random.randint(300, 900)  # 5-15 minutes
    elif 17 <= current_hour <= 19:  # Evening rush
        base_wait = random.randint(240, 720)  # 4-12 minutes
    elif 20 <= current_hour <= 23:  # Prime time
        base_wait = random.randint(180, 480)  # 3-8 minutes
    else:  # Night/early morning
        base_wait = random.randint(600, 1800)  # 10-30 minutes

    # Add randomization
    wait_time = base_wait + random.randint(-60, 60)
    return max(120, wait_time)  # Minimum 2 minutes
def simulate_realistic_browsing_session(driver):
    """Simulate more realistic browsing with varying session depth"""
    session_actions = []

    # Random number of actions (3-15)
    num_actions = random.randint(3, 15)

    for _ in range(num_actions):
        action_type = random.choices(
            ["scroll", "click", "form", "wait", "navigate"],
            weights=[0.3, 0.25, 0.15, 0.2, 0.1]
        )[0]

        if action_type == "scroll":
            # Random scroll amount and speed
            scroll_amount = random.randint(100, 1000)
            scroll_speed = random.uniform(0.5, 2.0)
            driver.execute_script(f"window.scrollBy(0, {scroll_amount});")
            time.sleep(scroll_speed)

        elif action_type == "click":
            # Click on random element
            clickable_elements = driver.find_elements(By.CSS_SELECTOR, "a, button, [onclick]")
            if clickable_elements:
                element = random.choice(clickable_elements)
                try:
                    driver.execute_script("arguments[0].scrollIntoView({behavior: 'smooth'});", element)
                    time.sleep(random.uniform(0.5, 1.5))
                    element.click()
                    time.sleep(random.uniform(1, 3))
                except:
                    pass

        elif action_type == "form":
            # Interact with form if exists
            forms = driver.find_elements(By.TAG_NAME, "form")
            if forms and random.random() < 0.3:  # 30% chance
                form = random.choice(forms)
                inputs = form.find_elements(By.TAG_NAME, "input")
                if inputs:
                    input_field = random.choice(inputs)
                    try:
                        input_field.click()
                        time.sleep(random.uniform(0.2, 0.5))
                        input_field.send_keys("test")
                        time.sleep(random.uniform(0.5, 1))
                        input_field.clear()
                    except:
                        pass

        elif action_type == "wait":
            # Random wait time
            wait_time = random.uniform(2, 10)
            time.sleep(wait_time)

        elif action_type == "navigate":
            # Sometimes navigate to new page
            if random.random() < 0.2:  # 20% chance
                links = driver.find_elements(By.TAG_NAME, "a")
                if links:
                    link = random.choice(links)
                    try:
                        link.click()
                        time.sleep(random.uniform(2, 5))
                        driver.back()
                        time.sleep(random.uniform(1, 3))
                    except:
                        pass

        # Random pause between actions
        time.sleep(random.uniform(0.5, 2))
def randomize_click_pattern():
    """Randomize click patterns to avoid detection"""
    patterns = [
        "immediate",      # Click immediately
        "delayed",       # Wait before clicking
        "hesitant",      # Multiple hover actions before click
        "indirect",      # Click nearby elements first
        "scrolled",      # Scroll before clicking
    ]

    return random.choice(patterns)
def execute_click_pattern(driver, target_element, pattern):
    """Execute different click patterns"""
    try:
        if pattern == "immediate":
            target_element.click()

        elif pattern == "delayed":
            time.sleep(random.uniform(1, 3))
            target_element.click()

        elif pattern == "hesitant":
            # Hover multiple times before clicking
            actions = ActionChains(driver)
            actions.move_to_element(target_element)
            actions.pause(random.uniform(0.5, 1))
            actions.perform()

            time.sleep(random.uniform(0.2, 0.5))

            actions = ActionChains(driver)
            actions.move_to_element(target_element)
            actions.pause(random.uniform(0.3, 0.8))
            actions.click()
            actions.perform()

        elif pattern == "indirect":
            # Click nearby elements first
            nearby_elements = driver.find_elements(By.XPATH, "//*[contains(@class, 'clickable')]")
            if nearby_elements:
                for _ in range(random.randint(1, 3)):
                    element = random.choice(nearby_elements)
                    try:
                        element.click()
                        time.sleep(random.uniform(0.5, 1.5))
                    except:
                        continue
            target_element.click()

        elif pattern == "scrolled":
            # Scroll around before clicking
            for _ in range(random.randint(1, 3)):
                scroll_amount = random.randint(-200, 200)
                driver.execute_script(f"window.scrollBy(0, {scroll_amount});")
                time.sleep(random.uniform(0.5, 1))
            target_element.click()
    except Exception as e:
        logger.warning(f"Error executing click pattern: {e}")
        # Fallback to simple click
        try:
            target_element.click()
        except:
            pass
# === NEW: Quick Warmup Session ===
def quick_warmup_session(driver, max_time=120):
    """Perform a quick warmup session that completes within max_time seconds"""
    start_time = time.time()
    elapsed_time = 0

    # Randomly select 2-4 sites for quick warmup
    num_sites = random.randint(2, 4)
    selected_sites = random.sample(random_sites, num_sites)

    for site in selected_sites:
        # Check if we have time left
        elapsed_time = time.time() - start_time
        if elapsed_time >= max_time:
            break

        try:
            logger.info(f"🌐 Quick warmup visiting: {site}")
            driver.get(site)

            # Wait for page to load (reduced time)
            time.sleep(random.uniform(1, 3))

            # Quick scroll (reduced time)
            scroll_amount = random.randint(100, 500)
            driver.execute_script(f"window.scrollBy(0, {scroll_amount});")
            time.sleep(random.uniform(0.2, 0.5))

            # Random pause on site (reduced time)
            pause_time = random.uniform(5, 15)
            time.sleep(pause_time)

            # Check time again
            elapsed_time = time.time() - start_time
            if elapsed_time >= max_time:
                break

        except Exception as e:
            logger.error(f"Error during quick warmup on {site}: {e}")
            continue

    logger.info(f"✅ Quick warmup completed in {elapsed_time:.2f} seconds")
# === CONFIGURATION ===
@dataclass
class Config:
    cpa_url: str = "https://www.gn3atrk.com/2K515N3L/54SZN79/?sub1=Sobuj"
    max_clicks: int = 100
    max_clicks_per_campaign: int = 50
    max_clicks_per_publisher: int = 30
    max_threads: int = 1  # Set to 1 for simpler countdown display
    infinite_loop: bool = False
    enable_encryption: bool = True
    rotation_strategy: str = "performance"  # random, round_robin, geographic, performance
    use_headless_browser: bool = True
    enable_captcha_solver: bool = True
    captcha_service: str = "2captcha"  # 2captcha, anticaptcha
    captcha_api_key: str = ""
    residential_proxy: bool = True
    proxy_provider: str = "brightdata"  # brightdata, smartproxy, oxylabs
    proxy_username: str = ""
    proxy_password: str = ""
    enable_cookie_persistence: bool = True
    cookie_file: str = "cookies.json"
    log_level: str = "INFO"
    dynamic_device_weights: bool = True
    success_rate_threshold: float = 0.7
    max_retries: int = 3
    retry_delay: int = 5
    user_agent_pool_size: int = 100
    enable_mobile_emulation: bool = True
    browser_profile_dir: str = "browser_profiles"
    enable_ip_geolocation_matching: bool = True
    auto_update_proxies: bool = True
    proxy_update_interval: int = 3600  # seconds
    min_wait_between_clicks: int = 120  # 2 minutes in seconds (UPDATED)
    max_wait_between_clicks: int = 600  # 10 minutes in seconds (UPDATED)
    # New timing pattern settings
    enable_human_timing_patterns: bool = True
    peak_click_hours: List[int] = None  # Hours when clicks are more likely
    off_peak_multiplier: float = 0.3  # Reduced click probability during off-peak hours
    # New settings for one-time use
    remove_ua_after_use: bool = True  # Remove user agent after use
    remove_proxy_after_use: bool = True  # Remove proxy after use
    disable_webgpu: bool = True  # Disable WebGPU for better undetectability
    # New settings for enhanced undetectability
    enable_mouse_simulation: bool = True
    enable_scroll_simulation: bool = True
    enable_session_simulation: bool = True
    enable_form_interaction: bool = True
    enable_cloudflare_bypass: bool = True
    enable_tls_fingerprinting: bool = True
    enable_header_ordering: bool = True
    enable_environment_consistency: bool = True
    enable_device_consistency: bool = True
    # New advanced settings
    enable_advanced_mouse_simulation: bool = True
    enable_canvas_fingerprinting: bool = True
    enable_webrtc_protection: bool = True
    enable_header_randomization: bool = True
    enable_session_simulation: bool = True
    enable_profile_management: bool = True
    enable_network_fingerprinting: bool = True
    enable_device_consistency: bool = True
    # Advanced timing settings
    mouse_movement_speed: float = 1.0  # Speed multiplier for mouse movements
    typing_speed: float = 1.0  # Characters per second
    scroll_speed: float = 1.0  # Pixels per second
    # Session simulation settings
    min_session_duration: int = 300  # seconds
    max_session_duration: int = 1800  # seconds
    min_page_visits: int = 3
    max_page_visits: int = 15
    # New randomness settings
    min_pre_click_sites: int = 2  # Minimum sites to visit before click (UPDATED)
    max_pre_click_sites: int = 8  # Maximum sites to visit before click (UPDATED)
    human_error_probability: float = 0.3  # Probability of making a human-like error
    random_interaction_probability: float = 0.7  # Probability of random page interaction
    # NEW: Enhanced randomness settings
    enable_rest_periods: bool = True  # Enable random rest periods to simulate human breaks
    min_rest_period: int = 300  # 5 minutes minimum rest
    max_rest_period: int = 3600  # 60 minutes maximum rest
    rest_probability: float = 0.15  # 15% chance of taking a rest after a click
    # NEW: Enhanced browser settings
    enable_browser_extension_simulation: bool = True  # Simulate random browser extensions
    enable_random_viewport_sizes: bool = True  # Randomize viewport sizes
    enable_natural_click_patterns: bool = True  # More natural click timing and patterns
    # NEW: Operational security settings
    enable_operational_hours: bool = False  # Enable operational hours restriction (UPDATED)
    operating_hours_start: int = 8  # 8 AM
    operating_hours_end: int = 23  # 11 PM
    enable_success_monitoring: bool = True  # Monitor success rates
    success_rate_threshold: float = 0.7  # Threshold for success rate alert
    # NEW: Browser selection
    browser_type: str = "random"  # random, chrome, firefox, edge
    use_playwright: bool = False  # Use Playwright instead of Selenium (recommended)
    # NEW: Behavioral variance settings
    behavioral_variance: dict = None

    def __post_init__(self):
        if self.behavioral_variance is None:
            self.behavioral_variance = {
                "min_session_duration": 180,  # 3 minutes
                "max_session_duration": 3600,  # 1 hour
                "random_action_probability": 0.4,  # 40% chance of random action
                "typing_speed_variation": 0.3,  # 30% variation in typing speed
                "mouse_pause_probability": 0.2,  # 20% chance of random pause
            }
# Load configuration from file if exists
def load_config():
    config = Config()
    config_file = "bot_config.ini"

    if os.path.exists(config_file):
        parser = configparser.ConfigParser()
        parser.read(config_file)

        if 'GENERAL' in parser:
            config.cpa_url = parser.get('GENERAL', 'cpa_url', fallback=config.cpa_url)
            config.max_clicks = parser.getint('GENERAL', 'max_clicks', fallback=config.max_clicks)
            config.max_threads = parser.getint('GENERAL', 'max_threads', fallback=config.max_threads)
            config.infinite_loop = parser.getboolean('GENERAL', 'infinite_loop', fallback=config.infinite_loop)
            config.log_level = parser.get('GENERAL', 'log_level', fallback=config.log_level)

        if 'TIMING' in parser:
            config.min_wait_between_clicks = parser.getint('TIMING', 'min_wait_between_clicks', fallback=config.min_wait_between_clicks)
            config.max_wait_between_clicks = parser.getint('TIMING', 'max_wait_between_clicks', fallback=config.max_wait_between_clicks)
            config.enable_human_timing_patterns = parser.getboolean('TIMING', 'enable_human_timing_patterns', fallback=True)

            # Parse peak hours
            peak_hours_str = parser.get('TIMING', 'peak_click_hours', fallback="8,9,10,11,12,13,19,20,21,22")
            config.peak_click_hours = [int(hour) for hour in peak_hours_str.split(',')]

            config.off_peak_multiplier = parser.getfloat('TIMING', 'off_peak_multiplier', fallback=0.3)

        if 'PROXY' in parser:
            config.proxy_provider = parser.get('PROXY', 'proxy_provider', fallback=config.proxy_provider)
            config.proxy_username = parser.get('PROXY', 'proxy_username', fallback=config.proxy_username)
            config.proxy_password = parser.get('PROXY', 'proxy_password', fallback=config.proxy_password)

        if 'CAPTCHA' in parser:
            config.captcha_service = parser.get('CAPTCHA', 'captcha_service', fallback=config.captcha_service)
            config.captcha_api_key = parser.get('CAPTCHA', 'captcha_api_key', fallback=config.captcha_api_key)

        if 'USAGE' in parser:
            config.remove_ua_after_use = parser.getboolean('USAGE', 'remove_ua_after_use', fallback=True)
            config.remove_proxy_after_use = parser.getboolean('USAGE', 'remove_proxy_after_use', fallback=True)

        if 'ADVANCED' in parser:
            config.disable_webgpu = parser.getboolean('ADVANCED', 'disable_webgpu', fallback=True)

        if 'ENHANCED' in parser:
            config.enable_mouse_simulation = parser.getboolean('ENHANCED', 'enable_mouse_simulation', fallback=True)
            config.enable_scroll_simulation = parser.getboolean('ENHANCED', 'enable_scroll_simulation', fallback=True)
            config.enable_session_simulation = parser.getboolean('ENHANCED', 'enable_session_simulation', fallback=True)
            config.enable_form_interaction = parser.getboolean('ENHANCED', 'enable_form_interaction', fallback=True)
            config.enable_cloudflare_bypass = parser.getboolean('ENHANCED', 'enable_cloudflare_bypass', fallback=True)
            config.enable_tls_fingerprinting = parser.getboolean('ENHANCED', 'enable_tls_fingerprinting', fallback=True)
            config.enable_header_ordering = parser.getboolean('ENHANCED', 'enable_header_ordering', fallback=True)
            config.enable_environment_consistency = parser.getboolean('ENHANCED', 'enable_environment_consistency', fallback=True)
            config.enable_device_consistency = parser.getboolean('ENHANCED', 'enable_device_consistency', fallback=True)

        if 'ADVANCED' in parser:
            config.enable_advanced_mouse_simulation = parser.getboolean('ADVANCED', 'enable_advanced_mouse_simulation', fallback=True)
            config.enable_canvas_fingerprinting = parser.getboolean('ADVANCED', 'enable_canvas_fingerprinting', fallback=True)
            config.enable_webrtc_protection = parser.getboolean('ADVANCED', 'enable_webrtc_protection', fallback=True)
            config.enable_header_randomization = parser.getboolean('ADVANCED', 'enable_header_randomization', fallback=True)
            config.enable_session_simulation = parser.getboolean('ADVANCED', 'enable_session_simulation', fallback=True)
            config.enable_profile_management = parser.getboolean('ADVANCED', 'enable_profile_management', fallback=True)
            config.enable_network_fingerprinting = parser.getboolean('ADVANCED', 'enable_network_fingerprinting', fallback=True)
            config.enable_device_consistency = parser.getboolean('ADVANCED', 'enable_device_consistency', fallback=True)

        # NEW: Load enhanced settings
        if 'ENHANCED_RANDOMNESS' in parser:
            config.enable_rest_periods = parser.getboolean('ENHANCED_RANDOMNESS', 'enable_rest_periods', fallback=True)
            config.min_rest_period = parser.getint('ENHANCED_RANDOMNESS', 'min_rest_period', fallback=300)
            config.max_rest_period = parser.getint('ENHANCED_RANDOMNESS', 'max_rest_period', fallback=3600)
            config.rest_probability = parser.getfloat('ENHANCED_RANDOMNESS', 'rest_probability', fallback=0.15)
            config.max_pre_click_sites = parser.getint('ENHANCED_RANDOMNESS', 'max_pre_click_sites', fallback=8)  # UPDATED

        if 'BROWSER_SETTINGS' in parser:
            config.enable_browser_extension_simulation = parser.getboolean('BROWSER_SETTINGS', 'enable_browser_extension_simulation', fallback=True)
            config.enable_random_viewport_sizes = parser.getboolean('BROWSER_SETTINGS', 'enable_random_viewport_sizes', fallback=True)
            config.enable_natural_click_patterns = parser.getboolean('BROWSER_SETTINGS', 'enable_natural_click_patterns', fallback=True)
            config.browser_type = parser.get('BROWSER_SETTINGS', 'browser_type', fallback='random')
            config.use_playwright = parser.getboolean('BROWSER_SETTINGS', 'use_playwright', fallback=False)

        if 'OPERATIONAL_SECURITY' in parser:
            config.enable_operational_hours = parser.getboolean('OPERATIONAL_SECURITY', 'enable_operational_hours', fallback=True)
            config.operating_hours_start = parser.getint('OPERATIONAL_SECURITY', 'operating_hours_start', fallback=8)
            config.operating_hours_end = parser.getint('OPERATIONAL_SECURITY', 'operating_hours_end', fallback=23)
            config.enable_success_monitoring = parser.getboolean('OPERATIONAL_SECURITY', 'enable_success_monitoring', fallback=True)
            config.success_rate_threshold = parser.getfloat('OPERATIONAL_SECURITY', 'success_rate_threshold', fallback=0.7)

    # Set default peak hours if not configured
    if config.peak_click_hours is None:
        config.peak_click_hours = [8, 9, 10, 11, 12, 13, 19, 20, 21, 22]

    return config
config = load_config()
# Setup logging (file UTF-8, safe console)
logging.basicConfig(
    level=getattr(logging, config.log_level),
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler("bot.log", encoding="utf-8"),
        SafeStreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger(__name__)
# === Generate encryption key if enabled ===
if config.enable_encryption:
    encryption_key_file = "encryption_key.key"
    if os.path.exists(encryption_key_file):
        with open(encryption_key_file, "rb") as f:
            encryption_key = f.read()
    else:
        encryption_key = Fernet.generate_key()
        with open(encryption_key_file, "wb") as f:
            f.write(encryption_key)
    cipher_suite = Fernet(encryption_key)
else:
    cipher_suite = None
# === NEW: Load referers from file ===
def load_referers():
    """Load referers from referer.txt file"""
    referers = []
    referer_file = "referer.txt"

    if os.path.exists(referer_file):
        try:
            with open(referer_file, "r", encoding="utf-8") as f:
                for line in f:
                    line = line.strip()
                    if line and not line.startswith("#"):  # Skip empty lines and comments
                        referers.append(line)
            logger.info(f"✅ Loaded {len(referers)} referers from {referer_file}")
        except Exception as e:
            logger.error(f"❌ Error loading referers: {e}")
            # Fallback to default referer
            referers = ["https://www.google.com/"]
            logger.info("Using default referer: https://www.google.com/")
    else:
        # Create referer.txt with default referer if it doesn't exist
        try:
            with open(referer_file, "w", encoding="utf-8") as f:
                f.write("# Add one referer per line\n")
                f.write("# Lines starting with # are comments\n")
                f.write("https://www.google.com/\n")
            logger.info(f"✅ Created {referer_file} with default referer")
            referers = ["https://www.google.com/"]
        except Exception as e:
            logger.error(f"❌ Error creating referer file: {e}")
            referers = ["https://www.google.com/"]

    return referers
# === Load referers ===
referers = load_referers()
# === NEW FEATURE: Detect Wi-Fi Adapter Name Automatically ===
def get_wifi_adapter_name():
    try:
        result = subprocess.run(["netsh", "interface", "show", "interface"], capture_output=True, text=True)
        lines = result.stdout.splitlines()
        for line in lines:
            if "Wi-Fi" in line or "Wireless" in line:
                return line.split()[-1]
    except:
        pass
    return "Wi-Fi"  # fallback default
wifi_adapter_name = get_wifi_adapter_name()
logger.info(f"📡 Detected Wi-Fi adapter: {wifi_adapter_name}")
# === ENHANCED: MAC Changer (Windows) - TMAC Style ===
def random_mac():
    hex_digits = "0123456789ABCDEF"
    mac = "02"
    for i in range(5):
        mac += "-" + random.choice(hex_digits) + random.choice(hex_digits)
    return mac
# === NEW MAC ADDRESS FUNCTIONS ===
def get_current_mac(adapter_name):
    """Get the current MAC address of the specified network adapter"""
    try:
        # Try using ipconfig command as an alternative
        result = subprocess.run(
            ["ipconfig", "/all"],
            capture_output=True, text=True, check=True
        )
        lines = result.stdout.splitlines()
        current_adapter = None
        for line in lines:
            if adapter_name.lower() in line.lower():
                current_adapter = line.strip()
            elif current_adapter and "Physical Address" in line:
                mac = line.split(":")[-1].strip()
                return mac

        # If not found with ipconfig, try getmac
        result = subprocess.run(
            ["getmac", "/fo", "list", "/v", "/nh"],
            capture_output=True, text=True, check=True
        )
        lines = result.stdout.splitlines()
        for i, line in enumerate(lines):
            if adapter_name in line:
                # Look for the MAC address in the next few lines
                for j in range(i+1, min(i+5, len(lines))):
                    if "Physical Address" in lines[j]:
                        mac = lines[j].split(":")[-1].strip()
                        return mac
        return None
    except Exception as e:
        logger.error(f"Error getting MAC address: {e}")
        return None
def change_mac_windows(adapter_name):
    """Change MAC address and return the new MAC if successful"""
    mac = random_mac()
    try:
        logger.info(f"🔄 Changing MAC address to {mac}...")

        # Disconnect from the network
        logger.info("📡 Disconnecting from network...")
        subprocess.run(["netsh", "interface", "set", "interface", adapter_name, "admin=disable"], check=True)
        time.sleep(2)

        # Change the MAC address in the registry
        logger.info("🔧 Updating registry with new MAC address...")
        subprocess.run(["reg", "add",
                        r"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0001",
                        "/v", "NetworkAddress", "/d", mac, "/f"], check=True)

        # Reconnect to the network
        logger.info("📡 Reconnecting to network...")
        subprocess.run(["netsh", "interface", "set", "interface", adapter_name, "admin=enable"], check=True)

        # Wait for the connection to be established
        logger.info("⏳ Waiting for connection to establish...")
        time.sleep(10)

        # Verify the connection is active
        result = subprocess.run(["netsh", "interface", "show", "interface", adapter_name], capture_output=True, text=True)
        if "Connected" in result.stdout or "Connecting" in result.stdout:
            logger.info(f"✅ MAC address changed to {mac} and reconnected successfully")
            return mac
        else:
            logger.warning(f"⚠️ MAC address changed to {mac} but connection status is unknown")
            return mac
    except Exception as e:
        logger.error(f"⚠️ MAC change failed: {e}")
        return None
def change_device_mac_address():
    """Change the device MAC address without requiring reboot"""
    mac = random_mac()
    try:
        logger.info(f"🔄 Changing device MAC address to {mac}...")

        # Change the MAC address in the registry
        subprocess.run(["reg", "add",
                        r"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}\0001",
                        "/v", "NetworkAddress", "/d", mac, "/f"], check=True)

        # Reset the network adapter to apply the new MAC without reboot
        logger.info("🔄 Resetting network adapter to apply new MAC address...")

        # Disable and re-enable the network adapter
        result = subprocess.run(["netsh", "interface", "show", "interface"], capture_output=True, text=True)
        lines = result.stdout.splitlines()
        adapter_index = -1
        for i, line in enumerate(lines):
            if "Wi-Fi" in line or "Wireless" in line:
                adapter_index = i
                break

        if adapter_index >= 0:
            adapter_name = lines[adapter_index].split()[-1]
            logger.info(f"🔄 Resetting adapter: {adapter_name}")

            # Disable the adapter
            subprocess.run(["netsh", "interface", "set", "interface", adapter_name, "admin=disable"], check=True)
            time.sleep(2)

            # Enable the adapter
            subprocess.run(["netsh", "interface", "set", "interface", adapter_name, "admin=enable"], check=True)
            time.sleep(5)

            logger.info(f"✅ Device MAC address changed to {mac} and adapter reset")
            return mac
        else:
            logger.warning("⚠️ Could not find Wi-Fi adapter for reset")
            return mac
    except Exception as e:
        logger.error(f"⚠️ Device MAC change failed: {e}")
        return None
def get_all_mac_addresses():
    """Get MAC addresses of all network adapters"""
    try:
        mac_addresses = {}

        # Try using ipconfig first
        try:
            result = subprocess.run(
                ["ipconfig", "/all"],
                capture_output=True, text=True, check=True
            )
            lines = result.stdout.splitlines()
            current_adapter = None

            for line in lines:
                if line.strip() == "":
                    continue
                # Check if this line contains an adapter name
                if "adapter" in line.lower() and ":" in line:
                    current_adapter = line.split(":")[0].strip()
                # Check if this line contains a physical address
                elif current_adapter and "physical address" in line.lower():
                    mac = line.split(":")[-1].strip()
                    if mac != "":
                        mac_addresses[current_adapter] = mac
        except Exception as e:
            logger.warning(f"Failed to get MAC addresses using ipconfig: {e}")

        # If we didn't get any MAC addresses with ipconfig, try getmac
        if not mac_addresses:
            try:
                result = subprocess.run(
                    ["getmac", "/fo", "list", "/v", "/nh"],
                    capture_output=True, text=True, check=True
                )
                lines = result.stdout.splitlines()

                current_adapter = None
                for line in lines:
                    if line.strip() == "":
                        continue
                    if not line.startswith(" "):
                        current_adapter = line.strip()
                    elif "Physical Address" in line and current_adapter:
                        mac = line.split(":")[-1].strip()
                        if mac != "":
                            mac_addresses[current_adapter] = mac
            except Exception as e:
                logger.warning(f"Failed to get MAC addresses using getmac: {e}")

        return mac_addresses
    except Exception as e:
        logger.error(f"Error getting all MAC addresses: {e}")
        return {}
# === NEW FEATURE: Enhanced Cache Generator & Cleaner ===
def generate_realistic_cache(visited_sites):
    """Generate realistic cache for the sites that will be visited"""
    os.makedirs("fake_cache", exist_ok=True)

    for site_url in visited_sites:
        try:
            # Parse the URL to get domain
            parsed_url = urlparse(site_url)
            domain = parsed_url.netloc

            # Create a directory for this domain
            domain_dir = os.path.join("fake_cache", domain)
            os.makedirs(domain_dir, exist_ok=True)

            # Generate different types of cache files for this site
            num_files = random.randint(3, 8)

            for i in range(num_files):
                # Generate realistic cache file names
                file_types = [
                    f"cache_{random.randint(10000, 99999)}.dat",
                    f"f_{random.randint(100000, 999999)}.tmp",
                    f"{random.choice(['img', 'css', 'js'])}_{random.randint(1000, 9999)}_{random.randint(1, 9)}",
                    f"{hashlib.md5(os.urandom(16)).hexdigest()[:16]}"
                ]

                filename = random.choice(file_types)
                filepath = os.path.join(domain_dir, filename)

                # Generate realistic content based on file type
                if filename.startswith("img_"):
                    # Simulate image cache
                    content = os.urandom(random.randint(1000, 5000))
                elif filename.startswith("css_"):
                    # Simulate CSS cache
                    content = f"/* Cached CSS from {domain} */\n".encode() + os.urandom(random.randint(500, 2000))
                elif filename.startswith("js_"):
                    # Simulate JavaScript cache
                    content = f"// Cached JS from {domain}\n".encode() + os.urandom(random.randint(1000, 3000))
                else:
                    # Generic cache
                    content = os.urandom(random.randint(500, 2000))

                with open(filepath, "wb") as f:
                    f.write(content)

            logger.info(f"📂 Generated {num_files} cache files for {domain}")

        except Exception as e:
            logger.error(f"Error generating cache for {site_url}: {e}")
def delete_cache():
    """Delete the entire cache directory"""
    if os.path.exists("fake_cache"):
        for root, dirs, files in os.walk("fake_cache", topdown=False):
            for name in files:
                try:
                    os.remove(os.path.join(root, name))
                except:
                    pass
            for name in dirs:
                try:
                    os.rmdir(os.path.join(root, name))
                except:
                    pass
        try:
            os.rmdir("fake_cache")
            logger.info("🗑️ Deleted cache directory.")
        except:
            pass
# === Load proxies from file ===
def load_proxies(file):
    try:
        with open(file, "r", encoding="utf-8") as f:
            return [line.strip() for line in f if line.strip()]
    except:
        return []
# === ENHANCED: Format proxy for all types ===
def format_proxy(proxy_raw):
    """Format proxy for all types including various formats"""
    # Handle different proxy formats
    if "@" in proxy_raw:
        # Format: login:password@hostname:port or hostname:port@login:password
        if proxy_raw.count("@") == 1:
            if ":" in proxy_raw.split("@")[0]:
                # login:password@hostname:port
                auth, host_port = proxy_raw.split("@")
                username, password = auth.split(":")
                if ":" not in host_port:
                    # Add default port if missing
                    host_port += ":8080"
            else:
                # hostname:port@login:password
                host_port, auth = proxy_raw.split("@")
                username, password = auth.split(":")
                if ":" not in host_port:
                    # Add default port if missing
                    host_port += ":8080"
        else:
            # Invalid format
            return {"http": proxy_raw, "https": proxy_raw}
    elif "://" in proxy_raw:
        # Format: protocol://hostname:port or protocol://login:password@hostname:port
        protocol, rest = proxy_raw.split("://", 1)
        if "@" in rest:
            # protocol://login:password@hostname:port
            auth, host_port = rest.split("@", 1)
            username, password = auth.split(":")
            if ":" not in host_port:
                # Add default port if missing
                host_port += ":8080"
        else:
            # protocol://hostname:port
            host_port = rest
            if ":" not in host_port:
                # Add default port if missing
                host_port += ":8080"
            username, password = None, None
    else:
        # Format: hostname:port or login:password:hostname:port
        if proxy_raw.count(":") >= 3:
            # login:password:hostname:port
            parts = proxy_raw.split(":")
            username = parts[0]
            password = parts[1]
            host_port = f"{parts[2]}:{parts[3]}"
        else:
            # hostname:port
            if ":" not in proxy_raw:
                # Add default port if missing
                proxy_raw += ":8080"
            host_port = proxy_raw
            username, password = None, None

    # Determine protocol
    if "://" in proxy_raw:
        protocol = proxy_raw.split("://")[0].lower()
    else:
        # Default to http if no protocol specified
        protocol = "http"

    # Format based on protocol
    if username and password:
        if protocol == "socks5":
            return {"http": f"socks5://{username}:{password}@{host_port}", "https": f"socks5://{username}:{password}@{host_port}"}
        elif protocol == "socks4":
            return {"http": f"socks4://{username}:{password}@{host_port}", "https": f"socks4://{username}:{password}@{host_port}"}
        else:  # http or https
            return {"http": f"http://{username}:{password}@{host_port}", "https": f"http://{username}:{password}@{host_port}"}
    else:
        if protocol == "socks5":
            return {"http": f"socks5://{host_port}", "https": f"socks5://{host_port}"}
        elif protocol == "socks4":
            return {"http": f"socks4://{host_port}", "https": f"socks4://{host_port}"}
        else:  # http or https
            return {"http": f"http://{host_port}", "https": f"http://{host_port}"}
def check_proxy(proxy_raw):
    proxy = format_proxy(proxy_raw)
    start_time = time.time()
    try:
        r = requests.get("https://ipinfo.io/json", proxies=proxy, timeout=10, verify=False)
        latency = (time.time() - start_time) * 1000  # in ms
        if r.status_code == 200:
            return True, latency
    except:
        return False, 0
    return False, 0
# === FIXED: Remove only the first occurrence of a proxy from file ===
def remove_line_from_file(filename, line_to_remove):
    try:
        with open(filename, "r", encoding="utf-8") as f:
            lines = [line.strip() for line in f if line.strip()]

        removed = False
        new_lines = []
        for line in lines:
            if not removed and line == line_to_remove:
                removed = True
                continue
            new_lines.append(line)

        with open(filename, "w", encoding="utf-8") as f:
            for line in new_lines:
                f.write(line + "\n")

        if removed:
            logger.info(f"✅ Removed proxy: {line_to_remove}")
        else:
            logger.warning(f"⚠️ Proxy '{line_to_remove}' not found in {filename}")
    except Exception as e:
        logger.error(f"⚠️ Failed to update {filename}: {e}")
# === ENHANCED: Batch proxy health check with detailed output ===
def batch_proxy_health_check(proxies_list):
    logger.info("🔍 Starting batch proxy health check...")
    good_proxies = []
    bad_proxies = []
    total = len(proxies_list)
    proxy_scores = {}

    for idx, proxy in enumerate(proxies_list, 1):
        is_good, latency = check_proxy(proxy)
        if is_good:
            # Check proxy quality
            is_high_quality = check_proxy_quality(proxy)

            # Score based on latency and quality
            score = max(0, 100 - (latency / 10))
            if is_high_quality:
                score += 20  # Bonus for high quality proxies
            else:
                score -= 10  # Penalty for low quality proxies

            proxy_scores[proxy] = score
            good_proxies.append(proxy)
            logger.info(f"✅ [{idx}/{total}] Proxy OK: {proxy} | Latency: {latency:.2f}ms | Score: {score:.2f} | Quality: {'High' if is_high_quality else 'Low'}")
        else:
            bad_proxies.append(proxy)
            logger.info(f"❌ [{idx}/{total}] Proxy BAD: {proxy}")
            with open("bad_proxies.txt", "a", encoding="utf-8") as f:
                f.write(proxy + "\n")


    logger.info(f"🔍 Proxy health check complete. Good: {len(good_proxies)}, Bad: {len(bad_proxies)}\n")
    return good_proxies, bad_proxies, proxy_scores
# === Auto-update proxies ===
def update_proxies():
    if not config.auto_update_proxies:
        return

    logger.info("🔄 Auto-updating proxies...")
    # This would integrate with proxy provider APIs
    # For demonstration, we'll just check if proxies need updating
    proxy_file = "proxies.txt"
    if os.path.exists(proxy_file):
        mod_time = os.path.getmtime(proxy_file)
        if time.time() - mod_time > config.proxy_update_interval:
            # In a real implementation, this would fetch new proxies
            logger.info("Proxy list is outdated. Fetching new proxies...")
            # Simulate fetching new proxies
            new_proxies = [
                "user:pass@proxy1.example.com:8080",
                "user:pass@proxy2.example.com:8080",
                "user:pass@proxy3.example.com:8080"
            ]
            with open(proxy_file, "w") as f:
                for proxy in new_proxies:
                    f.write(proxy + "\n")
            logger.info(f"✅ Updated proxy list with {len(new_proxies)} new proxies")
    else:
        logger.warning("Proxy file not found. Creating new one...")
        with open(proxy_file, "w") as f:
            f.write("user:pass@proxy1.example.com:8080\n")
        logger.info("✅ Created new proxy file")
# === Load user agents ===
def load_user_agents(filename):
    try:
        with open(filename, "r", encoding="utf-8") as f:
            return [line.strip() for line in f if line.strip()]
    except:
        return []
# === LOAD USER AGENTS for all devices ===
iphone_agents = load_user_agents("iphoneuseragent.txt")
samsung_agents = load_user_agents("samsunguseragent.txt")
windows_agents = load_user_agents("windowsuseragent.txt")
googlepixel_agents = load_user_agents("googlepixeluseragent.txt")
motorola_agents = load_user_agents("motorolauseragent.txt")
oneplus_agents = load_user_agents("oneplususeragent.txt")
vivo_agents = load_user_agents("vivouseragent.txt")
xiaomi_agents = load_user_agents("xiaomiuseragent.txt")
zte_agents = load_user_agents("zteuseragent.txt")
# === Device percentage distribution ===
device_percentage_distribution = {
    "iPhone": 40,     # 40% of clicks will be iPhone
    "Samsung": 23,    # 23% of clicks will be Samsung
    "GooglePixel": 12, # 12% of clicks will be Google Pixel
    "Motorola": 10,    # 10% of clicks will be Motorola
    "OnePlus": 5,     # 5% of clicks will be OnePlus
    "Xiaomi": 5,      # 5% of clicks will be Xiaomi
    "ZTE": 3,         # 3% of clicks will be ZTE
    "Vivo": 2,        # 2% of clicks will be Vivo
    "Windows": 0      # 0% Windows (disabled)
}
# === Success rate tracking for dynamic weights ===
device_success_rates = {
    "iPhone": {"success": 0, "total": 0},
    "Samsung": {"success": 0, "total": 0},
    "Windows": {"success": 0, "total": 0},
    "GooglePixel": {"success": 0, "total": 0},
    "Motorola": {"success": 0, "total": 0},
    "OnePlus": {"success": 0, "total": 0},
    "Vivo": {"success": 0, "total": 0},
    "Xiaomi": {"success": 0, "total": 0},
    "ZTE": {"success": 0, "total": 0}
}
# === EXPANDED: Random sites for warm-up ===
random_sites = [
    "https://google.com", "https://youtube.com", "https://facebook.com", "https://reddit.com",
    "https://amazon.com", "https://wikipedia.org", "https://instagram.com", "https://x.com",
    "https://yahoo.com", "https://bing.com", "https://tiktok.com", "https://linkedin.com",
    "https://yelp.com", "https://nytimes.com", "https://imdb.com", "https://quora.com",
    "https://pinterest.com", "https://weather.com", "https://espn.com", "https://etsy.com",
    "https://walmart.com", "https://target.com", "https://tripadvisor.com", "https://healthline.com",
    "https://netflix.com", "https://twitch.tv", "https://bankofamerica.com", "https://costco.com",
    "https://home depot.com", "https://zillow.com", "https://usps.com", "https://indeed.com",
    "https://cnn.com", "https://usatoday.com", "https://cvs.com", "https://spotify.com",
    "https://microsoft.com", "https://adobe.com", "https://ebay.com", "https://lowes.com",
    "https://realtor.com", "https://expedia.com", "https://nbcnews.com", "https://cbssports.com",
    "https://health.com", "https://mayoClinic.org", "https://irs.gov", "https://cigna.com",
    "https://britannica.com", "https://chase.com", "https://marriott.com", "https://foxnews.com",
    "https://chicagotribune.com", "https://nyc.gov", "https://time.com", "https://forbes.com",
    "https://cnn.com", "https://usa.gov", "https://craigslist.org", "https://yahoo.net",
    "https://reddit.com", "https://msn.com", "https://fandom.com", "https://merriam-webster.com",
    "https://quora.com", "https://urbanspoon.com", "https://boxoffice.com", "https://weatherbug.com",
    "https://rotfl.com", "https://rotten tomatoes.com", "https://grubhub.com", "https://nopCommerce.com",
    "https://espn.com", "https://nba.com", "https://nfl.com", "https://amc.com",
    "https://github.com", "https://stackoverflow.com", "https://craigslist.org", "https://etsy.com",
    "https://reddit.com", "https://tripadvisor.com", "https://food.com", "https://allrecipes.com",
    "https://hulu.com", "https://spotify.com", "https://pandora.com", "https://bankofamerica.com",
    "https://chase.com", "https://wellsfargo.com", "https://citi.com", "https://boeingsite.com",
    "https://delta.com", "https://united.com", "https://ss.gov", "https://irs.gov",
    "https://healthcare.gov", "https://cdc.gov", "https://nih.gov", "https://usa.gov",
    "https://ny.gov", "https://ca.gov", "https://fl.gov", "https://tx.gov",
    "https://usa.edu", "https://harvard.edu", "https://mit.edu", "https://stanford.edu",
    "https://cornell.edu", "https://caltech.edu", "https://princeton.edu", "https://yale.edu",
    "https://usc.edu", "https://berkeley.edu", "https://utexas.edu", "https://wustl.edu",
    "https://umich.edu", "https://osu.edu", "https://psu.edu", "https://uchicago.edu",
    "https://northwestern.edu", "https://duke.edu", "https://nytimes.com", "https://washingtonpost.com",
    "https://latimes.com", "https://wsj.com", "https://bloomberg.com", "https://cnn.com",
    "https://foxnews.com", "https://nbcnews.com", "https://abcnews.go.com", "https://cbsnews.com",
    "https://usatoday.com", "https://reuters.com", "https://apnews.com", "https://npr.org",
    "https://msnbc.com", "https://bbc.com", "https://cnbc.com", "https://huffpost.com",
    "https://buzzfeed.com", "https://vice.com", "https://vox.com", "https://wired.com",
    "https://techcrunch.com", "https://engadget.com", "https://gizmodo.com", "https://lifehacker.com",
    "https://thespruce.com", "https://health.com", "https://webmd.com", "https://mayoclinic.org",
    "https://nih.gov", "https://cdc.gov", "https://irs.gov", "https://ssa.gov",
    "https://veterans.gov", "https://govtrack.us", "https://usa.gov",
    # NEW SITES ADDED
    "https://discord.com", "https://slack.com", "https://zoom.us", "https://teams.microsoft.com",
    "https://dropbox.com", "https://drive.google.com", "https://onedrive.live.com", "https://icloud.com",
    "https://github.com", "https://gitlab.com", "https://bitbucket.org", "https://codepen.io",
    "https://stackoverflow.com", "https://quora.com", "https://medium.com", "https://dev.to",
    "https://tumblr.com", "https://wordpress.com", "https://blogger.com", "https://wix.com",
    "https://squarespace.com", "https://shopify.com", "https://woocommerce.com", "https://magento.com",
    "https://salesforce.com", "https://hubspot.com", "https://mailchimp.com", "https://constantcontact.com",
    "https://adobe.com", "https://canva.com", "https://figma.com", "https://sketch.com",
    "https://dribbble.com", "https://behance.net", "https://unsplash.com", "https://pexels.com",
    "https://shutterstock.com", "https://gettyimages.com", "https://istockphoto.com", "https://adobe.com/stock",
    "https://spotify.com", "https://music.apple.com", "https://soundcloud.com", "https://pandora.com",
    "https://youtube.com", "https://vimeo.com", "https://dailymotion.com", "https://twitch.tv",
    "https://netflix.com", "https://hulu.com", "https://disneyplus.com", "https://hbo.com",
    "https://amazon.com/prime", "https://paramountplus.com", "https://peacocktv.com", "https://appletv.com",
    "https://booking.com", "https://airbnb.com", "https://expedia.com", "https://kayak.com",
    "https://tripadvisor.com", "https://hotels.com", "https://priceline.com", "https://orbitz.com",
    "https://opentable.com", "https://yelp.com", "https://foursquare.com", "https://tripadvisor.com",
    "https://uber.com", "https://lyft.com", "https://doordash.com", "https://grubhub.com",
    "https://postmates.com", "https://instacart.com", "https://shipt.com", "https://walmart.com/grocery"
]
# === Carrier list by country for geolocation matching ===
carriers_by_country = {
    "US": ["Verizon", "AT&T", "T-Mobile", "Sprint", "Metro PCS", "U.S. Cellular",
           "Boost Mobile", "Cricket Wireless", "Virgin Mobile", "Straight Talk"],
    "UK": ["Vodafone", "EE", "O2", "Three", "Giffgaff", "Tesco Mobile", "Virgin Mobile", "BT Mobile"],
    "DE": ["Deutsche Telekom", "Vodafone", "O2", "E-Plus", "Blau", "Congstar", "Lycamobile"],
    "FR": ["Orange", "SFR", "Bouygues", "Free Mobile", "Virgin Mobile", "Lycamobile"],
    "IT": ["TIM", "Vodafone", "Wind Tre", "Iliad", "PosteMobile", "CoopVoce"],
    "ES": ["Movistar", "Vodafone", "Orange", "Yoigo", "MásMóvil", "Lycamobile"],
    "BR": ["Claro", "Vivo", "TIM", "Oi", "Nextel", "Algar Telecom"],
    "IN": ["Jio", "Airtel", "Vi", "BSNL"],
    "CA": ["Rogers", "Bell", "Telus", "Freedom Mobile", "Videotron", "SaskTel"],
    "AU": ["Telstra", "Optus", "Vodafone", "TPG", "Amaysim"],
    # Add more countries as needed
}
# === NEW: Timezone mapping by country ===
def get_timezone_for_location(country, city=None):
    """Get appropriate timezone for a given country/city"""
    # Common timezone mapping by country
    country_timezones = {
        "US": "America/New_York",  # Default to EST, can be refined by region
        "DE": "Europe/Berlin",
        "UK": "Europe/London",
        "FR": "Europe/Paris",
        "IT": "Europe/Rome",
        "ES": "Europe/Madrid",
        "BR": "America/Sao_Paulo",
        "IN": "Asia/Kolkata",
        "CA": "America/Toronto",
        "AU": "Australia/Sydney",
        "JP": "Asia/Tokyo",
        "KR": "Asia/Seoul",
        "CN": "Asia/Shanghai",
        "RU": "Europe/Moscow",
    }

    # Return timezone based on country
    if country in country_timezones:
        return country_timezones[country]

    # Default to UTC if country not found
    return "UTC"
# === NEW: Verify proxy location ===
def verify_proxy_location(proxy):
    """Verify proxy location from multiple sources"""
    try:
        formatted_proxy = format_proxy(proxy)

        # Check with ipinfo.io
        r1 = requests.get("https://ipinfo.io/json", proxies=formatted_proxy, timeout=10, verify=False)
        data1 = r1.json()

        # Check with ipapi.co (backup)
        r2 = requests.get("https://ipapi.co/json/", proxies=formatted_proxy, timeout=10, verify=False)
        data2 = r2.json()

        # Compare countries
        if data1.get("country") == data2.get("country"):
            return {
                "ip": data1.get("ip"),
                "country": data1.get("country"),
                "city": data1.get("city"),
                "region": data1.get("region"),
                "timezone": get_timezone_for_location(data1.get("country")),
                "isp": data1.get("org"),
                "verified": True
            }
        else:
            logger.warning(f"⚠️ Location mismatch for proxy: {proxy}")
            return None

    except Exception as e:
        logger.error(f"Proxy verification failed: {e}")
        return None
# === Enhanced Device Fingerprint Generation ===
class DeviceFingerprint:
    def __init__(self, user_agent=None):
        self.user_agent = user_agent
        self.seed = str(random.randint(1000000000, 9999999999))
        random.seed(self.seed)
        self.fingerprint_data = {}
        self.device_info = self.parse_user_agent(user_agent) if user_agent else {}

    def parse_user_agent(self, ua):
        """Parse user agent to extract device information"""
        device_info = {
            'device_type': 'Unknown',
            'device_model': 'Unknown',
            'os_name': 'Unknown',
            'os_version': 'Unknown',
            'browser_name': 'Unknown',
            'browser_version': 'Unknown'
        }

        # Detect device type and model
        if 'iPhone' in ua:
            device_info['device_type'] = 'iPhone'
            # Extract iPhone model
            model_match = re.search(r'iPhone(?:\s+OS)?[\s;]+.*?(\d+,\d+)', ua)
            if model_match:
                model_version = model_match.group(1).replace(',', '.')
                device_info['device_model'] = f"iPhone {model_version}"
            else:
                device_info['device_model'] = 'iPhone'

            # Extract iOS version
            ios_match = re.search(r'iPhone OS (\d+_\d+(_\d+)?)', ua)
            if ios_match:
                ios_version = ios_match.group(1).replace('_', '.')
                device_info['os_version'] = f"iOS {ios_version}"
            else:
                device_info['os_version'] = 'iOS'

        elif 'iPad' in ua:
            device_info['device_type'] = 'iPad'
            device_info['device_model'] = 'iPad'

            # Extract iOS version
            ios_match = re.search(r'CPU OS (\d+_\d+(_\d+)?)', ua)
            if ios_match:
                ios_version = ios_match.group(1).replace('_', '.')
                device_info['os_version'] = f"iOS {ios_version}"
            else:
                device_info['os_version'] = 'iOS'

        elif 'Android' in ua:
            device_info['os_name'] = 'Android'

            # Extract Android version
            android_match = re.search(r'Android (\d+\.\d+(\.\d+)?)', ua)
            if android_match:
                device_info['os_version'] = f"Android {android_match.group(1)}"
            else:
                device_info['os_version'] = 'Android'

            # Detect device manufacturer and model
            if 'SM-' in ua:
                device_info['device_type'] = 'Samsung'
                model_match = re.search(r'SM-([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"Samsung SM-{model_match.group(1)}"
                else:
                    device_info['device_model'] = 'Samsung'
            elif 'Pixel' in ua:
                device_info['device_type'] = 'GooglePixel'
                model_match = re.search(r'Pixel (\d+)', ua)
                if model_match:
                    device_info['device_model'] = f"Google Pixel {model_match.group(1)}"
                else:
                    device_info['device_model'] = 'Google Pixel'
            elif 'Motorola' in ua:
                device_info['device_type'] = 'Motorola'
                model_match = re.search(r'Motorola ([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"Motorola {model_match.group(1)}"
                else:
                    device_info['device_model'] = 'Motorola'
            elif 'OnePlus' in ua:
                device_info['device_type'] = 'OnePlus'
                model_match = re.search(r'OnePlus ([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"OnePlus {model_match.group(1)}"
                else:
                    device_info['device_model'] = 'OnePlus'
            elif 'vivo' in ua.lower():
                device_info['device_type'] = 'Vivo'
                model_match = re.search(r'vivo ([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"Vivo {model_match.group(1)}"
                else:
                    device_info['device_model'] = 'Vivo'
            elif 'Mi' in ua or 'Xiaomi' in ua:
                device_info['device_type'] = 'Xiaomi'
                model_match = re.search(r'(Mi|Xiaomi) ([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"Xiaomi {model_match.group(2)}"
                else:
                    device_info['device_model'] = 'Xiaomi'
            elif 'ZTE' in ua:
                device_info['device_type'] = 'ZTE'
                model_match = re.search(r'ZTE ([A-Za-z0-9]+)', ua)
                if model_match:
                    device_info['device_model'] = f"ZTE {model_match.group(1)}"
                else:
                    device_info['device_model'] = 'ZTE'
            else:
                device_info['device_type'] = 'Android'
                device_info['device_model'] = 'Android Device'

        elif 'Windows' in ua:
            device_info['device_type'] = 'Windows'
            device_info['device_model'] = 'Windows PC'

            # Extract Windows version
            win_match = re.search(r'Windows NT (\d+\.\d+)', ua)
            if win_match:
                win_version = win_match.group(1)
                if win_version == '10.0':
                    device_info['os_version'] = 'Windows 10'
                elif win_version == '6.3':
                    device_info['os_version'] = 'Windows 8.1'
                elif win_version == '6.1':
                    device_info['os_version'] = 'Windows 7'
                else:
                    device_info['os_version'] = f'Windows {win_version}'
            else:
                device_info['os_version'] = 'Windows'

        # Detect browser
        if 'Chrome' in ua:
            device_info['browser_name'] = 'Chrome'
            chrome_match = re.search(r'Chrome/(\d+\.\d+\.\d+\.\d+)', ua)
            if chrome_match:
                device_info['browser_version'] = chrome_match.group(1)
            else:
                device_info['browser_version'] = 'Latest'
        elif 'Firefox' in ua:
            device_info['browser_name'] = 'Firefox'
            firefox_match = re.search(r'Firefox/(\d+\.\d+)', ua)
            if firefox_match:
                device_info['browser_version'] = firefox_match.group(1)
            else:
                device_info['browser_version'] = 'Latest'
        elif 'Safari' in ua and 'Chrome' not in ua:
            device_info['browser_name'] = 'Safari'
            safari_match = re.search(r'Version/(\d+\.\d+)', ua)
            if safari_match:
                device_info['browser_version'] = safari_match.group(1)
            else:
                device_info['browser_version'] = 'Latest'
        elif 'Edge' in ua:
            device_info['browser_name'] = 'Edge'
            edge_match = re.search(r'Edge/(\d+\.\d+\.\d+\.\d+)', ua)
            if edge_match:
                device_info['browser_version'] = edge_match.group(1)
            else:
                device_info['browser_version'] = 'Latest'

        return device_info

    def generate_imei(self):
        # Generate a realistic IMEI based on device type
        if self.device_info.get('device_type') == 'iPhone':
            # iPhone IMEIs start with 35
            imei = "35" + "".join([str(random.randint(0, 9)) for _ in range(13)])
        elif self.device_info.get('device_type') == 'Samsung':
            # Samsung IMEIs vary, but many start with 35
            imei = "35" + "".join([str(random.randint(0, 9)) for _ in range(13)])
        else:
            # Generic IMEI
            imei = "35" + "".join([str(random.randint(0, 9)) for _ in range(13)])
        return imei

    def generate_device_id(self):
        # Generate a unique device ID based on user agent
        ua_hash = hashlib.md5(self.user_agent.encode()).hexdigest()[:8]
        random_part = str(random.randint(1000, 9999))
        return f"{ua_hash}{random_part}"

    def get_screen_resolution(self):
        # Return realistic screen resolutions based on parsed device info
        device_type = self.device_info.get('device_type', 'Unknown')
        device_model = self.device_info.get('device_model', '')

        if 'iPhone' in device_model:
            if 'iPhone 14' in device_model or 'iPhone 13' in device_model:
                return f"{random.choice([390, 428])}x{random.choice([844, 926])}"
            elif 'iPhone 12' in device_model or 'iPhone 11' in device_model:
                return f"{random.choice([375, 414])}x{random.choice([812, 896])}"
            else:
                return f"{random.choice([320, 375, 414])}x{random.choice([568, 667, 736, 812])}"
        elif 'iPad' in device_model:
            return f"{random.choice([768, 810, 820, 834])}x{random.choice([1024, 1080, 1180, 1194])}"
        elif 'Samsung' in device_model:
            if 'Galaxy S' in device_model:
                return f"{random.choice([360, 384, 412, 420])}x{random.choice([640, 732, 846, 915])}"
            elif 'Galaxy Note' in device_model:
                return f"{random.choice([384, 412, 420])}x{random.choice([732, 846, 915])}"
            else:
                return f"{random.choice([360, 384, 412])}x{random.choice([640, 732, 846])}"
        elif 'Pixel' in device_model:
            if 'Pixel 6' in device_model or 'Pixel 7' in device_model:
                return f"{random.choice([393, 412])}x{random.choice([846, 915])}"
            else:
                return f"{random.choice([393, 411])}x{random.choice([786, 823])}"
        elif 'Windows' in device_type:
            return f"{random.choice([1366, 1536, 1920])}x{random.choice([768, 864, 1080])}"
        else:
            # Generic Android resolution
            return f"{random.randint(300, 450)}x{random.randint(500, 900)}"

    def get_viewport_size(self, resolution):
        # Calculate viewport from resolution
        width, height = map(int, resolution.split('x'))
        # Account for browser chrome, status bar, etc.
        viewport_height = height - random.randint(50, 150)
        return f"{width}x{viewport_height}"

    def get_hardware_info(self):
        # Return realistic hardware specs based on device info
        device_type = self.device_info.get('device_type', 'Unknown')
        device_model = self.device_info.get('device_model', '')

        if 'iPhone' in device_model:
            if 'iPhone 14' in device_model or 'iPhone 13' in device_model:
                cpu = "Apple A15 Bionic"
                cores = 6
                memory = random.choice([4, 6])
            elif 'iPhone 12' in device_model:
                cpu = "Apple A14 Bionic"
                cores = 6
                memory = random.choice([4, 6])
            elif 'iPhone 11' in device_model:
                cpu = "Apple A13 Bionic"
                cores = 6
                memory = random.choice([4, 6])
            else:
                cpu = "Apple A12 Bionic"
                cores = 6
                memory = random.choice([3, 4])
        elif 'iPad' in device_model:
            cpu = "Apple"
            if 'M1' in device_model or 'M2' in device_model:
                cpu += f" {random.choice(['M1', 'M2'])}"
                cores = 8
                memory = random.choice([8, 16])
            else:
                cpu += " A"
                cpu += str(random.choice([12, 13, 14]))
                cores = random.choice([6, 8])
                memory = random.choice([4, 6, 8])
        elif 'Samsung' in device_model:
            if 'Galaxy S22' in device_model or 'Galaxy S21' in device_model:
                cpu = "Qualcomm Snapdragon 8 Gen 1"
                cores = 8
                memory = random.choice([8, 12])
            elif 'Galaxy S20' in device_model:
                cpu = "Qualcomm Snapdragon 865"
                cores = 8
                memory = random.choice([8, 12])
            else:
                cpu = "Qualcomm Snapdragon"
                cpu += str(random.choice([855, 865, 888]))
                cores = 8
                memory = random.choice([6, 8])
        elif 'Pixel' in device_model:
            if 'Pixel 6' in device_model or 'Pixel 7' in device_model:
                cpu = "Google Tensor"
                cores = 8
                memory = random.choice([8, 12])
            else:
                cpu = "Qualcomm Snapdragon"
                cpu += str(random.choice([765, 768, 780]))
                cores = 8
                memory = random.choice([6, 8])
        elif 'Windows' in device_type:
            cpu = "Intel Core i"
            cpu += str(random.choice([5, 7, 9])) + "-" + str(random.choice([3000, 7000, 9000]))
            cores = random.choice([4, 6, 8])
            memory = random.choice([8, 16, 32])
        else:
            # Generic Android hardware
            cpu_brands = ["Qualcomm Snapdragon", "MediaTek Helio", "Samsung Exynos"]
            cpu = random.choice(cpu_brands)
            if "Qualcomm" in cpu:
                cpu += " " + str(random.choice([680, 720, 750, 765, 778, 888]))
            elif "MediaTek" in cpu:
                cpu += " " + str(random.choice(["G80", "G85", "G90", "G95", "G96", "G99"]))
            else:
                cpu += " " + str(random.choice([980, 981, 982, 983, 1080, 2100, 2200]))
            cores = random.choice([6, 8])
            memory = random.choice([4, 6, 8])

        return {
            "cpu": cpu,
            "cores": cores,
            "memory": memory
        }

    def get_os_build(self):
        # Use the OS version extracted from user agent
        os_version = self.device_info.get('os_version', 'Unknown')
        if os_version != 'Unknown':
            return os_version

        # Fallback to generating a realistic OS build number
        device_type = self.device_info.get('device_type', 'Unknown')
        if "iPhone" in device_type or "iPad" in device_type:
            return f"iOS {random.choice([14, 15, 16])}.{random.randint(0, 7)}.{random.randint(0, 9)}"
        elif "Android" in device_type:
            return f"Android {random.choice([10, 11, 12, 13])}.{random.randint(0, 9)}"
        elif "Windows" in device_type:
            return f"Windows 10 Build {random.randint(19000, 22600)}"
        else:
            return f"Android {random.choice([10, 11, 12, 13])}.{random.randint(0, 9)}"

    def get_battery_info(self):
        # Generate realistic battery info
        return {
            "level": random.randint(20, 95),
            "charging": random.choice([True, False]),
            "chargingTime": random.randint(0, 3600),
            "dischargingTime": random.randint(1800, 7200)
        }

    def get_memory_info(self):
        # Generate realistic memory info
        hardware = self.get_hardware_info()
        device_memory = hardware.get("memory", 4)

        return {
            "deviceMemory": device_memory,
            "totalJSHeapSize": random.randint(10000000, 50000000),
            "usedJSHeapSize": random.randint(5000000, 30000000),
            "jsHeapSizeLimit": random.randint(100000000, 500000000)
        }

    def get_connection_info(self):
        # Generate realistic connection info
        return {
            "effectiveType": random.choice(["4g", "3g", "2g"]),
            "rtt": random.randint(50, 500),
            "downlink": random.randint(1, 10),
            "saveData": random.choice([True, False])
        }

    def get_sensor_info(self):
        # Generate realistic sensor info
        return {
            "accelerometer": {
                "x": random.uniform(-10, 10),
                "y": random.uniform(-10, 10),
                "z": random.uniform(-10, 10)
            },
            "gyroscope": {
                "alpha": random.uniform(0, 360),
                "beta": random.uniform(-180, 180),
                "gamma": random.uniform(-90, 90)
            },
            "magnetometer": {
                "x": random.uniform(-100, 100),
                "y": random.uniform(-100, 100),
                "z": random.uniform(-100, 100)
            }
        }

    def get_webgl_info(self):
        # Generate realistic WebGL info based on device
        device_type = self.device_info.get('device_type', 'Unknown')

        if 'iPhone' in device_type or 'iPad' in device_type:
            # Apple devices use their own GPUs
            vendor = "Apple Inc."
            if 'M1' in self.device_info.get('device_model', '') or 'M2' in self.device_info.get('device_model', ''):
                renderer = "Apple M1"
            else:
                renderer = "Apple A"
                renderer += str(random.choice([12, 13, 14, 15]))
                renderer += " GPU"
        elif 'Windows' in device_type:
            # Windows devices can have various GPUs
            gpu_vendors = ["NVIDIA Corporation", "Intel Inc.", "AMD"]
            vendor = random.choice(gpu_vendors)

            if vendor == "NVIDIA Corporation":
                gpu_models = [
                    "NVIDIA GeForce RTX 3080/PCIe/SSE2",
                    "NVIDIA GeForce RTX 3070/PCIe/SSE2",
                    "NVIDIA GeForce RTX 3060/PCIe/SSE2",
                    "NVIDIA GeForce GTX 1660/PCIe/SSE2"
                ]
                renderer = random.choice(gpu_models)
            elif vendor == "Intel Inc.":
                gpu_models = [
                    "Intel(R) Iris(R) Xe Graphics",
                    "Intel(R) UHD Graphics 630",
                    "Intel(R) HD Graphics 530"
                ]
                renderer = random.choice(gpu_models)
            else:  # AMD
                gpu_models = [
                    "AMD Radeon RX 6800 XT",
                    "AMD Radeon RX 6700 XT",
                    "AMD Radeon RX 5600 XT"
                ]
                renderer = random.choice(gpu_models)
        else:
            # Android devices use various mobile GPUs
            gpu_vendors = ["Qualcomm", "ARM", "Imagination Technologies"]
            vendor = random.choice(gpu_vendors)

            if vendor == "Qualcomm":
                renderer = "Qualcomm Adreno"
                renderer += str(random.choice([530, 610, 615, 618, 619, 630, 640, 650, 660]))
            elif vendor == "ARM":
                renderer = "ARM Mali"
                renderer += "-G"
                renderer += str(random.choice([31, 52, 71, 72, 76, 77, 78]))
            else:  # Imagination Technologies
                renderer = "Imagination Technologies PowerVR GE"
                renderer += str(random.choice([8320, 8440, 8600]))

        return {
            "vendor": vendor,
            "renderer": renderer,
            "version": "WebGL 2.0"
        }

    def get_canvas_fingerprint_with_noise(self):
        """Generate a canvas fingerprint with realistic noise"""
        # Create a canvas
        img = Image.new('RGB', (200, 50), color=(255, 255, 255))
        draw = ImageDraw.Draw(img)

        # Try to use a common font
        try:
            font = ImageFont.truetype("arial.ttf", 14)
        except:
            font = ImageFont.load_default()

        # Draw text
        text = "Canvas fingerprinting test"
        draw.text((10, 10), text, fill=(0, 0, 0), font=font)

        # Add some random noise (slight variations that occur in real browsers)
        pixels = img.load()
        for i in range(img.size[0]):
            for j in range(img.size[1]):
                # Only modify some pixels
                if random.random() < 0.01:  # 1% of pixels
                    r, g, b = pixels[i, j]
                    # Small random variations
                    pixels[i, j] = (
                        max(0, min(255, r + random.randint(-2, 2))),
                        max(0, min(255, g + random.randint(-2, 2))),
                        max(0, min(255, b + random.randint(-2, 2)))
                    )

        # Get the image data
        buffer = BytesIO()
        img.save(buffer, format="PNG")
        image_data = buffer.getvalue()

        # Generate hash
        return hashlib.md5(image_data).hexdigest()

    def get_webgl_fingerprint(self):
        """Generate a realistic WebGL fingerprint"""
        import json

        # Get the device info to determine appropriate GPU
        device_type = self.device_info.get('device_type', 'Unknown')

        # Generate realistic WebGL parameters based on device
        if 'iPhone' in device_type or 'iPad' in device_type:
            vendor = "Apple Inc."
            renderer = "Apple GPU"
            max_texture_size = 4096
            max_viewport_dims = [4096, 4096]
        elif 'Windows' in device_type:
            gpu_vendors = ["NVIDIA Corporation", "Intel Inc.", "AMD"]
            vendor = random.choice(gpu_vendors)

            if vendor == "NVIDIA Corporation":
                renderer = random.choice([
                    "NVIDIA GeForce RTX 3080/PCIe/SSE2",
                    "NVIDIA GeForce RTX 3070/PCIe/SSE2",
                    "NVIDIA GeForce RTX 3060/PCIe/SSE2"
                ])
                max_texture_size = 16384
                max_viewport_dims = [16384, 16384]
            elif vendor == "Intel Inc.":
                renderer = random.choice([
                    "Intel(R) Iris(R) Xe Graphics",
                    "Intel(R) UHD Graphics 630"
                ])
                max_texture_size = 16384
                max_viewport_dims = [16384, 16384]
            else:  # AMD
                renderer = random.choice([
                    "AMD Radeon RX 6800 XT",
                    "AMD Radeon RX 6700 XT"
                ])
                max_texture_size = 16384
                max_viewport_dims = [16384, 16384]
        else:  # Android
            gpu_vendors = ["Qualcomm", "ARM", "Imagination Technologies"]
            vendor = random.choice(gpu_vendors)

            if vendor == "Qualcomm":
                renderer = f"Qualcomm Adreno {random.choice([530, 610, 615, 618, 619, 630, 640, 650, 660])}"
            elif vendor == "ARM":
                renderer = f"ARM Mali-G{random.choice([31, 52, 71, 72, 76, 77, 78])}"
            else:  # Imagination Technologies
                renderer = f"Imagination Technologies PowerVR GE{random.choice([8320, 8440, 8600])}"

            max_texture_size = 4096
            max_viewport_dims = [4096, 4096]

        # Generate WebGL parameters
        webgl_params = {
            "vendor": vendor,
            "renderer": renderer,
            "version": "WebGL 2.0",
            "shading_language_version": "WebGL GLSL ES 3.00",
            "max_texture_size": max_texture_size,
            "max_viewport_dims": max_viewport_dims,
            "max_cube_map_texture_size": max_texture_size,
            "max_renderbuffer_size": max_texture_size,
            "max_texture_image_units": random.choice([16, 32]),
            "max_vertex_texture_image_units": random.choice([16, 32]),
            "max_fragment_uniform_vectors": random.choice([1024, 2048]),
            "max_vertex_uniform_vectors": random.choice([1024, 2048]),
            "max_varying_vectors": random.choice([32, 64]),
            "aliased_line_width_range": [1, 1],
            "aliased_point_size_range": [1, 255],
            "max_texture_max_anisotropy_ext": random.choice([16, 32]),
            "max_anisotropy": random.choice([16, 32])
        }

        # Generate hash from the parameters
        params_str = json.dumps(webgl_params, sort_keys=True)
        return hashlib.md5(params_str.encode()).hexdigest(), webgl_params

    def get_audio_fingerprint(self):
        # Generate a unique audio fingerprint
        audio_data = str(random.randint(100000, 999999))
        return hashlib.md5(audio_data.encode()).hexdigest()

    def get_font_list(self):
        # Generate a realistic font list based on device
        device_type = self.device_info.get('device_type', 'Unknown')

        common_fonts = [
            "Arial", "Times New Roman", "Courier New", "Verdana", "Georgia",
            "Palatino", "Garamond", "Bookman", "Comic Sans MS", "Trebuchet MS",
            "Arial Black", "Impact"
        ]

        if 'iPhone' in device_type or 'iPad' in device_type:
            # Apple devices have San Francisco font
            apple_fonts = ["San Francisco", "Apple SD Gothic Neo", "Apple Myungjo"]
            common_fonts.extend(apple_fonts)
        elif 'Windows' in device_type:
            # Windows devices have Segoe UI and other Microsoft fonts
            windows_fonts = ["Segoe UI", "Calibri", "Cambria", "Consolas", "Tahoma"]
            common_fonts.extend(windows_fonts)
        elif 'Android' in device_type:
            # Android devices have Roboto and other Google fonts
            android_fonts = ["Roboto", "Noto Sans", "Droid Sans", "Google Sans"]
            common_fonts.extend(android_fonts)

        return random.sample(common_fonts, random.randint(5, 10))

    def get_timezone_offset(self, timezone):
        # Get timezone offset in minutes
        try:
            tz = pytz.timezone(timezone)
            now = datetime.datetime.now(tz)
            return int(now.utcoffset().total_seconds() / 60)
        except:
            return 0

    def get_all_fingerprints(self, ip_info):
        # Get all fingerprint data based on user agent and IP info
        _, city, country, region, tz, isp, *rest =  ip_info

        # Get country code for carrier matching
        country_code = country.upper() if country else "US"

        # Select carrier based on IP geolocation
        if country_code in carriers_by_country:
            carrier = random.choice(carriers_by_country[country_code])
        else:
            # Fallback to US carriers if country not in list
            carrier = random.choice(carriers_by_country.get("US", ["Verizon", "AT&T", "T-Mobile"]))

        # Generate all fingerprint data
        webgl_hash, webgl_params = self.get_webgl_fingerprint()

        self.fingerprint_data = {
            "imei": self.generate_imei(),
            "device_id": self.generate_device_id(),
            "screen_resolution": self.get_screen_resolution(),
            "viewport_size": self.get_viewport_size(self.get_screen_resolution()),
            "hardware": self.get_hardware_info(),
            "os_build": self.get_os_build(),
            "battery": self.get_battery_info(),
            "memory": self.get_memory_info(),
            "connection": self.get_connection_info(),
            "sensors": self.get_sensor_info(),
            "webgl": webgl_params,
            "webgl_hash": webgl_hash,
            "canvas": self.get_canvas_fingerprint_with_noise(),
            "audio": self.get_audio_fingerprint(),
            "fonts": self.get_font_list(),
            "timezone_offset": self.get_timezone_offset(tz),
            "carrier": carrier,
            "location": {
                "city": city,
                "region": region,
                "country": country,
                "timezone": tz
            },
            "isp": isp,
            "device_info": self.device_info  # Include parsed device info
        }

        return self.fingerprint_data

    def encrypt_fingerprint(self):
        # Encrypt the fingerprint data if encryption is enabled
        if config.enable_encryption and cipher_suite:
            fingerprint_str = json.dumps(self.fingerprint_data)
            return cipher_suite.encrypt(fingerprint_str.encode()).decode()
        return json.dumps(self.fingerprint_data)

    # NEW: WebGL/Canvas spoofing injection
    def inject_webgl_canvas_spoofing(self, driver):
        """Inject JavaScript to spoof WebGL and Canvas fingerprints"""
        webgl_info = self.get_webgl_info()
        canvas_hash = self.get_canvas_fingerprint_with_noise()

        script = f"""
        (function() {{
            // Spoof WebGL
            const originalGetParameter = WebGLRenderingContext.prototype.getParameter;
            WebGLRenderingContext.prototype.getParameter = function(parameter) {{
                const params = {json.dumps(webgl_info)};
                if (params.hasOwnProperty(parameter)) {{
                    return params[parameter];
                }}
                return originalGetParameter.call(this, parameter);
            }};

            // Spoof Canvas
            const originalGetImageData = CanvasRenderingContext2D.prototype.getImageData;
            CanvasRenderingContext2D.prototype.getImageData = function() {{
                const imageData = originalGetImageData.apply(this, arguments);
                // Add noise to canvas fingerprint
                for (let i = 0; i < imageData.data.length; i += 4) {{
                    if (Math.random() > 0.9) {{
                        imageData.data[i] = Math.min(255, imageData.data[i] + Math.floor(Math.random() * 5 - 2.5));
                        imageData.data[i+1] = Math.min(255, imageData.data[i+1] + Math.floor(Math.random() * 5 - 2.5));
                        imageData.data[i+2] = Math.min(255, imageData.data[i+2] + Math.floor(Math.random() * 5 - 2.5));
                    }}
                }}
                return imageData;
            }};

            // Spoof Canvas fingerprint hash
            const originalToDataURL = HTMLCanvasElement.prototype.toDataURL;
            HTMLCanvasElement.prototype.toDataURL = function() {{
                const dataURL = originalToDataURL.apply(this, arguments);
                return dataURL.replace(/[a-f0-9]{{8}}-[a-f0-9]{{4}}-[a-f0-9]{{4}}-[a-f0-9]{{4}}-[a-f0-9]{{12}}/, '{canvas_hash}');
            }};
        }})();
        """

        try:
            driver.execute_script(script)
        except Exception as e:
            logger.warning(f"Error injecting WebGL/Canvas spoofing: {e}")

    # FIXED: Simulate realistic mouse movement with bounds checking
    def simulate_human_mouse_movement(self, driver, target_element=None):
        """Simulate realistic mouse movements with random paths"""
        try:
            # Get viewport dimensions first
            viewport_width = driver.execute_script("return window.innerWidth")
            viewport_height = driver.execute_script("return window.innerHeight")

            # Create a new action chain
            action = ActionChains(driver)

            # Start from a safe position within the viewport
            current_x = viewport_width // 2
            current_y = viewport_height // 2

            # Move to the starting position first
            action.move_by_offset(current_x, current_y)
            action.perform()

            # Create a new action chain for the actual movement
            action = ActionChains(driver)

            if target_element:
                # Get target position and ensure it's within bounds
                try:
                    target_x = target_element.location['x'] + target_element.size['width'] // 2
                    target_y = target_element.location['y'] + target_element.size['height'] // 2

                    # Ensure target is within viewport bounds
                    target_x = max(10, min(viewport_width - 10, target_x))
                    target_y = max(10, min(viewport_height - 10, target_y))

                    # Move directly to the element
                    try:
                        action.move_to_element(target_element)
                        action.pause(random.uniform(0.2, 0.8))

                        # Randomly decide whether to click
                        if random.random() > 0.3:  # 70% chance to click
                            action.click()

                        action.perform()
                        return
                    except Exception as e:
                        logger.warning(f"Error moving to element: {e}")
                        # Fall back to manual movement
                        pass
                except Exception as e:
                    logger.warning(f"Error getting element position: {e}")

            # If no target element or element movement failed, use manual movement
            # Generate a random target position within safe bounds
            target_x = random.randint(10, max(11, viewport_width - 10))
            target_y = random.randint(10, max(11, viewport_height - 10))

            # Create a simple path with fewer waypoints to reduce errors
            num_waypoints = min(3, random.randint(2, 5))

            for i in range(num_waypoints):
                # Calculate intermediate position
                progress = (i + 1) / (num_waypoints + 1)
                next_x = current_x + (target_x - current_x) * progress
                next_y = current_y + (target_y - current_y) * progress

                # Ensure the position is within bounds
                next_x = max(5, min(viewport_width - 5, next_x))
                next_y = max(5, min(viewport_height - 5, next_y))

                # Calculate the offset from current position
                offset_x = next_x - current_x
                offset_y = next_y - current_y

                try:
                    # Move to the intermediate position
                    action.move_by_offset(offset_x, offset_y)
                    action.pause(random.uniform(0.1, 0.3))

                    # Update current position
                    current_x = next_x
                    current_y = next_y
                except Exception as e:
                    logger.warning(f"Error moving to intermediate position: {e}")
                    # Reset to center and create a new action chain
                    current_x = viewport_width // 2
                    current_y = viewport_height // 2
                    action = ActionChains(driver)

            # Final move to target
            try:
                # Calculate final offset
                final_offset_x = target_x - current_x
                final_offset_y = target_y - current_y

                # Move to the final position
                action.move_by_offset(final_offset_x, final_offset_y)
                action.pause(random.uniform(0.2, 0.5))

                # Perform the action
                action.perform()
            except Exception as e:
                logger.warning(f"Error in final mouse movement: {e}")

        except Exception as e:
            logger.error(f"Error in mouse movement simulation: {e}")

    # NEW: Simulate realistic scrolling
    def simulate_realistic_scroll(self, driver):
        """Simulate realistic scrolling behavior"""
        try:
            # Get the total height of the page
            total_height = driver.execute_script("return document.body.scrollHeight")

            # Scroll in multiple steps
            viewport_height = driver.execute_script("return window.innerHeight")
            scroll_steps = random.randint(2, 5)

            for i in range(scroll_steps):
                # Calculate scroll position
                scroll_position = (i + 1) * (total_height / scroll_steps)
                # Add some randomness
                scroll_position += random.randint(-50, 50)
                scroll_position = max(0, min(scroll_position, total_height - viewport_height))

                # Scroll to position
                driver.execute_script(f"window.scrollTo(0, {scroll_position});")

                # Random pause
                time.sleep(random.uniform(0.5, 2))

            # Sometimes scroll back up
            if random.random() > 0.7:
                driver.execute_script("window.scrollTo(0, 0);")
                time.sleep(random.uniform(0.5, 1))
        except Exception as e:
            logger.warning(f"Error during scroll simulation: {e}")

    # NEW: Simulate form interaction
    def simulate_form_interaction(self, driver):
        """Simulate interacting with form elements if present"""
        try:
            # Find input fields
            inputs = driver.find_elements(By.TAG_NAME, "input")
            if not inputs:
                return

            # Select a random input
            input_field = random.choice(inputs)

            # Check if it's visible and interactable
            if input_field.is_displayed() and input_field.is_enabled():
                # Scroll to the element
                driver.execute_script("arguments[0].scrollIntoView(true);", input_field)
                time.sleep(random.uniform(0.5, 1))

                # Click on the input field
                input_field.click()
                time.sleep(random.uniform(0.2, 0.5))

                # Type some random text if it's a text input
                input_type = input_field.get_attribute("type")
                if input_type in ["text", "search", "email", "password"]:
                    # Generate random text
                    text = ''.join(random.choices(string.ascii_lowercase + string.digits, k=random.randint(3, 10)))
                    input_field.send_keys(text)
                    time.sleep(random.uniform(0.5, 1))

                    # Clear the text (simulate typing and then deleting)
                    for _ in range(len(text)):
                        input_field.send_keys(Keys.BACKSPACE)
                        time.sleep(random.uniform(0.05, 0.2))
        except Exception as e:
            logger.error(f"Error during form interaction: {e}")

    # NEW: Simulate human errors
    def simulate_human_errors(self, driver):
        """Simulate human-like errors during browsing"""
        try:
            error_type = random.choice(["wrong_url", "form_error", "refresh", "click_wrong"])

            if error_type == "wrong_url":
                # Navigate to a random wrong URL
                wrong_url = f"https://{random.choice(['example', 'test', 'demo'])}{random.choice(['.com', '.org', '.net'])}"
                logger.info(f"🚫 Simulating wrong navigation to: {wrong_url}")
                driver.get(wrong_url)
                time.sleep(random.uniform(1, 3))
                # Go back
                driver.back()
                time.sleep(random.uniform(1, 2))

            elif error_type == "form_error":
                # Try to find a form and submit invalid data
                forms = driver.find_elements(By.TAG_NAME, "form")
                if forms:
                    form = random.choice(forms)
                    inputs = form.find_elements(By.TAG_NAME, "input")
                    if inputs:
                        # Fill with invalid data (e.g., invalid email)
                        for input_field in inputs:
                            input_type = input_field.get_attribute("type")
                            if input_type == "email":
                                input_field.click()
                                invalid_email = "invalid-email"
                                input_field.send_keys(invalid_email)
                                time.sleep(random.uniform(0.5, 1))
                                # Try to submit
                                submit_buttons = form.find_elements(By.XPATH, ".//input[@type='submit']") + \
                                               form.find_elements(By.XPATH, ".//button[@type='submit']")
                                if submit_buttons:
                                    submit_button = random.choice(submit_buttons)
                                    submit_button.click()
                                    time.sleep(random.uniform(1, 3))
                                    # Go back (assuming form submission leads to a new page)
                                    driver.back()
                                    time.sleep(random.uniform(1, 2))
                                break

            elif error_type == "refresh":
                # Refresh the page
                logger.info("🔄 Simulating page refresh")
                driver.refresh()
                time.sleep(random.uniform(2, 4))

            elif error_type == "click_wrong":
                # Click on a random non-important element
                elements = driver.find_elements(By.TAG_NAME, "img") + \
                          driver.find_elements(By.XPATH, "//a[contains(@href, 'image') or contains(@href, 'photo')]")

                if elements:
                    element = random.choice(elements)
                    if element.is_displayed():
                        # Scroll to element
                        driver.execute_script("arguments[0].scrollIntoView({block: 'center'});", element)
                        time.sleep(random.uniform(0.5, 1))

                        # Click
                        element.click()
                        time.sleep(random.uniform(1, 3))

                        # 70% chance to go back
                        if random.random() < 0.7:
                            driver.back()
                            time.sleep(random.uniform(1, 2))

        except Exception as e:
            logger.warning(f"Error during human error simulation: {e}")

    # NEW: Random element interaction
    def random_element_interaction(self, driver):
        """Randomly click on non-important elements"""
        try:
            # Find clickable elements that are not important (like images, non-critical links)
            elements = driver.find_elements(By.TAG_NAME, "img") + \
                      driver.find_elements(By.XPATH, "//a[contains(@href, 'image') or contains(@href, 'photo')]")

            if elements:
                element = random.choice(elements)
                if element.is_displayed():
                    # Scroll to element
                    driver.execute_script("arguments[0].scrollIntoView({block: 'center'});", element)
                    time.sleep(random.uniform(0.5, 1))

                    # Click
                    element.click()
                    time.sleep(random.uniform(1, 3))

                    # 50% chance to go back
                    if random.random() < 0.5:
                        driver.back()
                        time.sleep(random.uniform(1, 2))
        except Exception as e:
            logger.warning(f"Error during random element interaction: {e}")

    # FIXED: Random pre-click site interaction with reduced mouse movement
    def random_pre_click_interaction(self, driver, min_sites=None, max_sites=None, max_time=None):
        """Visit random sites and interact with them before clicking the target"""
        if min_sites is None:
            min_sites = config.min_pre_click_sites
        if max_sites is None:
            max_sites = config.max_pre_click_sites

        start_time = time.time()
        num_sites = random.randint(min_sites, max_sites)
        selected_sites = random.sample(random_sites, num_sites)

        for site in selected_sites:
            # Check if we have time left
            if max_time and (time.time() - start_time) >= max_time:
                logger.info(f"⏱️ Pre-click interaction stopped due to time limit")
                break

            try:
                logger.info(f"🌐 Visiting pre-click site: {site}")
                driver.get(site)

                # Wait for page to load (reduced time)
                time.sleep(random.uniform(1, 3))

                # Decide whether to interact and how many interactions
                if random.random() < config.random_interaction_probability:  # 70% chance to interact
                    # List of possible interactions
                    interactions = [
                        self.simulate_realistic_scroll,
                        self.simulate_human_mouse_movement,
                        self.simulate_form_interaction,
                        self.random_element_interaction,
                        self.simulate_human_errors
                    ]

                    # Number of interactions (0 to 3)
                    num_interactions = random.randint(0, 3)
                    selected_interactions = random.sample(interactions, min(num_interactions, len(interactions)))

                    for interaction in selected_interactions:
                        # Check time before each interaction
                        if max_time and (time.time() - start_time) >= max_time:
                            break
                        try:
                            interaction(driver)
                            time.sleep(random.uniform(1, 3))
                        except Exception as e:
                            logger.warning(f"Error during interaction on {site}: {e}")
                            continue

                # Random time on site (with reduced range to save time)
                time_on_site = random.uniform(5, 15)  # Reduced from 5-30 to 5-15
                time.sleep(time_on_site)

            except Exception as e:
                logger.error(f"Error during pre-click interaction on {site}: {e}")
                continue

    # NEW: Set timezone and locale based on proxy geolocation (FIXED)
    def set_timezone_locale_from_proxy(self, driver, proxy):
        """Set browser timezone and locale based on proxy geolocation"""
        try:
            # Get proxy geolocation
            formatted_proxy = format_proxy(proxy)
            response = requests.get("https://ipinfo.io/json", proxies=formatted_proxy, timeout=10, verify=False)

            if response.status_code == 200:
                data = response.json()
                country = data.get("country", "US")
                city = data.get("city", "")

                # Get correct timezone based on actual location
                timezone = get_timezone_for_location(country, city)

                # Set timezone via Chrome DevTools Protocol
                try:
                    driver.execute_cdp_cmd("Emulation.setTimezoneOverride", {"timezoneId": timezone})
                    logger.info(f"✅ Timezone set to {timezone} for {country}")
                except Exception as e:
                    logger.error(f"❌ Failed to set timezone: {e}")
                    # Fallback: set via JavaScript
                    driver.execute_script(f"""
                        Date.prototype.getTimezoneOffset = function() {{
                            const offsets = {{
                                "Europe/Berlin": -120,  // CEST (UTC+2)
                                "America/New_York": 240,  // EDT (UTC-4)
                                "Europe/London": 60,     // BST (UTC+1)
                                "Asia/Tokyo": -540       // JST (UTC+9)
                            }};
                            return offsets["{timezone}"] || 0;
                        }};
                    """)

                # Set locale based on country
                locale_map = {
                    "US": "en-US",
                    "GB": "en-GB",
                    "DE": "de-DE",
                    "FR": "fr-FR",
                    "IT": "it-IT",
                    "ES": "es-ES",
                    "BR": "pt-BR",
                    "IN": "en-IN",
                    "CA": "en-CA",
                    "AU": "en-AU"
                }

                locale = locale_map.get(country, "en-US")

                # Set Accept-Language header
                driver.execute_cdp_cmd("Network.setExtraHTTPHeaders", {
                    "headers": {"Accept-Language": locale}
                })

                # Override navigator.language
                driver.execute_script(f"""
                    Object.defineProperty(navigator, 'language', {{
                        get: function() {{ return '{locale}'; }}
                    }});
                    Object.defineProperty(navigator, 'languages', {{
                        get: function() {{ return ['{locale}', 'en-US']; }}
                    }});
                """)

                logger.info(f"🌍 Set timezone to {timezone} and locale to {locale} based on proxy location {country}")

        except Exception as e:
            logger.error(f"Failed to set timezone/locale from proxy: {e}")

    # NEW: Block WebRTC leaks
    def block_webrtc_leaks(self, driver):
        """Block WebRTC from leaking real IP"""
        # Disable WebRTC using Chrome preferences
        chrome_options = Options()
        chrome_options.add_argument("--disable-webrtc")
        chrome_options.add_argument("--disable-webrtc-encryption")

        # Add WebRTC blocking JavaScript
        webrtc_block_script = """
        // Block WebRTC
        const originalRTC = window.RTCPeerConnection || window.webkitRTCPeerConnection || window.mozRTCPeerConnection;
        if (originalRTC) {
            window.RTCPeerConnection = function(...args) {
                const pc = new originalRTC(...args);
                const originalCreateOffer = pc.createOffer;
                const originalCreateAnswer = pc.createAnswer;
                const originalSetLocalDescription = pc.setLocalDescription;

                pc.createOffer = function(...args) {
                    return originalCreateOffer.apply(this, args).then((desc) => {
                        desc.sdp = desc.sdp.replace(/(IP4 [0-9.]*)/g, 'IP4 0.0.0.0');
                        return desc;
                    });
                };

                pc.createAnswer = function(...args) {
                    return originalCreateAnswer.apply(this, args).then((desc) => {
                        desc.sdp = desc.sdp.replace(/(IP4 [0-9.]*)/g, 'IP4 0.0.0.0');
                        return desc;
                    });
                };

                pc.setLocalDescription = function(...args) {
                    if (args[0] && args[0].sdp) {
                        args[0].sdp = args[0].sdp.replace(/(IP4 [0-9.]*)/g, 'IP4 0.0.0.0');
                    }
                    return originalSetLocalDescription.apply(this, args);
                };

                return pc;
            };
        }
        """

        try:
            driver.execute_script(webrtc_block_script)
            logger.info("🔒 WebRTC leak protection enabled")
        except Exception as e:
            logger.warning(f"Error blocking WebRTC leaks: {e}")
# === TLS fingerprint spoofing ===
class TLSAdapter(requests.adapters.HTTPAdapter):
    def init_poolmanager(self, *args, **kwargs):
        # Set custom TLS settings for JA3 fingerprint spoofing
        import ssl
        context = ssl.create_default_context()
        # Use a common cipher suite to avoid detection
        context.set_ciphers('ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384')
        context.options |= ssl.OP_NO_SSLv2
        context.options |= ssl.OP_NO_SSLv3
        context.options |= ssl.OP_NO_TLSv1
        context.options |= ssl.OP_NO_TLSv1_1
        # Enable HTTP/2
        try:
            import ssl
            if hasattr(ssl, "OP_NO_COMPRESSION"):
                context.options |= ssl.OP_NO_COMPRESSION
        except:
            pass
        kwargs['ssl_context'] = context
        return super(TLSAdapter, self).init_poolmanager(*args, **kwargs)
# === NEW: Realistic HTTP Headers with Ordering and Timing ===
def realistic_http_headers():
    """Generate realistic HTTP headers with proper ordering and timing"""
    # Common header orders for different browsers
    header_orders = {
        "chrome": [
            "Host", "Connection", "Cache-Control", "sec-ch-ua", "sec-ch-ua-mobile",
            "sec-ch-ua-platform", "Upgrade-Insecure-Requests", "User-Agent",
            "Accept", "Sec-Fetch-Site", "Sec-Fetch-Mode", "Sec-Fetch-User",
            "Sec-Fetch-Dest", "Accept-Encoding", "Accept-Language", "Cookie"
        ],
        "firefox": [
            "Host", "User-Agent", "Accept", "Accept-Language", "Accept-Encoding",
            "Connection", "Upgrade-Insecure-Requests", "Sec-Fetch-Dest",
            "Sec-Fetch-Mode", "Sec-Fetch-Site", "Sec-Fetch-User", "Cookie"
        ],
        "safari": [
            "Host", "Connection", "Accept-Encoding", "Accept-Language",
            "User-Agent", "Upgrade-Insecure-Requests", "Cookie"
        ]
    }

    # Select a browser type
    browser_type = random.choice(["chrome", "firefox", "safari"])
    order = header_orders[browser_type]

    # Generate headers
    headers = {
        "User-Agent": "",  # Will be filled later
        "Accept": random.choice([
            "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8",
            "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
            "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
        ]),
        "Accept-Language": random.choice([
            "en-US,en;q=0.9",
            "en-GB,en;q=0.9",
            "en;q=0.9"
        ]),
        "Accept-Encoding": random.choice([
            "gzip, deflate, br",
            "gzip, deflate"
        ]),
        "Connection": random.choice(["keep-alive", "close"]),
        "Upgrade-Insecure-Requests": "1",
        "Sec-Fetch-Dest": random.choice(["document", "empty"]),
        "Sec-Fetch-Mode": random.choice(["navigate", "cors", "no-cors"]),
        "Sec-Fetch-Site": random.choice(["none", "same-origin", "cross-site"]),
        "Cache-Control": random.choice(["max-age=0", "no-cache"])
    }

    # Add browser-specific headers
    if browser_type == "chrome":
        headers["sec-ch-ua"] = random.choice([
            '"Chromium";v="110", "Not A(Brand";v="24", "Google Chrome";v="110"',
            '"Chromium";v="108", "Not A(Brand";v="24", "Google Chrome";v="108"'
        ])
        headers["sec-ch-ua-mobile"] = random.choice(["?0", "?1"])
        headers["sec-ch-ua-platform"] = random.choice([
            '"Windows"', '"macOS"', '"Linux"', '"Android"'
        ])

    # Reorder headers according to the browser's pattern
    ordered_headers = {}
    for header_name in order:
        if header_name in headers:
            ordered_headers[header_name] = headers[header_name]

    # Add any remaining headers
    for header_name, value in headers.items():
        if header_name not in ordered_headers:
            ordered_headers[header_name] = value

    return ordered_headers, browser_type
# === NEW: Cloudflare Challenge Solver ===
def solve_cloudflare_challenge(driver, url):
    """Handle Cloudflare challenges using human-like behavior"""
    # Wait for the challenge page to load
    time.sleep(random.uniform(2, 5))

    # Check if we're facing a Cloudflare challenge
    try:
        cf_challenge = driver.find_element(By.ID, "cf-challenge-stage")
        if cf_challenge:
            logger.info("🛡️ Detected Cloudflare challenge, solving...")

            # Wait a bit as a human would
            time.sleep(random.uniform(3, 7))

            # Look for the verification button
            try:
                verify_button = WebDriverWait(driver, 10).until(
                    EC.element_to_be_clickable((By.ID, "cf-challenge-stage-input"))
                )

                # Human-like movement to the button
                # Note: We'll use a simple click since we don't have the mouse movement function here
                verify_button.click()

                # Wait for the challenge to complete
                time.sleep(random.uniform(5, 10))

                # Check if we're through
                if "cf-challenge-stage" not in driver.page_source:
                    logger.info("✅ Cloudflare challenge solved successfully")
                    return True
                else:
                    logger.warning("⚠️ Cloudflare challenge may not have been solved")

            except Exception as e:
                logger.error(f"❌ Error solving Cloudflare challenge: {e}")

                # Try alternative approach - wait for automatic redirect
                logger.info("⏳ Waiting for automatic redirect...")
                time.sleep(random.uniform(10, 15))

                if "cf-challenge-stage" not in driver.page_source:
                    logger.info("✅ Cloudflare challenge solved automatically")
                    return True

    except Exception:
        # No Cloudflare challenge detected
        pass

    return False
# === NEW: Realistic Session Simulation ===
def simulate_realistic_session(driver, target_url):
    """Simulate a realistic browsing session with random duration and interactions"""
    # Navigate to the target URL
    driver.get(target_url)

    # Random session duration (5-30 minutes)
    session_duration = random.randint(300, 1800)
    session_start = time.time()

    # Random interactions during the session
    while time.time() - session_start < session_duration:
        # Random pause between actions
        time.sleep(random.uniform(5, 30))

        # Random action
        action = random.random()

        if action < 0.3:  # 30% chance to scroll
            # Scroll to a random position
            scroll_to_bottom = random.random() < 0.7  # 70% chance to scroll down
            if scroll_to_bottom:
                # Scroll down
                driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            else:
                # Scroll up
                driver.execute_script("window.scrollTo(0, 0);")

        elif action < 0.5:  # 20% chance to move mouse
            # Move mouse to a random element
            try:
                links = driver.find_elements(By.TAG_NAME, "a")
                buttons = driver.find_elements(By.TAG_NAME, "button")
                clickable_elements = links + buttons

                if clickable_elements:
                    target_element = random.choice(clickable_elements)
                    # Simple click without mouse movement simulation
                    target_element.click()

                    # Small chance to actually click
                    if random.random() < 0.2:  # 20% chance to click
                        time.sleep(random.uniform(2, 5))

                        # 50% chance to go back
                        if random.random() < 0.5:
                            driver.back()
                            time.sleep(random.uniform(2, 5))
            except Exception as e:
                logger.warning(f"Error during mouse movement: {e}")

        elif action < 0.7:  # 20% chance to open a new tab
            # Open a new tab and navigate to a random site
            driver.execute_script("window.open('');")
            driver.switch_to.window(driver.window_handles[-1])

            # Visit a random site
            random_site = random.choice(random_sites)
            driver.get(random_site)
            time.sleep(random.uniform(3, 10))

            # Close the tab
            driver.close()
            driver.switch_to.window(driver.window_handles[0])
            time.sleep(random.uniform(1, 3))

        else:  # 30% chance to do nothing (just wait)
            # Simulate reading content
            time.sleep(random.uniform(10, 60))

    logger.info(f"✅ Completed realistic session of {session_duration} seconds")
# === NEW: Form Interaction Simulation ===
def simulate_form_interaction(driver, form_data=None):
    """Simulate human-like form filling"""
    # Find forms on the page
    forms = driver.find_elements(By.TAG_NAME, "form")
    if not forms:
        return False

    # Select a random form
    form = random.choice(forms)

    # Find input fields in the form
    inputs = form.find_elements(By.TAG_NAME, "input")
    textareas = form.find_elements(By.TAG_NAME, "textarea")
    selects = form.find_elements(By.TAG_NAME, "select")

    fields = inputs + textareas + selects
    if not fields:
        return False

    # Randomly decide whether to fill the form (50% chance)
    if random.random() < 0.5:
        logger.info("📝 Simulating form interaction...")

        # Fill each field with human-like behavior
        for field in fields:
            field_type = field.get_attribute("type")
            tag_name = field.tag_name.lower()

            # Skip hidden fields and submit buttons
            if field_type == "hidden" or field_type == "submit" or field.is_displayed() == False:
                continue

            # Scroll to the field if needed
            driver.execute_script("arguments[0].scrollIntoView({block: 'center'});", field)
            time.sleep(random.uniform(0.5, 1.5))

            # Click on the field
            field.click()
            time.sleep(random.uniform(0.3, 1.0))

            # Fill the field based on its type
            if tag_name == "select":
                # Select a random option
                options = field.find_elements(By.TAG_NAME, "option")
                if options:
                    option = random.choice(options)
                    option.click()
                    time.sleep(random.uniform(0.5, 1.0))
            elif field_type in ["text", "email", "password", "search", "tel", "url"]:
                # Type text with human-like cadence
                if form_data and field.get_attribute("name") in form_data:
                    text = form_data[field.get_attribute("name")]
                else:
                    # Generate realistic fake data based on field type
                    if field_type == "email":
                        text = f"{''.join(random.choices('abcdefghijklmnopqrstuvwxyz', k=8))}@{random.choice(['gmail.com', 'yahoo.com', 'outlook.com'])}"
                    elif field_type == "password":
                        text = ''.join(random.choices('abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*', k=12))
                    elif field_type == "tel":
                        text = f"{random.randint(100, 999)}-{random.randint(100, 999)}-{random.randint(1000, 9999)}"
                    else:
                        text = ''.join(random.choices('abcdefghijklmnopqrstuvwxyz ', k=random.randint(5, 15)))

                # Type with realistic timing
                for char in text:
                    field.send_keys(char)
                    time.sleep(random.uniform(0.05, 0.2))

                # Random pause after typing
                time.sleep(random.uniform(0.5, 2.0))

                # Sometimes tab to the next field
                if random.random() < 0.3:  # 30% chance
                    field.send_keys(Keys.TAB)
                    time.sleep(random.uniform(0.3, 1.0))
            elif tag_name == "textarea":
                # Type longer text with human-like cadence
                if form_data and field.get_attribute("name") in form_data:
                    text = form_data[field.get_attribute("name")]
                else:
                    # Generate a paragraph of text
                    words = [
                        "the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog",
                        "this", "is", "a", "sample", "text", "for", "form", "filling",
                        "human", "behavior", "simulation", "requires", "realistic", "timing"
                    ]
                    text = ' '.join(random.choices(words, k=random.randint(10, 30)))

                # Type with realistic timing
                for char in text:
                    field.send_keys(char)
                    time.sleep(random.uniform(0.05, 0.15))

                # Random pause after typing
                time.sleep(random.uniform(0.5, 2.0))

        # Decide whether to submit the form (70% chance)
        if random.random() < 0.7:
            # Find the submit button
            submit_buttons = form.find_elements(By.XPATH, ".//input[@type='submit']") + \
                           form.find_elements(By.XPATH, ".//button[@type='submit']") + \
                           form.find_elements(By.XPATH, ".//button[contains(text(), 'Submit')]") + \
                           form.find_elements(By.XPATH, ".//button[contains(text(), 'Send')]")

            if submit_buttons:
                submit_button = random.choice(submit_buttons)
                submit_button.click()
                time.sleep(random.uniform(2, 5))

                # 50% chance to go back after submission
                if random.random() < 0.5:
                    driver.back()
                    time.sleep(random.uniform(2, 5))

        return True

    return False
# === NEW: Environment Consistency ===
def ensure_consistent_environment(driver, proxy_info=None):
    """Ensure timezone, language, and other settings match the IP geolocation"""
    # Get IP geolocation if proxy is used
    if proxy_info and config.enable_ip_geolocation_matching:
        try:
            proxy = format_proxy(proxy_info)
            response = requests.get("https://ipinfo.io/json", proxies=proxy, timeout=10, verify=False)
            if response.status_code == 200:
                ip_info = response.json()
                country = ip_info.get("country", "US")
                timezone = get_timezone_for_location(country)  # Use our new function

                # Set browser timezone to match IP location
                driver.execute_script(f"""
                    // Set timezone
                    Date.prototype.getTimezoneOffset = function() {{
                        const offset = new Date().toLocaleString("en", {{timeZoneName: "short"}}).split(" ").pop();
                        const timezoneOffsets = {{
                                    "EST": -300, "EDT": -240,  // Eastern
                                    "CST": -360, "CDT": -300,  // Central
                                    "MST": -420, "MDT": -360,  // Mountain
                                    "PST": -480, "PDT": -420,  // Pacific
                                    "GMT": 0, "BST": 60,       // UK
                                    "CET": 60, "CEST": 120,    // Central Europe
                                    "EET": 120, "EEST": 180,   // Eastern Europe
                                    "MSK": 180, "MSD": 240,    // Moscow
                                    "CST": 480, "CDT": 540,    // China Standard
                                    "JST": 540, "KST": 540    // Japan/Korea
                                }};
                        return -(timezoneOffsets[offset] || 0);
                    }};

                    // Set timezone for Intl API
                    Intl.DateTimeFormat.prototype.resolvedOptions = function() {{
                        return {{
                            locale: navigator.language,
                            calendar: "gregorian",
                            numberingSystem: "latn",
                            timeZone: "{timezone}",
                            hourCycle: "h23"
                        }};
                    }};
                """)

                # Set language based on country
                country_languages = {
                    "US": "en-US",
                    "UK": "en-GB",
                    "DE": "de-DE",
                    "FR": "fr-FR",
                    "IT": "it-IT",
                    "ES": "es-ES",
                    "BR": "pt-BR",
                    "IN": "hi-IN",
                    "CA": "en-CA",
                    "AU": "en-AU",
                    "JP": "ja-JP",
                    "KR": "ko-KR",
                    "CN": "zh-CN",
                    "RU": "ru-RU"
                }

                language = country_languages.get(country, "en-US")

                # Set browser language
                driver.execute_script(f"""
                    Object.defineProperty(navigator, 'language', {{
                        get: () => '{language}',
                    }});

                    Object.defineProperty(navigator, 'languages', {{
                        get: () => ['{language}', 'en'],
                    }});
                """)

                logger.info(f"🌍 Set environment to match {country} timezone and language")

        except Exception as e:
            logger.warning(f"Failed to set consistent environment: {e}")
# === NEW: Device Parameter Consistency ===
def ensure_consistent_device_parameters(driver, device_info):
    """Ensure all device parameters are consistent with each other"""
    # Get screen resolution from device info
    resolution = device_info.get("screen_resolution", "1920x1080")
    width, height = map(int, resolution.split('x'))

    # Calculate viewport size (typically smaller than screen resolution)
    viewport_height = height - random.randint(50, 150)

    # Set consistent screen parameters
    driver.execute_script(f"""
        // Set screen resolution
        Object.defineProperty(screen, 'width', {{
            get: () => {width},
        }});

        Object.defineProperty(screen, 'height', {{
            get: () => {height},
        }});

        Object.defineProperty(screen, 'availWidth', {{
            get: () => {width},
        }});

        Object.defineProperty(screen, 'availHeight', {{
            get: () => {height - random.randint(30, 60)},
        }});

        Object.defineProperty(screen, 'colorDepth', {{
            get: () => 24,
        }});

        Object.defineProperty(screen, 'pixelDepth', {{
            get: () => 24,
        }});

        // Set viewport size
        Object.defineProperty(window, 'innerWidth', {{
            get: () => {width},
        }});

        Object.defineProperty(window, 'innerHeight', {{
            get: () => {viewport_height},
        }});

        Object.defineProperty(window, 'outerWidth', {{
            get: () => {width},
        }});

        Object.defineProperty(window, 'outerHeight', {{
            get: () => {height},
        }});

        // Set device pixel ratio
        Object.defineProperty(window, 'devicePixelRatio', {{
            get: () => {random.choice([1, 1.5, 2])},
        }});
    """)

    # Set hardware concurrency based on device type
    device_type = device_info.get("device_type", "Unknown")
    if device_type in ["iPhone", "iPad", "Samsung", "GooglePixel", "Motorola", "OnePlus", "Vivo", "Xiaomi", "ZTE"]:
        # Mobile devices typically have 4-8 cores
        hardware_concurrency = random.randint(4, 8)
    else:
        # Desktop devices typically have 4-16 cores
        hardware_concurrency = random.randint(4, 16)

    driver.execute_script(f"""
        // Set hardware concurrency
        Object.defineProperty(navigator, 'hardwareConcurrency', {{
            get: () => {hardware_concurrency},
        }});

        // Set device memory based on device type
        Object.defineProperty(navigator, 'deviceMemory', {{
            get: () => {random.choice([2, 4, 6, 8])},
        }});
    """)

    logger.info(f"📱 Set consistent device parameters for {device_type}")
# === Select device with percentage distribution ===
def select_device():
    # Create a pool based on percentage distribution
    pool = []
    for device, percentage in device_percentage_distribution.items():
        # Add the device to the pool according to its percentage
        pool.extend([device] * percentage)

    # Shuffle the pool to ensure randomness
    random.shuffle(pool)

    # Return a random device from the pool
    return random.choice(pool)
# === Get and remove user agent ===
def get_and_remove_ua(device_name, agents_list, filename):
    if not agents_list:
        logger.error(f"❌ No more UAs left for {device_name}")
        return None

    # Select a random user agent
    ua = random.choice(agents_list)

    # Remove from the in-memory list
    agents_list.remove(ua)

    # Remove from the file if configured
    if config.remove_ua_after_use:
        remove_line_from_file(filename, ua)

    # Return the user agent as is (without randomization)
    return ua
# === Extract phone model from UA ===
def extract_phone_model(ua):
    # Use the enhanced device fingerprint parser
    device_fingerprint = DeviceFingerprint(ua)
    device_info = device_fingerprint.device_info

    if device_info.get('device_model') != 'Unknown':
        return device_info['device_model']

    # Fallback to simple extraction
    match = re.search(r'FBDV/([^;]+)', ua)
    if match:
        return match.group(1)
    if "iPhone" in ua:
        return "iPhone"
    if "Samsung" in ua or "SM-" in ua:
        return "Samsung"
    if "GooglePixel" in ua or "Pixel" in ua:
        return "GooglePixel"
    if "Motorola" in ua:
        return "Motorola"
    if "OnePlus" in ua:
        return "OnePlus"
    if "Vivo" in ua:
        return "Vivo"
    if "Xiaomi" in ua:
        return "Xiaomi"
    if "ZTE" in ua:
        return "ZTE"
    if "Windows" in ua:
        return "Windows PC"
    return "Unknown Device"
# === Enhanced IP info retrieval (FIXED) ===
def get_ip_info(proxy):
    try:
        # Try multiple sources for more reliable data
        sources = [
            "https://ipinfo.io/json",
            "https://ipapi.co/json/"
        ]

        for url in sources:
            try:
                r = requests.get(url, proxies=proxy, timeout=10, verify=False)
                data = r.json()

                # Extract consistent data
                country = data.get("country_code") or data.get("country")
                city = data.get("city") or data.get("region_name")
                region = data.get("region") or data.get("region_name")

                # Use our timezone function based on country
                timezone = get_timezone_for_location(country)

                isp = data.get("org") or data.get("isp") or data.get("organization")

                # Get coordinates
                if "loc" in data:
                    lat, lng = data["loc"].split(",")
                else:
                    lat = data.get("latitude", "0")
                    lng = data.get("longitude", "0")

                return (
                    data.get("ip", ""),
                    city,
                    country,
                    region,
                    timezone,
                    isp,
                    lat,
                    lng
                )
            except:
                continue

        # Fallback if all sources fail
        return "Unknown", "Unknown", "Unknown", "Unknown", "UTC", "Unknown ISP", "0", "0"
    except Exception as e:
        logger.error(f"IP info retrieval failed: {e}")
        return "Unknown", "Unknown", "Unknown", "Unknown", "UTC", "Unknown ISP", "0", "0"
# === Get local time ===
def get_local_time(tz):
    try:
        now = datetime.datetime.now(pytz.timezone(tz))
    except:
        now = datetime.datetime.utcnow()
    return now.strftime("%Y-%m-%d %H:%M:%S %Z")
# === ENHANCED: Simulate scroll behavior with more randomness ===
def simulate_scroll():
    # Simulate realistic scroll behavior with more randomness
    scroll_events = []
    total_scroll = random.randint(500, 1500)
    scroll_events.append({"y": 0, "delay": random.randint(100, 500)})

    current_y = 0
    while current_y < total_scroll:
        # More variable scroll amounts and speeds
        scroll_amount = random.randint(30, 250)

        # Add random pauses during scrolling (human behavior)
        if random.random() < 0.2:  # 20% chance of a pause
            pause_time = random.randint(200, 1500)
            scroll_events.append({"y": current_y, "delay": pause_time})

        # Variable scroll speed
        scroll_speed = random.uniform(0.3, 2.5)
        current_y += scroll_amount
        if current_y > total_scroll:
            current_y = total_scroll
        scroll_events.append({"y": current_y, "delay": int(scroll_speed * 1000)})

    # Add more random scroll ups with variable speeds
    num_scroll_ups = random.randint(1, 4)
    for _ in range(num_scroll_ups):
        scroll_up = random.randint(30, 200)
        scroll_speed = random.uniform(0.3, 2.5)
        current_y = max(0, current_y - scroll_up)
        scroll_events.append({"y": current_y, "delay": int(scroll_speed * 1000)})

        # Add random pauses during scroll up
        if random.random() < 0.3:  # 30% chance of a pause during scroll up
            pause_time = random.randint(200, 1500)
            scroll_events.append({"y": current_y, "delay": pause_time})

    # Scroll to bottom
    scroll_events.append({"y": total_scroll, "delay": random.randint(100, 500)})

    return scroll_events
# === ENHANCED: Simulate mouse movement with more imperfections ===
def simulate_mouse_movement():
    # Simulate realistic mouse movement with more imperfections
    movements = []
    current_x, current_y = random.randint(100, 500), random.randint(100, 500)

    # Random number of movement sequences
    num_sequences = random.randint(3, 12)

    for _ in range(num_sequences):
        target_x = random.randint(50, 1000)
        target_y = random.randint(50, 800)

        # Control points for Bezier curve with more randomness
        cp1_x = current_x + random.randint(-300, 300)
        cp1_y = current_y + random.randint(-300, 300)
        cp2_x = target_x + random.randint(-300, 300)
        cp2_y = target_y + random.randint(-300, 300)

        # Variable number of steps for each movement
        steps = random.randint(8, 25)

        # Add random micro-movements and hesitations
        for i in range(steps + 1):
            t = i / steps
            # Bezier curve formula
            x = (1-t)**3 * current_x + 3*(1-t)**2*t * cp1_x + 3*(1-t)*t**2 * cp2_x + t**3 * target_x
            y = (1-t)**3 * current_y + 3*(1-t)**2*t * cp1_y + 3*(1-t)*t**2 * cp2_y + t**3 * target_y

            # Add random micro-movements
            if random.random() < 0.1:  # 10% chance of micro-movement
                x += random.randint(-5, 5)
                y += random.randint(-5, 5)

            # Variable speed with more randomness
            base_speed = random.uniform(0.01, 0.15)

            # Add random hesitations (pauses during movement)
            if random.random() < 0.05:  # 5% chance of hesitation
                hesitation = random.randint(100, 500)
                movements.append({"x": int(x), "y": int(y), "delay": hesitation})

            movements.append({"x": int(x), "y": int(y), "delay": int(base_speed * 1000)})

        # Sometimes move to a random intermediate point
        if random.random() < 0.3:  # 30% chance of intermediate point
            intermediate_x = random.randint(50, 1000)
            intermediate_y = random.randint(50, 800)

            # Move to intermediate point
            int_steps = random.randint(3, 8)
            for i in range(int_steps + 1):
                t = i / int_steps
                x = current_x + (intermediate_x - current_x) * t
                y = current_y + (intermediate_y - current_y) * t

                # Variable speed
                speed = random.uniform(0.02, 0.1)
                movements.append({"x": int(x), "y": int(y), "delay": int(speed * 1000)})

            current_x, current_y = intermediate_x, intermediate_y
        else:
            current_x, current_y = target_x, target_y

    return movements
# === Enhanced JS environment simulation ===
def simulate_js_env(headers, ua, device_fingerprint, device_type, ip_info):
    # Create cookies
    cookies = SimpleCookie()

    # Generate encrypted cookies if enabled
    if config.enable_encryption and cipher_suite:
        session_id = cipher_suite.encrypt(f"{random.randint(10000000, 99999999)}-{random.randint(1000,9999)}".encode()).decode()
        user_id = cipher_suite.encrypt(f"{random.randint(10000, 99999)}".encode()).decode()
    else:
        session_id = f"{random.randint(10000000, 99999999)}-{random.randint(1000,9999)}"
        user_id = f"{random.randint(10000, 99999)}"

    cookies["sessionid"] = session_id
    cookies["userid"] = user_id

    # Get fingerprint data
    fingerprint_data = device_fingerprint.get_all_fingerprints(ip_info)

    # Get device info
    device_info = fingerprint_data.get("device_info", {})

    # Get a random referer from the loaded referers
    referer = random.choice(referers) if referers else "https://www.google.com/"

    # Generate a random click ID
    click_id = hashlib.md5(f"{random.randint(10000000, 99999999)}-{time.time()}".encode()).hexdigest()

    # Update headers with realistic, standard data
    headers.update({
        "Accept-Language": random.choice(["en-US,en;q=0.9", "es-US,es;q=0.9", "en-GB,en;q=0.8"]),
        "Upgrade-Insecure-Requests": "1",
        "Sec-Fetch-Site": "none",
        "Sec-Fetch-Mode": "navigate",
        "Sec-Fetch-User": "?1",
        "Sec-Fetch-Dest": "document",
        "Referer": referer,
        "Origin": f"https://{urlparse(config.cpa_url).netloc}",
        "Cookie": "; ".join([f"{key}={val.value}" for key, val in cookies.items()]),
        "DNT": "1",  # Do Not Track
        "Cache-Control": "no-cache",
        "Pragma": "no-cache",
    })

    # Add Client-Hint headers which are more realistic than custom X- headers
    if "Chrome" in device_info.get("browser_name", ""):
        headers.update({
            "Sec-CH-UA": f'"Chromium";v="{random.randint(100, 120)}", "Google Chrome";v="{random.randint(100, 120)}", "Not-A.Brand";v="99"',
            "Sec-CH-UA-Mobile": "?1" if "iPhone" in device_type or "Android" in device_type else "?0",
            "Sec-CH-UA-Platform": f'"{device_info.get("os_name", "Android")}"',
        })

    return headers, click_id
# === ENHANCED: Smart delay ===
def smart_delay():
    # Introduce realistic mouse and scroll movement delays
    # Use a more complex distribution to simulate human behavior
    delay_type = random.choice(['short', 'medium', 'long'])

    if delay_type == 'short':
        delay = random.uniform(0.5, 2.0)
    elif delay_type == 'medium':
        delay = random.uniform(2.0, 5.0)
    else:  # long
        delay = random.uniform(5.0, 10.0)

    # Add some randomness to the delay
    delay *= random.uniform(0.8, 1.2)

    # Ensure minimum delay
    delay = max(0.5, delay)

    time.sleep(delay)
# === ENHANCED: Visit random sites with more randomness ===
def shuffle_and_visit_sites(sites, headers, proxy, session):
    """Visit random sites and return the list of successfully visited sites"""
    # Randomize the number of sites to visit (2-8)
    num_sites = random.randint(config.min_pre_click_sites, config.max_pre_click_sites)

    if num_sites == 0:
        logger.info("🌐 Skipping site visits (random choice)")
        return []

    visited = random.sample(sites, k=min(num_sites, len(sites)))

    # Add a random Facebook profile to the visited sites
    fb_id = f"1000{random.randint(10000000,99999999)}"
    fb_url = f"https://www.facebook.com/profile.php?id={fb_id}"
    visited.append(fb_url)

    # Shuffle the list to randomize the order
    random.shuffle(visited)

    successfully_visited = []

    for site in visited:
        try:
            # Disable SSL verification for this request
            r = session.get(site, headers=headers, timeout=10, verify=False)
            logger.info(f"🌐 Visited: {site} | Status: {r.status_code}")

            # Only add to successfully_visited if status code is 2xx
            if 200 <= r.status_code < 300:
                successfully_visited.append(site)

            # Randomly decide whether to scroll on this site
            if random.random() < 0.7:  # 70% chance to scroll
                # Simulate scroll behavior on the site
                scroll_events = simulate_scroll()
                for event in scroll_events:
                    logger.debug(f"🖱️ Scrolled to Y: {event['y']} | Delay: {event['delay']}ms")
                    time.sleep(event['delay'] / 1000)

            # Randomly decide whether to move mouse on this site
            if random.random() < 0.6:  # 60% chance to move mouse
                # Simulate mouse movement
                movements = simulate_mouse_movement()
                for movement in movements:
                    logger.debug(f"🖱️ Mouse moved to X: {movement['x']}, Y: {movement['y']} | Delay: {movement['delay']}ms")
                    time.sleep(movement['delay'] / 1000)

            # Random delay between sites
            if random.random() < 0.5:  # 50% chance of delay
                delay = random.uniform(0.5, 3.0)
                time.sleep(delay)

        except Exception as e:
            logger.error(f"⚠️ Visit failed: {site} | {e}")

    return successfully_visited
# === Banner ===
def banner():
    console.print(Panel.fit(
        "[bold green]WELCOME TO WORLD BEST UNDETECTABLE BOT[/bold green]\n"
        "[bold blue]MADE BY STARBOY MACCLUM ARMAN[/bold blue]\n"
        "[bold red]HACKER GANG YOU ARE UNDETECTABLE[/bold red]",
        title="🚀 ULTIMATE CPA BOT 🚀"
    ))
# === Log click ===
def log_click(phone_model, proxy, ip, city, region, country, time_str, isp, device_id, click_id):
    log_line = (f"✅ CLICK SUCCESS | UA: {phone_model} | Proxy: {proxy} | IP: {ip} | "
                f"City: {city}, State: {region}, Country: {country} | Time: {time_str} | "
                f"ISP: {isp} | Device ID: {device_id} | Click ID: {click_id}\n")
    with open("click_log.txt", "a", encoding="utf-8") as f:
        f.write(log_line)
# === Print clicks left ===
def print_clicks_left():
    console.print(f"🎯 Clicks Left per Device - iPhone: {len(iphone_agents)}, Samsung: {len(samsung_agents)}, "
          f"Windows: {len(windows_agents)}, GooglePixel: {len(googlepixel_agents)}, Motorola: {len(motorola_agents)}, "
          f"OnePlus: {len(oneplus_agents)}, Vivo: {len(vivo_agents)}, Xiaomi: {len(xiaomi_agents)}, ZTE: {len(zte_agents)}")
# === Select proxy based on rotation strategy ===
def select_proxy(proxies, proxy_scores, rotation_strategy, used_proxies):
    if rotation_strategy == "random":
        available_proxies = [p for p in proxies if p not in used_proxies]
        if not available_proxies:
            return None
        return random.choice(available_proxies)

    elif rotation_strategy == "round_robin":
        available_proxies = [p for p in proxies if p not in used_proxies]
        if not available_proxies:
            return None
        # Simple round robin: return the first available proxy
        return available_proxies[0]

    elif rotation_strategy == "geographic":
        # For geographic rotation, we'd need to group proxies by location
        # For simplicity, we'll just use random rotation here
        available_proxies = [p for p in proxies if p not in used_proxies]
        if not available_proxies:
            return None
        return random.choice(available_proxies)

    elif rotation_strategy == "performance":
        # Sort proxies by score (highest first)
        sorted_proxies = sorted(proxy_scores.items(), key=lambda x: x[1], reverse=True)
        for proxy, score in sorted_proxies:
            if proxy not in used_proxies:
                return proxy
        return None

    else:
        # Default to random
        available_proxies = [p for p in proxies if p not in used_proxies]
        if not available_proxies:
            return None
        return random.choice(available_proxies)
# === CAPTCHA Solver ===
def solve_captcha(site_key, url):
    if not config.enable_captcha_solver or not config.captcha_api_key:
        return None

    try:
        if config.captcha_service == "2captcha":
            # 2Captcha API
            api_url = "http://2captcha.com/in.php"
            params = {
                "key": config.captcha_api_key,
                "method": "userrecaptcha",
                "googlekey": site_key,
                "pageurl": url,
                "json": 1
            }

            response = requests.post(api_url, data=params, verify=False)
            result = response.json()

            if result.get("status") == 1:
                captcha_id = result.get("request")
                # Wait for solution
                time.sleep(15)  # Initial wait
                result_url = f"http://2captcha.com/res.php?key={config.captcha_api_key}&action=get&id={captcha_id}&json=1"
                while True:
                    response = requests.get(result_url, verify=False)
                    result = response.json()

                    if result.get("status") == 1:
                        return result.get("request")
                    elif result.get("request") == "CAPCHA_NOT_READY":
                        time.sleep(5)
                    else:
                        break

        elif config.captcha_service == "anticaptcha":
            # Anti-Captcha API
            api_url = "https://api.anti-captcha.com/createTask"
            payload = {
                "clientKey": config.captcha_api_key,
                "task": {
                    "type": "RecaptchaV2TaskProxyless",
                    "websiteURL": url,
                    "websiteKey": site_key
                }
            }

            response = requests.post(api_url, json=payload, verify=False)
            result = response.json()

            if result.get("errorId") == 0:
                task_id = result.get("taskId")
                # Wait for solution
                time.sleep(15)  # Initial wait

                result_url = "https://api.anti-captcha.com/getTaskResult"
                payload = {
                    "clientKey": config.captcha_api_key,
                    "taskId": task_id
                }

                while True:
                    response = requests.post(result_url, json=payload, verify=False)
                    result = response.json()

                    if result.get("status") == "ready":
                        return result.get("solution", {}).get("gRecaptchaResponse")
                    elif result.get("status") == "processing":
                        time.sleep(5)
                    else:
                        break
    except Exception as e:
        logger.error(f"⚠️ CAPTCHA solving failed: {e}")

    return None
# === NEW: Calculate rest period ===
def calculate_rest_period():
    """Calculate a rest period to simulate human breaks"""
    if not config.enable_rest_periods:
        return 0

    # Only take a rest with the configured probability
    if random.random() > config.rest_probability:
        return 0

    # Calculate rest period duration
    rest_period = random.randint(config.min_rest_period, config.max_rest_period)
    logger.info(f"😴 Taking a rest period of {rest_period // 60} minutes...")
    return rest_period
# === NEW: Check if within operating hours ===
def is_within_operational_hours():
    """Check if current time is within configured operating hours"""
    if not config.enable_operational_hours:
        return True

    current_hour = datetime.datetime.now().hour
    start_hour = config.operating_hours_start
    end_hour = config.operating_hours_end

    # Handle case where end hour is after midnight (e.g., 23 to 8)
    if end_hour < start_hour:
        return current_hour >= start_hour or current_hour < end_hour
    else:
        return start_hour <= current_hour < end_hour
# === Calculate human-like wait time ===
def calculate_wait_time():
    """Calculate a wait time that mimics human behavior patterns"""
    if not config.enable_human_timing_patterns:
        # Simple random wait time
        return random.randint(config.min_wait_between_clicks, config.max_wait_between_clicks)

    # Use the natural timing patterns
    return calculate_natural_wait_time()
# === NEW: Monitor success rates ===
def monitor_success_rates():
    """Monitor success rates and alert if they drop below threshold"""
    if not config.enable_success_monitoring:
        return

    total_success = sum(device_success_rates[device]["success"] for device in device_success_rates)
    total_attempts = sum(device_success_rates[device]["total"] for device in device_success_rates)

    if total_attempts == 0:
        return

    success_rate = total_success / total_attempts

    if success_rate < config.success_rate_threshold:
        logger.warning(f"⚠️ Success rate dropped to {success_rate:.2%} (threshold: {config.success_rate_threshold:.2%})")
        logger.warning("Consider adjusting settings or taking a break to avoid detection")

        # Log detailed device success rates
        for device in device_success_rates:
            device_success = device_success_rates[device]["success"]
            device_total = device_success_rates[device]["total"]
            if device_total > 0:
                device_rate = device_success / device_total
                logger.info(f"   {device}: {device_success}/{device_total} ({device_rate:.2%})")
# === NEW: Get random viewport size ===
def get_random_viewport_size(device_type):
    """Generate a random viewport size based on device type"""
    if "iPhone" in device_type or "iPad" in device_type:
        # Mobile viewport sizes
        viewport_sizes = [
            (375, 667),   # iPhone 6/7/8
            (375, 812),   # iPhone X/11/12
            (414, 736),   # iPhone 6/7/8 Plus
            (414, 896),   # iPhone 11 Pro Max
            (390, 844),   # iPhone 12/13 Pro
            (428, 926),   # iPhone 12/13 Pro Max
            (320, 568),   # iPhone 5/SE
        ]
    elif "Windows" in device_type:
        # Desktop viewport sizes
        viewport_sizes = [
            (1366, 768),
            (1440, 900),
            (1536, 864),
            (1920, 1080),
            (2560, 1440),
            (1024, 768),
            (1280, 720),
            (1600, 900),
        ]
    else:
        # Android viewport sizes
        viewport_sizes = [
            (360, 640),
            (360, 740),
            (411, 731),
            (412, 846),
            (393, 786),
            (393, 851),
            (412, 892),
        ]

    return random.choice(viewport_sizes)
# === NEW: Simulate browser extensions ===
def simulate_browser_extensions(driver):
    """Simulate random browser extensions"""
    if not config.enable_browser_extension_simulation:
        return

    # List of common browser extensions to simulate
    common_extensions = [
        "AdBlock",
        "uBlock Origin",
        "LastPass",
        "1Password",
        "Grammarly",
        "Honey",
        "HTTPS Everywhere",
        "Dark Reader",
        "Video DownloadHelper",
        "Pocket",
        "Evernote Web Clipper",
        "Save to Google Drive",
        "Markdown Viewer",
        "JSON Viewer",
        "ColorZilla",
        "WhatFont",
        "Session Buddy",
        "Tab Wrangler",
        "The Great Suspender",
        "OneTab",
        "Bookmark Manager",
        "History Manager",
        "Google Translate",
        "WordReference",
        "Merriam-Webster Dictionary",
    ]

    # Randomly select 3-7 extensions to simulate
    num_extensions = random.randint(3, 7)
    selected_extensions = random.sample(common_extensions, num_extensions)

    # Create a properly formatted JavaScript string
    extensions_list = ', '.join(f'"{ext}"' for ext in selected_extensions)

    # Inject JavaScript to simulate these extensions
    extensions_script = f"""
    // Simulate browser extensions
    window.chrome = window.chrome || {{}};
    window.chrome.runtime = window.chrome.runtime || {{}};

    // Create simulated extension objects
    const extensions = [{extensions_list}];
    window.chrome.runtime.id = 'abcdefghijklmnop';

    // Add extension metadata to navigator
    Object.defineProperty(navigator, 'plugins', {{
        get: function() {{
            const plugins = [];

            // Add simulated plugins for each extension
            extensions.forEach(ext => {{
                plugins.push({{
                    name: ext + ' Extension',
                    filename: ext.toLowerCase().replace(' ', '-') + '.js',
                    description: ext + ' browser extension',
                    version: '{random.randint(1, 5)}.{random.randint(0, 9)}.{random.randint(0, 9)}',
                    length: 1
                }});
            }});

            return plugins;
        }}
    }});

    // Add extension-related properties to navigator
    Object.defineProperty(navigator, 'mimeTypes', {{
        get: function() {{
            const mimeTypes = [];

            // Add MIME types for each extension
            extensions.forEach(ext => {{
                mimeTypes.push({{
                    type: 'application/x-' + ext.toLowerCase().replace(' ', '-'),
                    description: ext + ' Extension',
                    suffixes: ext.toLowerCase().replace(' ', '-'),
                    enabledPlugin: {{
                        name: ext + ' Extension',
                        filename: ext.toLowerCase().replace(' ', '-') + '.js'
                    }}
                }});
            }});

            return mimeTypes;
        }}
    }});
    """

    try:
        driver.execute_script(extensions_script)
        logger.info(f"🧩 Simulated {num_extensions} browser extensions")
    except Exception as e:
        logger.warning(f"Error simulating browser extensions: {e}")
# === FIXED: Process single click with improved error handling ===
def process_click(click_num, proxy, device, ua, device_fingerprint, ip_info, session, retry_count=3):
    visited_sites = []  # Initialize list to track visited sites

    try:
        # Setup Selenium WebDriver for realistic browser simulation
        options = Options()

        # Set headless mode
        if config.use_headless_browser:
            options.add_argument("--headless")

        # Set user agent
        options.add_argument(f"--user-agent={ua}")

        # Set proxy
        proxy_str = proxy.split("://")[1] if "://" in proxy else proxy
        if "@" in proxy_str:
            auth, proxy_addr = proxy_str.split("@")
            username, password = auth.split(":")
            options.add_argument(f"--proxy-server={proxy_addr}")
            options.add_argument(f"--proxy-auth={username}:{password}")
        else:
            options.add_argument(f"--proxy-server={proxy_str}")

        # Add additional options for better undetectability
        options.add_argument("--disable-blink-features=AutomationControlled")
        options.add_argument("--disable-dev-shm-usage")
        options.add_argument("--no-sandbox")
        options.add_argument("--disable-gpu")
        options.add_argument("--disable-web-security")
        options.add_argument("--disable-features=VizDisplayCompositor")

        # Disable WebGPU if configured
        if config.disable_webgpu:
            options.add_argument("--disable-webgpu")

        # Set random viewport size if configured
        if config.enable_random_viewport_sizes:
            viewport_width, viewport_height = get_random_viewport_size(device)
            options.add_argument(f"--window-size={viewport_width},{viewport_height}")

        # Create WebDriver
        service = Service()
        driver = webdriver.Chrome(service=service, options=options)

        try:
            # Set window size if not already set
            if not config.enable_random_viewport_sizes:
                driver.set_window_size(1920, 1080)

            # Start timing the click processing
            click_start_time = time.time()

            # 1. Perform quick warmup session (max 2 minutes)
            quick_warmup_session(driver, max_time=120)

            # 2. Calculate remaining time for pre-click interactions
            elapsed_time = time.time() - click_start_time
            remaining_time = max(0, 120 - elapsed_time)

            # 3. Perform random pre-click interactions within the remaining time
            if config.enable_session_simulation and remaining_time > 0:
                try:
                    device_fingerprint.random_pre_click_interaction(driver, max_time=remaining_time)
                except Exception as e:
                    logger.warning(f"Error during enhanced session simulation: {e}")

            # 4. Inject WebGL/Canvas spoofing
            if config.enable_canvas_fingerprinting:
                device_fingerprint.inject_webgl_canvas_spoofing(driver)

            # 5. Set timezone and locale based on proxy
            if config.enable_environment_consistency:
                device_fingerprint.set_timezone_locale_from_proxy(driver, proxy)

            # 6. Block WebRTC leaks if enabled
            if config.enable_webrtc_protection:
                device_fingerprint.block_webrtc_leaks(driver)

            # 7. Simulate browser extensions if enabled
            if config.enable_browser_extension_simulation:
                simulate_browser_extensions(driver)

            # 8. Ensure device consistency
            if config.enable_device_consistency:
                device_specs = ensure_device_consistency(ua, device_fingerprint)
                ensure_consistent_device_parameters(driver, device_specs)

            # Get realistic HTTP headers with ordering and timing
            headers, browser_type = realistic_http_headers()
            headers["User-Agent"] = ua  # Set the user agent

            # Enhance headers with fingerprint data
            headers, click_id = simulate_js_env(headers, ua, device_fingerprint, device, ip_info)

            # Visit random sites first and get the list of visited sites
            try:
                visited_sites = shuffle_and_visit_sites(random_sites, headers, proxy, session)
            except Exception as e:
                logger.warning(f"Error visiting random sites: {e}")

            # Generate realistic cache for the successfully visited sites
            if visited_sites:
                try:
                    generate_realistic_cache(visited_sites)
                except Exception as e:
                    logger.warning(f"Error generating cache: {e}")

            # Get IP info again to ensure consistency
            ip, city, country, region, tz, isp, lat, lng = ip_info

            # Visit CPA offer
            driver.get(config.cpa_url)

            # Check for CAPTCHA
            if "captcha" in driver.page_source.lower() or "recaptcha" in driver.page_source.lower():
                logger.warning("⚠️ CAPTCHA detected! Attempting to solve...")
                # Extract site key (simplified)
                site_key_match = re.search(r'sitekey="([^"]+)"', driver.page_source)
                if site_key_match:
                    site_key = site_key_match.group(1)
                    captcha_solution = solve_captcha(site_key, config.cpa_url)
                    if captcha_solution:
                        # Submit CAPTCHA solution
                        driver.execute_script(f"""
                            document.querySelector('[name="g-recaptcha-response"]').value = '{captcha_solution}';
                            document.querySelector('form').submit();
                        """)
                        logger.info("✅ CAPTCHA solved successfully!")
                    else:
                        logger.error("❌ Failed to solve CAPTCHA")
                        return False

            # The main action is visiting the URL. For this type of direct offer link,
            # the click is registered upon a successful visit and redirect.
            # The complex clicking logic is unnecessary and was causing errors.

            # Simulate a longer dwell time to ensure all tracking scripts fire
            dwell = random.randint(15, 45)
            logger.info(f"⏳ Dwelling on CPA offer page for {dwell} seconds to ensure tracking...")
            time.sleep(dwell)

            # Perform some final, non-critical human-like interactions on the page.
            # These are wrapped in try/except blocks to prevent them from crashing the process.
            if config.enable_scroll_simulation and random.random() < 0.7:
                try:
                    device_fingerprint.simulate_realistic_scroll(driver)
                except Exception as e:
                    logger.warning(f"Error during final scroll simulation: {e}")

            if config.enable_mouse_simulation and random.random() < 0.6:
                try:
                    device_fingerprint.simulate_human_mouse_movement(driver)
                except Exception as e:
                    logger.warning(f"Error during final mouse movement: {e}")

            # If we've reached this point without a major crash, consider the visit successful.
            phone_model = extract_phone_model(ua)
            time_str = get_local_time(tz)
            device_id = device_fingerprint.generate_device_id()

            # Log the successful visit
            log_click(phone_model, proxy, ip, city, region, country, time_str, isp, device_id, click_id)

            # Update success rate
            device_success_rates[device]["success"] += 1
            device_success_rates[device]["total"] += 1

            # Print success message
            console.print(f"[{click_num}] ✅ Visit Successful | IP: {ip} | Device: {device} ({phone_model}) | "
                  f"Location: {city}, {region}, {country} | Time: {time_str}")

            return True
        finally:
            # Close the browser
            driver.quit()
    except Exception as e:
        logger.error(f"[{click_num}] ❌ CPA Click Failed: {e}")
        device_success_rates[device]["total"] += 1

        if retry_count > 0:
            logger.info(f"🔄 Retrying... ({retry_count} attempts left)")
            time.sleep(config.retry_delay)
            return process_click(click_num, proxy, device, ua, device_fingerprint, ip_info, session, retry_count - 1)
        return False
    finally:
        # Always delete cache after the click attempt
        try:
            delete_cache()
        except Exception as e:
            logger.warning(f"Error deleting cache: {e}")
# === Countdown Timer ===
def countdown_timer(seconds):
    start_time = time.time()
    end_time = start_time + seconds
    while time.time() < end_time:
        remaining = int(end_time - time.time())
        minutes = remaining // 60
        seconds = remaining % 60
        print(f"\rNext click in: {minutes} minutes {seconds:02d} seconds", end="", flush=True)
        time.sleep(1)
    print()
# === FIXED: Main function with proper proxy handling ===
def main():
    proxies_raw = load_proxies("proxies.txt")[:]
    good_proxies, bad_proxies, proxy_scores = batch_proxy_health_check(proxies_raw)
    proxies_raw = good_proxies[:]

    used_proxies = set()
    click = 0
    successful_clicks = 0
    campaign_clicks = 0
    publisher_clicks = 0

    banner()

    # Get original MAC addresses
    original_macs = get_all_mac_addresses()
    if original_macs:
        logger.info("🔍 Original MAC Addresses:")
        for adapter, mac in original_macs.items():
            logger.info(f"   {adapter}: {mac}")
    else:
        logger.warning("⚠️ Could not get original MAC addresses")

    # Initialize MAC tracking variables
    previous_macs = original_macs.copy()
    current_macs = original_macs.copy()

    # Log enhanced features status
    logger.info("🔧 Enhanced Features Status:")
    logger.info(f"   Mouse Simulation: {'Enabled' if config.enable_mouse_simulation else 'Disabled'}")
    logger.info(f"   Scroll Simulation: {'Enabled' if config.enable_scroll_simulation else 'Disabled'}")
    logger.info(f"   Session Simulation: {'Enabled' if config.enable_session_simulation else 'Disabled'}")
    logger.info(f"   Form Interaction: {'Enabled' if config.enable_form_interaction else 'Disabled'}")
    logger.info(f"   Cloudflare Bypass: {'Enabled' if config.enable_cloudflare_bypass else 'Disabled'}")
    logger.info(f"   TLS Fingerprinting: {'Enabled' if config.enable_tls_fingerprinting else 'Disabled'}")
    logger.info(f"   Header Ordering: {'Enabled' if config.enable_header_ordering else 'Disabled'}")
    logger.info(f"   Environment Consistency: {'Enabled' if config.enable_environment_consistency else 'Disabled'}")
    logger.info(f"   Device Consistency: {'Enabled' if config.enable_device_consistency else 'Disabled'}")
    logger.info(f"   Canvas Fingerprinting: {'Enabled' if config.enable_canvas_fingerprinting else 'Disabled'}")
    logger.info(f"   WebRTC Protection: {'Enabled' if config.enable_webrtc_protection else 'Disabled'}")
    logger.info(f"   Header Randomization: {'Enabled' if config.enable_header_randomization else 'Disabled'}")
    logger.info(f"   Network Fingerprinting: {'Enabled' if config.enable_network_fingerprinting else 'Disabled'}")
    logger.info(f"   Human Error Simulation: {'Enabled' if config.human_error_probability > 0 else 'Disabled'}")
    logger.info(f"   Random Element Interaction: {'Enabled' if config.random_interaction_probability > 0 else 'Disabled'}")

    # Log WebGPU status
    if config.disable_webgpu:
        logger.info("🔒 WebGPU is disabled for enhanced undetectability")

    # Log randomness settings
    logger.info(f"🎲 Randomness Settings:")
    logger.info(f"   Pre-click sites: {config.min_pre_click_sites}-{config.max_pre_click_sites}")
    logger.info(f"   Human error probability: {config.human_error_probability}")
    logger.info(f"   Random interaction probability: {config.random_interaction_probability}")

    # Log new enhanced features
    logger.info(f"🚀 Enhanced Features:")
    logger.info(f"   Rest Periods: {'Enabled' if config.enable_rest_periods else 'Disabled'}")
    logger.info(f"   Browser Extension Simulation: {'Enabled' if config.enable_browser_extension_simulation else 'Disabled'}")
    logger.info(f"   Random Viewport Sizes: {'Enabled' if config.enable_random_viewport_sizes else 'Disabled'}")
    logger.info(f"   Natural Click Patterns: {'Enabled' if config.enable_natural_click_patterns else 'Disabled'}")
    logger.info(f"   Operational Hours: {'Enabled' if config.enable_operational_hours else 'Disabled'}")
    logger.info(f"   Success Monitoring: {'Enabled' if config.enable_success_monitoring else 'Disabled'}")

    # Log new behavioral variance features
    logger.info("🚀 New Enhanced Features:")
    logger.info(f"   Behavioral Variance: Enabled")
    logger.info(f"   Proxy Quality Check: Enabled")
    logger.info(f"   Advanced Device Consistency: Enabled")
    logger.info(f"   Natural Timing Patterns: Enabled")
    logger.info(f"   Session Depth Variation: Enabled")
    logger.info(f"   Click Pattern Randomization: Enabled")

    # Log timing requirements
    logger.info("⏱️ Timing Requirements:")
    logger.info(f"   Warm-up and pre-visit: Maximum 2 minutes (120 seconds)")
    logger.info(f"   Minimum wait between clicks: {config.min_wait_between_clicks} seconds (2 minutes)")
    logger.info(f"   Maximum wait between clicks: {config.max_wait_between_clicks} seconds (10 minutes)")

    with ThreadPoolExecutor(max_workers=config.max_threads) as executor:
        while (config.infinite_loop or click < config.max_clicks) and proxies_raw:
            # Check if within operating hours
            if config.enable_operational_hours and not is_within_operational_hours():
                current_hour = datetime.datetime.now().hour
                start_hour = config.operating_hours_start
                end_hour = config.operating_hours_end

                logger.info(f"⏰ Outside operating hours ({current_hour}:00). Waiting until {start_hour}:00...")

                # Calculate time until next operating hour
                now = datetime.datetime.now()
                if current_hour < start_hour:
                    # Same day
                    next_operating = now.replace(hour=start_hour, minute=0, second=0, microsecond=0)
                else:
                    # Next day
                    next_operating = (now + datetime.timedelta(days=1)).replace(hour=start_hour, minute=0, second=0, microsecond=0)

                wait_seconds = (next_operating - now).total_seconds()
                countdown_timer(int(wait_seconds))
                continue

            if campaign_clicks >= config.max_clicks_per_campaign:
                console.print(f"🚫 Campaign limit reached ({config.max_clicks_per_campaign} clicks)")
                break
            if publisher_clicks >= config.max_clicks_per_publisher:
                console.print(f"🚫 Publisher limit reached ({config.max_clicks_per_publisher} clicks)")
                break

            print_clicks_left()
            device = select_device()
            current_proxy = select_proxy(proxies_raw, proxy_scores, config.rotation_strategy, used_proxies)
            if not current_proxy:
                console.print("🛑 No unused working proxies left. Stopping.")
                break

            if device == "iPhone":
                ua = get_and_remove_ua("iPhone", iphone_agents, "iphoneuseragent.txt")
            elif device == "Samsung":
                ua = get_and_remove_ua("Samsung", samsung_agents, "samsunguseragent.txt")
            elif device == "Windows":
                ua = get_and_remove_ua("Windows", windows_agents, "windowsuseragent.txt")
            elif device == "GooglePixel":
                ua = get_and_remove_ua("GooglePixel", googlepixel_agents, "googlepixeluseragent.txt")
            elif device == "Motorola":
                ua = get_and_remove_ua("Motorola", motorola_agents, "motorolauseragent.txt")
            elif device == "OnePlus":
                ua = get_and_remove_ua("OnePlus", oneplus_agents, "oneplususeragent.txt")
            elif device == "Vivo":
                ua = get_and_remove_ua("Vivo", vivo_agents, "vivouseragent.txt")
            elif device == "Xiaomi":
                ua = get_and_remove_ua("Xiaomi", xiaomi_agents, "xiaomiuseragent.txt")
            elif device == "ZTE":
                ua = get_and_remove_ua("ZTE", zte_agents, "zteuseragent.txt")
            else:
                ua = None

            if not ua:
                console.print(f"⚠️ Skipping {device} click due to no available user agents.")
                continue

            proxy = format_proxy(current_proxy)
            ip_info = get_ip_info(proxy)

            # Create device fingerprint with the user agent
            device_fingerprint = DeviceFingerprint(ua)

            session = requests.Session()
            if config.enable_tls_fingerprinting:
                session.mount('https://', TLSAdapter())
            session.proxies = proxy

            click += 1
            future = executor.submit(
                process_click,
                click,
                current_proxy,
                device,
                ua,
                device_fingerprint,
                ip_info,
                session
            )
            result = future.result()

            if result:
                successful_clicks += 1
                campaign_clicks += 1
                publisher_clicks += 1
                console.print("\n🎉🎉🎉 HEY BOSS YOU HAVE GOT A SUCCESSFUL CLICK 🎉🎉🎉")
                console.print(f"📊 Total successful clicks: {successful_clicks}")

                # Monitor success rates
                monitor_success_rates()

                # Change MAC addresses after successful click
                if platform.system() == "Windows":
                    console.print("\n🔄 Changing MAC addresses after successful click...")

                    # Change Wi-Fi MAC address
                    previous_wifi_mac = current_macs.get(wifi_adapter_name, "Unknown")
                    new_wifi_mac = change_mac_windows(wifi_adapter_name)
                    if new_wifi_mac:
                        current_macs[wifi_adapter_name] = new_wifi_mac
                        console.print(f"\n📡 Wi-Fi MAC Address Changed:")
                        console.print(f"   Adapter: {wifi_adapter_name}")
                        console.print(f"   Original: {original_macs.get(wifi_adapter_name, 'Unknown')}")
                        console.print(f"   Previous: {previous_wifi_mac}")
                        console.print(f"   Current:  {new_wifi_mac}")
                    else:
                        console.print("⚠️ Failed to change Wi-Fi MAC address")

                    # Change device MAC address
                    previous_device_macs = current_macs.copy()
                    new_device_mac = change_device_mac_address()

                    # Get updated MAC addresses after device MAC change
                    updated_macs = get_all_mac_addresses()
                    if updated_macs:
                        current_macs = updated_macs

                        console.print(f"\n🖥️ Device MAC Address Changed:")
                        for adapter in original_macs:
                            if adapter in updated_macs and updated_macs[adapter] != original_macs[adapter]:
                                console.print(f"   Adapter: {adapter}")
                                console.print(f"   Original: {original_macs[adapter]}")
                                console.print(f"   Previous: {previous_device_macs.get(adapter, 'Unknown')}")
                                console.print(f"   Current:  {updated_macs[adapter]}")
                    else:
                        console.print("⚠️ Could not verify device MAC address change")

                # Calculate rest period
                rest_period = calculate_rest_period()
                if rest_period > 0:
                    countdown_timer(rest_period)

                # Add a small delay after MAC changes
                time.sleep(3)

            # FIXED: Only remove the used proxy from the file and in-memory list
            used_proxies.add(current_proxy)
            if config.remove_proxy_after_use and current_proxy in proxies_raw:
                proxies_raw.remove(current_proxy)
                remove_line_from_file("proxies.txt", current_proxy)
                console.print(f"🗑️ Removed used proxy: {current_proxy}")

            # Wait between clicks (2-10 minutes)
            wait = random.randint(config.min_wait_between_clicks, config.max_wait_between_clicks)
            console.print(f"\n⏳ Waiting for next click...")
            countdown_timer(wait)

    console.print("\n==============================")
    console.print("BOT RUN SUMMARY")
    console.print("==============================")
    console.print(f"Total proxies loaded: {len(good_proxies) + len(bad_proxies)}")
    console.print(f"Proxies removed as bad: {len(bad_proxies)}")
    console.print(f"Total successful clicks: {successful_clicks}")
    console.print(f"Campaign clicks: {campaign_clicks}/{config.max_clicks_per_campaign}")
    console.print(f"Publisher clicks: {publisher_clicks}/{config.max_clicks_per_publisher}")
    console.print(f"Good proxies unused: {len(good_proxies) - len(used_proxies)}")

    console.print("\n📋 MAC Address Summary:")
    final_macs = get_all_mac_addresses()
    for adapter in original_macs:
        console.print(f"\n{adapter}:")
        console.print(f"   Original: {original_macs[adapter]}")
        console.print(f"   Final:    {final_macs.get(adapter, 'Unknown')}")
        if original_macs[adapter] != final_macs.get(adapter, 'Unknown'):
            console.print(f"   Status:   ✅ Changed")
        else:
            console.print(f"   Status:   ❌ Unchanged")

    console.print("==============================\n")
# === Run the bot ===
if __name__ == "__main__":
    main()
