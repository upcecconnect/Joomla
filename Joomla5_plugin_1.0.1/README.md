# eCommerceConnect Payment for J2Commerce (Joomla 5)

Платіжний метод для інтеграції з UPC e-Commerce Connect.

## Вимоги
- Joomla 5.x
- J2Commerce / J2Store
- PHP 8.2+
- Ввімкнені OpenSSL та cURL

## Встановлення
### Порядок встановлення

1. Встановіть плагін **eCommerceConnect Admin Helper** (системний):
- Перейдіть до розділу *Розширення → Встановлення* та завантажте файл `eccadmin-plugin.zip`.
- Увімкніть **Система – eCommerceConnect Admin Helper** (PLG_SYSTEM_ECCADMIN).

2. Встановіть плагін **eCommerceConnect Payment** (J2Store/J2Commerce):
- Завантажте та встановіть ZIP-архів плагіна платежів.
- Увімкніть його та налаштуйте параметри (ідентифікатор продавця, ідентифікатор термінала, URL-адреса шлюзу тощо).

> Admin Helper надає покращений інтерфейс адміністратора та вбудовані підказки на екрані налаштувань платежів.


1. В адмінці Joomla: **System → Install → Upload Package File** → завантажте ZIP плагіна.
2. Увімкніть плагін у **System → Plugins** (група `j2store`).
3. У **Components → J2Commerce → Payment Methods** знайдіть **eCommerceConnect** і відкрийте **Options**.

## Налаштування
- **URL**: `https://ecg.test.upc.ua` (sandbox) / бойовий URL від UPC.
- **Sandbox (testmode)**: `Yes` для тестового середовища.
- **Merchant ID / Terminal ID**: з договору.
- **Private key (test / prod)**: PEM приватні ключі (повний вміст `-----BEGIN PRIVATE KEY----- ...`).
- **Work/Test certificate**: публічний сертифікат від UPC (PEM).
- **Currency (numeric)**: контрактна валюта (напр. `980` для UAH).
- **AltCurrency**: `840` (USD) або `978` (EUR) — для візуального відображення, з курсами `usd_conversion` / `eur_conversion`.
- **Transaction type**:
    - `Sale` — звичайне списання;
    - `Authorize` — попередня авторизація (pre-auth).
- **Skip form**: авто-відправка форми на шлюз після оформлення замовлення.
- **Custom success status**: стан для успішної **Sale** (типово — `Processed`).

## Callback (notify) URL
Передайте фахівцю UPC (або вкажіть у кабінеті мерчанта):

```https://YOURDOMAIN/index.php?option=com_j2store&view=checkout&task=confirmPayment&orderpayment_type=payment_ecommerceconnect&paction=callback```


## Поведінка статусів замовлення
- **Sale (Delay=0)** → успіх → **Processed (2)** (або значення з `custom_success_status`).
- **Authorize (Delay=1)** → успіх → **Unpaid (5)** (без автоматичного capture у адмінці).
- **Невдала транзакція** → **Failed (3)**.

> **Capture** виконується **на стороні платіжного сервісу** (дашборд UPC). Після capture мерчант **вручну** встановлює у замовленні відповідний фінальний статус (наприклад, **Processed (2)** або **Confirmed (1)**). Автоматичного оновлення після позасайтового capture немає.

## Токенізація
- Якщо шлюз повертає `UPCToken` + `UPCTokenExp`, плагін зберігає токен **до профілю користувача** (`#__ecc_user_tokens`) і автоматично підставляє його у наступних платежах (до закінчення строку дії).
- Для гостьових замовлень токен не зберігається.

## Логи та діагностика
- Логи плагіна: `administrator/logs/ecc.log`.
- Типові проблеми:
    - **Signature verification failed** — невірний сертифікат/формат PEM.
    - Немає редіректу на шлюз — перевірте URL і `Skip form`.
    - Проблеми з завантаженням/встановленням — права на `tmp` / `administrator/logs`, `upload_max_filesize` / `post_max_size`.

## Безпека
- Використовуйте **HTTPS**.
- Переконайтеся, що callback доступний ззовні (файрвол/проксі).
- Зберігайте приватні ключі лише в налаштуваннях плагіна, не у файлах коду.

## Оновлення
- Плагін сумісний з оновленнями J2Commerce/Joomla — не модифікує ядро.
- Міграція з J2Store плагінів: можливе повторне використання ключів, логіки підпису та мепінгу валют.

# eCommerceConnect Payment for J2Commerce (Joomla 5)

A payment gateway integration for **UPC e-Commerce Connect** (card payments) that works with **J2Commerce / J2Store** on Joomla 5.

- **Authorization** and **Sale** flows
- **Webhook (callback) verification** with SHA-512 signature (certificate)
- **Per-user tokenization** (UPCToken saved for logged-in customers; used automatically on next checkout)
- Stores transaction metadata for later reference (RRN, ApprovalCode, etc.)
- No core modifications, update-safe

