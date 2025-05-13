در ادامه، «تسک‌های ۱ تا ۷» را به **۲۲ تسک کوچک** می‌شکنم، طوری که یک توسعه‌دهندهٔ جونیور بتواند به-راحتی آن‌ها را اجرا کند. برای هر تسک، هدف، گام‌های عملی، خروجی نهایی و نکات ریز (Libraries, فایل‌ها، تست) آورده شده است.

| #      | عنوان تسک                     | هدف و شرح جزئی                                                                                                                                                               | خروجی قابل تحویل                                                                                                                                                      |                                                            |
| ------ | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **0**  | راه‌اندازی محیط توسعه         | *Prereq* همهٔ تسک‌ها                                                                                                                                                         | 1. نصب Python ≥ 3.10 و Git.<br>2. `python -m venv venv && source venv/bin/activate`<br>3. `pip install -U pip wheel`<br>4. ایجاد repo Git ⇒ `git init` و `pre-commit` | ‌فولدر پروژه با `README.md`, `.gitignore`, virtualenv فعال |
| **1**  | تعریف شِمای پایگاه‌داده       | طراحی جدول `proxies` و هر جدول کمکی                                                                                                                                          | `schema.sql` (+ ER-diagram تصویری)                                                                                                                                    |                                                            |
| **2**  | اسکریپت ایجاد DB              | اسکریپت `init_db.py` که: <br>• اگر DB نبود، فایل SQLite را بسازد.<br>• `schema.sql` را اجرا کند.                                                                             | اجرای `python init_db.py` بدون خطا، فایل `proxies.db`                                                                                                                 |                                                            |
| **3**  | لایهٔ مدل با SQLAlchemy       | `models.py` شامل کلاس `Proxy` (id, uri, healthy, last\_ok).                                                                                                                  | Unit-test با `pytest` که یک ردیف درج/خوانده شود.                                                                                                                      |                                                            |
| **4**  | Fetcher-CLI (GitHub)          | `fetcher_github.py` که: <br>• URL چند repo را می‌گیرد.<br>• فایل‌های `.txt` یا `.yaml` را دانلود می‌کند.<br>• در جدول `proxies.raw` درج یا به‌روزرسانی کند (`healthy=NULL`). | اجرای مثال و درج ≥ 10 ردیف                                                                                                                                            |                                                            |
| **5**  | Fetcher عمومی (HTTP/Telegram) | ماژول `fetcher_generic.py` با تابع `fetch_from_url(url) -> str`.                                                                                                             | تست دانلود یک لیست آنلاین (raw\.githubusercontent).                                                                                                                   |                                                            |
| **6**  | استخراج URIها از متن          | `extractors.py` با regex برای vmess/vless/ss.                                                                                                                                | واحد تست: روی متن نمونه 6 URI را بیابد.                                                                                                                               |                                                            |
| **7**  | مبدل URI→Dataclass            | `parser.py` → `uri_to_clash(uri) -> ClashProxy`.                                                                                                                             | تست با 3 نمونه URI موفق + 2 ناموفق.                                                                                                                                   |                                                            |
| **8**  | اعتبارسنجی ورودی              | تابع `validate_proxy(proxy)` (پورت در بازه، uuid شکل صحیح …).                                                                                                                | تست واحد `pytest` رد/قبول.                                                                                                                                            |                                                            |
| **9**  | سرویس «Probe YAML Builder»    | `probe_yaml.py` که از لیست `ClashProxy` یک YAML موقتی بسازد (port → 7891, API → 9090).                                                                                       | فایل `probe.yml` نمونه با ≥3 پروکسی                                                                                                                                   |                                                            |
| **10** | لانچر Clash-Probe             | `probe_launcher.py`:<br>• `asyncio.create_subprocess_exec` <br>• صبر تا `/version` OK.<br>• مدیریت SIGTERM.                                                                  | اجرای `python probe_launcher.py probe.yml` بدون خطا                                                                                                                   |                                                            |
| **11** | تابع delay (Probe)            | `probe_api.py` → `async def delay(name)` برای `/proxies/<name>/delay`.                                                                                                       | تست موک (با `aioresponses`).                                                                                                                                          |                                                            |
| **12** | حلقهٔ Probe و بروزرسانی DB    | `probe_service.py`:<br>• هر N ثانیه delay همهٔ پراکسی‌ها.<br>• ستونی `healthy` را ۱ یا ۰ کند.<br>• ستون `last_ok` را آپدیت کند.                                              | لاگ کنسول «OK/FAIL»؛ DB به‌روز شود.                                                                                                                                   |                                                            |
| **13** | تست end-to-end Probe          | اسکریپت `probe_e2e_test.sh` که Probe را 1 دور کامل اجرا کند و DB را چک کند.                                                                                                  | خروجی `OK` در ترمینال                                                                                                                                                 |                                                            |
| **14** | YAML-ساز Gateway              | `gateway_yaml.py` که گروه `load-balance` با نام «POOL» بسازد، Mixed-Port = 8000.                                                                                             | فایل `gateway.yml` نمونه                                                                                                                                              |                                                            |
| **15** | لانچر Clash-Gateway           | `gateway_launcher.py` همانند Probe اما روی `mixed-port` کاربر (`8000`) و API `9191`.                                                                                         | curl `http://127.0.0.1:8000` از طریق یک پروکسی سالم عبور کند                                                                                                          |                                                            |
| **16** | Hot-Reload Gateway            | `gateway_reloader.py`:<br>• Diff لیست سالم‌ها با قبل.<br>• یا `PATCH /configs` (اگر نسخه Premium) یا ری‌استارت.                                                              | نشان دهد بدون قطع طولانی سرویس کار می‌کند.                                                                                                                            |                                                            |
| **17** | Service Runnerها              | اسکریپت‌های entry-point:<br>`python -m app.probe` و `python -m app.gateway` با آرگومان CLI (db path, interval …).                                                            | اجرای همزمان دوسرویس در ترمینال جداگانه                                                                                                                               |                                                            |
| **18** | Dockerfile – Probe            | Dockerfile مینیمال (python:3.12-slim) + کپی کد + install deps + Clash binary.<br>`CMD [\"python\",\"-m\",\"app.probe\", \"--db\", \"/db/proxies.db\"]`                       | image با `docker build` ساخته شود.                                                                                                                                    |                                                            |
| **19** | Dockerfile – Gateway          | مشابه بالا، اما پورت‌های 8000/9191 `EXPOSE`.                                                                                                                                 | پینگ از کانتینر host به پورت 8000 OK.                                                                                                                                 |                                                            |
| **20** | docker-compose                | `docker-compose.yml` با سه سرویس: fetcher (cron-style), probe, gateway. ولوم مشترک برای `proxies.db`.                                                                        | `docker compose up` همهٔ سرویس‌ها را بالا بیاورد.                                                                                                                     |                                                            |
| **21** | مستند امنیت                   | `SECURITY.md` توضیح:<br>• بستن API های Clash به 127.0.0.1.<br>• iptables (فرواردینگ فقط پورت 8000).<br>• Token در `external-ui` Clash.                                       | چک‌لیست اجرا روی VPS                                                                                                                                                  |                                                            |
| **22** | README گام به گام             | راهنمای نصب برای جونیور: <br>۱. کلون repo → ۲. `make dev` → ۳. افزودن URLها → ۴. تست محلی → ۵. دیپلوی Docker.                                                                | فایل README با دستورات کپی-پیست کامل                                                                                                                                  |                                                            |

