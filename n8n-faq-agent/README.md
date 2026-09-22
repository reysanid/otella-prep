
# FAQ Semantic Search — n8n AI Agent

## چی هست
یک AI agent برای پاسخ‌گویی معنایی (semantic search) به سوالات متداول یک دوره‌ی Design Thinking. کاربر سوالش رو از طریق یک فرم می‌فرسته و agent با جست‌وجو در embedding های ذخیره‌شده، دقیق‌ترین جواب رو برمی‌گردونه.

## Pipeline

**Ingestion (بارگذاری داده):**
Read/Write Files from Disk (faq.csv)
→ Extract from File (CSV)
→ Embeddings Ollama
→ Simple Vector Store (Insert Documents)

**Retrieval (پاسخ‌گویی):**
On new n8n form event (سوال کاربر)
→ Embeddings Ollama
→ Simple Vector Store (Get Many, limit 4)
→ Edit Fields (استخراج پاسخ نهایی)

## ابزارها
- **n8n** — روی Docker (Windows)
- **Ollama** (مدل `nomic-embed-text`) — برای embedding، به‌جای OpenAI (به‌خاطر محدودیت‌های دسترسی بین‌المللی)
- **In-memory Vector Store** — داده با هر ریستارت container پاک می‌شه، پس ingestion باید هر بار دوباره اجرا بشه

## چالش‌ها و رفع باگ
در مسیر ساخت این agent با چهار مشکل جدی مواجه شدم و همه رو دیباگ کردم:

1. **اتصال قطع در ingestion** — گره‌ی `Extract from File` به `Simple Vector Store` وصل نبود؛ روی canvas دستی وصلش کردم.
2. **Memory Key ناهماهنگ** — گره‌ی ingestion از `faq_store` استفاده می‌کرد ولی گره‌ی retrieval از یک key متفاوت؛ هماهنگشون کردم.
3. **تایپوی متادیتا** — عبارت `{{ json.answer$ }}` باید می‌شد `{{ $json.answer }}` — بعد از اصلاح، متادیتای answer درست ست شد.
4. **مسیر غلط در Edit Fields** — به‌جای `{{ $json.output[0].metadata.answer }}` باید `{{ $json.document.metadata.answer }}` می‌بود، چون ساختار خروجی retrieval متفاوت بود.

## وضعیت
Workflow به‌صورت end-to-end تست و تایید شده — ارسال یک سوال از طریق فرم، جواب درست رو برمی‌گردونه.