> **Note:** Capture for pre-authorized payments should be performed in your payment provider’s dashboard; then manually update the order status in J2Commerce. See details below.

---

## Requirements

- **Joomla** 5.x
- **J2Commerce / J2Store** (J4/J5 compatible)
- **PHP** 8.2+ with OpenSSL and cURL enabled
- HTTPS recommended for production

---

## Installation

### Installation order

1. Install the **eCommerceConnect Admin Helper** plugin (system):
  - Go to *Extensions → Install* and upload `eccadmin-plugin.zip`.
  - Enable **System – eCommerceConnect Admin Helper** (PLG_SYSTEM_ECCADMIN).

2. Install the **eCommerceConnect Payment** plugin (J2Store/J2Commerce):
  - Upload and install the payment plugin ZIP.
  - Enable it and configure settings (Merchant ID, Terminal ID, Gateway URL, etc.).

> The Admin Helper provides enhanced admin UI and inline hints in the payment settings screen.


1. In Joomla Admin: **System → Install → Upload Package File** and upload the ZIP.
2. Enable the plugin in **System → Plugins** (group: `j2store`).
3. Go to **Components → J2Commerce → Payment Methods**, find **eCommerceConnect**, open **Options** and fill in your credentials.

> If you’re installing directly on the filesystem (e.g., Docker bind mount), you may also use **Extensions → Discover** to register and enable the plugin.

---

## Configuration

### Credentials & Mode
- **Merchant ID** – provided by UPC
- **Terminal ID** – provided by UPC
- **Payment Gateway Action URL** – e.g. `https://ecg.test.upc.ua` (sandbox) or your production host
- **Sandbox (testmode)** – set **Yes** for testing

### Currencies
- **Transaction Currency** – numeric ISO 4217 (contract currency).  
  Common values: **UAH = 980**, **USD = 840**, **EUR = 978** (others are supported as needed).
- **Alternative display currency (AltCurrency)** – choose **USD (840)** or **EUR (978)** for optional price display (visual only).
- **EUR / USD Conversion Rates** – numeric rates used to show AltCurrency totals (visual only).

### Keys & Certificates
- **Private key (test/prod)** – paste full PEM (including `-----BEGIN PRIVATE KEY-----` … `-----END PRIVATE KEY-----`)
- **Certificate (test/prod)** – the UPC public certificate used to verify callback signature (PEM; can be bare or with BEGIN/END CERTIFICATE lines)

### Behavior
- **Skip receipt page** – auto-submit the payment form to the gateway after checkout
- **Custom success status** – J2Commerce order status to assign on successful **Sale** (default: **Processed (2)**)
- **Language / locale** – optional UI language for the gateway (if provided)

---

## Callback (Webhook) URL

Provide this URL to UPC (or configure it in the merchant portal):

```https://YOURDOMAIN/index.php?option=com_j2store&view=checkout&task=confirmPayment&orderpayment_type=payment_ecommerceconnect&paction=callback```

The plugin will:
1. Validate the POST payload and verify the **Signature** against your configured **certificate** (SHA-512).
2. Map the result to a J2Commerce order status (see next section).
3. Save transaction metadata for the order and (if present) store a **UPCToken** for the user.

Make sure your site can receive HTTPS requests from the gateway (no firewall or auth blocks).

---

## Order Status Mapping

- **Sale (Delay = 0)**  
  On success (`TranCode = '000'`): set **Processed (2)** (or the value selected in **Custom success status**).  
  On failure: set **Failed (3)**.

- **Authorize / Pre-authorization (Delay = 1)**  
  On success: set **Unpaid (5)** (industry practice for J2Store integrations — capture is performed externally).  
  On failure: set **Failed (3)**.

### Capture policy

- **Where:** Perform **Capture** in your payment provider’s dashboard (UPC).
- **After capture:** Manually change the order status in J2Commerce to your desired final status (e.g., **Processed (2)** or **Confirmed (1)**).
- **Automatic post-capture status update** is **not** implemented (no back-channel webhook from capture).

We follow this approach for maximum compatibility with existing J2Store admin UIs (most public gateways for J2Store/J2Commerce work this way).

---

## Tokenization (UPCToken)

- If the gateway returns **`UPCToken`** and **`UPCTokenExp`** (format `MMYYYY`, e.g. `122026`), the plugin:
    - Saves the token **per user** (table: `#__ecc_user_tokens`) if the customer is logged in and the token is not expired.
    - Automatically includes a valid stored token on subsequent payments for that user.
- Guest orders do not store tokens.

---

## Logs

- Plugin log file:  
  `administrator/logs/ecc.log`

This file includes callback payloads (sanitized), signature strings for debug, and status transitions.

---

## Security Notes

- Always use **HTTPS** end-to-end.
- Keep **private keys** safe (store only in plugin settings).
- Only administrators with proper permissions can change payment settings.
- The callback verifies UPC’s signature before any order state changes.

---