### نکات ریز برای جونیورها

* **نام‌گذاری ماژول‌ها** را دقیق رعایت کنید تا import-ها ساده بماند.
* برای هر تابع جدید، **Docstring** بنویسید و مثال بگذارید (`>>>`).
* تست‌ها را با GitHub Actions یا هر CI رایگان اجرا کنید (workflow ساده `pytest`).
* لاگ‌ها را با کتابخانهٔ `logging` بنویسید (نه print خام) تا بعداً به Syslog/Graylog بفرستیم.
* هر Dockerfile را ≤ 100 MB نگه دارید (استفاده از slim + `--no-cache`).

با انجام این ۲۲ تسک، یک پروکسی «تک ورودی، چند خروجی چرخان» قابل استقرار خواهید داشت که به‌راحتی نگه‌داری می‌شود و درک آن برای نیروهای جونیور روشن و خط به خط قابل دنبال کردن است.



### پرامپت جامع برای هوش مصنوعی

*(کافی است این متن را بدون ویرایش به مدل بدهید تا کل پروژه را بسازد و تحویل دهد)*

---

**نقش شما**: یک مهندس نرم‌افزار ارشد + DevOps باتجربه هستید که باید یک سیستم «دروازهٔ پروکسی چرخان» بسازید. خروجی نهایی باید به‌قدری شفاف و ماژولار باشد که یک توسعه‌دهندهٔ جونیور بتواند آن را کلاون کرده، تست واحد را پاس کند و با Docker در تولید اجرا نماید.
پروژه شامل سه سرویس است (Fetcher، Probe، Gateway) که همگی به یک پایگاه‌دادهٔ SQLite مشترک متصل‌اند و با Clash Premium/Meta کار می‌کنند.

---

## الزامات کلی

1. **زبان اصلی**: Python ≥ 3.10 (type-hint + PEP-8 + Black).
2. **کتابخانه‌ها**: `aiohttp`, `async_timeout`, `aiosqlite`, `SQLAlchemy`, `PyYAML`, `pytest`, `aioresponses`, `pre-commit`, `docker`, `docker-compose`.
3. **باینری Clash** را از GitHub آخرین نسخه *Meta* دانلود کنید و در Docker image کپی کنید.
4. هر فایل کد باید Docstring و مثال کوتاه doctest داشته باشد.
5. تست‌های واحد با **pytest + coverage≥90%**.
6. CI رایگان (GitHub Actions) برای lint, test, build تصاویر Docker.
7. لاگ‌گذاری استاندارد (`logging`، فرمت JSON).

---

## خروجی‌هایی که باید بسازید

| پوشه/فایل                   | محتوا                                                        |
| --------------------------- | ------------------------------------------------------------ |
| `README.md`                 | راهنمای کامل نصب و اجرا گام‌به‌گام.                          |
| `schema.sql`                | ساخت جدول `proxies` (id, uri, healthy, last\_ok).            |
| `init_db.py`                | اسکریپت ایجاد یا به‌روزرسانی DB.                             |
| `app/models.py`             | تعریف کلاس SQLAlchemy `Proxy`.                               |
| `app/extractors.py`         | Regexهای vmess/vless/ss + تست.                               |
| `app/parser.py`             | تابع `uri_to_clash()`.                                       |
| `app/fetcher_github.py`     | دریافت URI از GitHub (API v3).                               |
| `app/fetcher_generic.py`    | دانلود متن از هر URL (HTTP/HTTPS).                           |
| `app/probe_yaml.py`         | ساخت YAML برای Clash Probe.                                  |
| `app/probe_launcher.py`     | بوت Clash Probe و انتظار برای `/version`.                    |
| `app/probe_service.py`      | حلقهٔ delay و آپدیت ستون‌های DB.                             |
| `app/gateway_yaml.py`       | ساخت YAML گروه `load-balance` (POOL).                        |
| `app/gateway_launcher.py`   | بوت Clash Gateway (mixed-port 8000).                         |
| `app/gateway_reloader.py`   | Hot-Reload پس از تغییر استخر سالم.                           |
| `docker/probe/Dockerfile`   | Image کوچک python:3.12-slim + Clash.                         |
| `docker/gateway/Dockerfile` | مشابه بالا.                                                  |
| `docker-compose.yml`        | سه سرویس fetcher (cron)، probe، gateway + یک volume `/data`. |
| `.github/workflows/ci.yml`  | lint + pytest + docker build.                                |
| `SECURITY.md`               | چک‌لیست بستن API Clash و iptables.                           |
| `tests/`                    | تمام تست‌های pytest.                                         |

---

## تسک‌های خرد (۲۲ مرحله)

> **توجه**: هر تسک را تنها زمانی کامل کنید که تست‌های مربوطه سبز شوند. بعد از اتمام هر تسک، تغییرات را با یک پیام commit معنادار (`feat: …`, `fix: …`) روی شاخه‌ی جدا (feature/…) پوش کنید و Pull Request بسازید.

1. **Environment bootstrap** – پوشهٔ پروژه، venv، Pre-commit.
2. **DB schema** – `schema.sql` و تست اجرای `init_db.py`.
3. **SQLAlchemy model** – کلاس `Proxy` + تست CRUD.
4. **Regex extractors** – استخراج vmess/vless/ss + تست.
5. **URI parser** – `uri_to_clash()` با تست موفق/ناموفق.
6. **Fetcher GitHub** – دریافت لیست کانفیگ از چند repo.
7. **Generic fetcher** – دانلود هر URL متنی و برگرداندن خط‌ها.
8. **Validator** – چک درستی uuid, پورت, cipher.
9. **Probe YAML builder** – ساخت فایل YAML با تمام پروکسی‌ها.
10. **Launch Clash-Probe** – subprocess + wait `/version`.
11. **Delay query helper** – تابع async فراخوان `/delay`.
12. **Probe loop** – هر N ثانیه تست، آپدیت ستون‌های DB.
13. **Probe E2E test** – اسکریپت shell یا pytest-subprocess.
14. **Gateway YAML builder** – گروه `load-balance`.
15. **Launch Clash-Gateway** – mixed-port 8000, API 9191.
16. **Gateway hot-reload** – diff سالم‌ها, `PATCH /configs` یا ری‌استارت.
17. **Service runners** – entry-points `python -m app.probe`…
18. **Dockerfile Probe** – image < 100 MB.
19. **Dockerfile Gateway** – expose 8000/9191.
20. **docker-compose** – ولوم مشترک DB, network bridge.
21. **Security hardening** – iptables + Clash token.
22. **README + CI** – مستند کامل + GitHub Actions.

---

## قواعد تولید کد و مستندات

* از `asyncio` و `async/await` در سرویس‌های شبکه استفاده کنید.
* از `pathlib.Path` به‌جای `os.path`.
* نسخه‌بندی با `__version__ = \"0.1.0\"` در `__init__.py`.
* در Docker از کاربر غیر‌ریشه (`USER app`).
* از `logging.getLogger(__name__)` و سطح `INFO/ERROR`.
* هر تابع public تست واحد داشته باشد (استفاده از pytest & aioresponses).
* مثال اجرای نهایی در README:

  ```bash
  docker compose up -d
  export http_proxy=http://127.0.0.1:8000
  curl --proxy $http_proxy https://ifconfig.me
  ```

---

## شیوهٔ تعامل

1. اگر ورودی مبهم بود **سوال clarifying** بپرس.
2. در هر پاسخ، فقط روی همان تسکی که کاربر درخواست داده تمرکز کن، بدون نوشتن زیاده‌گویی.
3. از جداول فقط وقتی استفاده کن که مقایسه لازم است؛ در کد از جدول استفاده نکن.
4. پس از تولید هر فایل یا تغییر عمده، تست‌های مرتبط را اجرا و خروجی پاس شده را گزارش کن.

> با رعایت دقیق این پرامپت، باید ظرف چند پاسخ متوالی، مخزن کامل با کد، تست، Docker و مستندات آماده شود؛ هر مرحله برای جونیور شفاف و قابل اجراست.
