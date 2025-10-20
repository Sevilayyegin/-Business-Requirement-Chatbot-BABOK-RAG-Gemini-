# 🔹 BABOK + RAG + Gemini Tabanlı Business Requirement Chatbot

**(Yapay Zekâ Destekli İş Analizi Asistanı)**

---

## 🌟 Projenin Amacı

Bu proje, iş analizi sürecinde **gereksinimlerin tanımlanması, sınıflandırılması ve önceliklendirilmesini otomatikleştirmek** amacıyla geliştirilmiştir.
Klasik gereksinim mühendisliği süreçlerinde manuel olarak yapılan analizler, bu chatbot sayesinde **saniyeler içinde otomatik ve standartlara uygun şekilde** üretilebilmektedir.

### Sistem Özeti

🧠 **Veri temelli (RAG destekli)** — Bilgi tabanı üzerinde benzer gereksinimleri bulur.
📘 **Uluslararası standartlara uygun (BABOK)** — BABOK alanlarına göre yapılandırılmış çıktılar üretir.
⚙️ **Ölçeklenebilir (ChromaDB + Gemini)** — Büyük ölçekli verilerde semantik arama ve üretim yapabilir.
💬 **Kullanıcı dostu (Gradio arayüzü)** — İş analistleri için kolay, etkileşimli bir arayüz sağlar.

Kısaca sistem, bir iş analistinin yaptığı şu görevleri **otomatikleştirir**:

* Gereksinimin **Functional** (işlevsel) veya **Non-Functional** (işlevsel olmayan) olup olmadığını tahmin eder.
* **BABOK** yapısına uygun biçimde gereksinim dokümanı üretir.
* **RICE** ve **WSJF** metrikleriyle önceliklendirme skorları hesaplar.
* **Gradio tabanlı arayüz** üzerinden kullanıcıya okunabilir raporlar sunar.

---

## 🧩 Sistem Mimarisi

![architecture](Business%20Requirement%20Chatbot.png)

> 💡 **Veri Akışı:**
> Kullanıcı → Embedding (Gemini) → ChromaDB (RAG) → LLM (Gemini) → BABOK Gereksinim Formatı → RICE & WSJF → Gradio Arayüzü

| Katman                           | Teknoloji                       | Açıklama                                                        |
| :------------------------------- | :------------------------------ | :-------------------------------------------------------------- |
| 🤖 **LLM**                       | Google Gemini (1.5 Flash / Pro) | Gereksinimleri analiz eder ve BABOK uyumlu metin üretir.        |
| 📊 **Veri Tabanı**               | ChromaDB                        | Gereksinim embedding’lerini saklar ve semantik benzerlik arar.  |
| 🧠 **Embedding Modeli**          | `text-embedding-004`            | Metinleri sayısal vektör temsiline dönüştürür.                  |
| 📚 **Veri Seti**                 | PURE Annotate Dataset (Kaggle)  | Gereksinim cümleleri ve F/NF etiketleri içerir.                 |
| ⚖️ **Önceliklendirme Modülleri** | RICE ve WSJF                    | Gereksinimlerin iş değeri ve riskine göre sıralanmasını sağlar. |
| 💬 **Web Arayüzü**               | Gradio                          | Kullanıcıdan doğal dilde giriş alır, çıktıları görselleştirir.  |

---

## ⚙️ Kurulum ve Ortam Hazırlığı

### 1️⃣ Gerekli Kütüphanelerin Yüklenmesi

```bash
pip install -q chromadb sentence-transformers google-generativeai python-dotenv gradio pandas
pip install -q transformers==4.41.2 huggingface_hub==0.22.2
```

### 2️⃣ API Anahtarı Tanımlama

`.env` dosyanı oluştur ve içine Google Gemini API anahtarını ekle:

```bash
GOOGLE_API_KEY="your_api_key_here"
```

### 3️⃣ Projeyi Başlatma

```bash
python app.py
```

🔊 Tarayıcıda Gradio arayüzü otomatik olarak açılacaktır.

---

## 📊 Veri Seti — PURE Annotate Dataset

Proje, **Kaggle üzerindeki Public Requirements PURE Dataset**’i kullanmaktadır.
Her satır bir gereksinim cümlesini temsil eder:

| Tür            | Örnek                                     |
| -------------- | ----------------------------------------- |
| Functional     | “System shall export reports in PDF.”     |
| Non-Functional | “System shall encrypt user data at rest.” |

### Etiketleme Mantığı

* `NFR_boolean = 0` → İşlevsel (Functional Requirement)
* `NFR_boolean = 1` → İşlevsel Olmayan (Non-Functional Requirement)

Eğer veri setinde bu alan bulunmuyorsa, sistem otomatik olarak **heuristik analiz** yaparak güvenlik, performans, uyumluluk gibi kelimelere göre F/NF etiketlemesi yapar.

💡 Örnek Heuristik Kurallar:

* “güvenlik”, “şifreleme”, “performans” → **Non-Functional**
* “kaydet”, “oluştur”, “listele” → **Functional**

---

## 🔠 Embedding ve Bilgi Tabanı (ChromaDB)

Her gereksinim cümlesi **Gemini’nin embedding modeli** ile sayısal bir vektöre dönüştürülür.
Bu vektörler **ChromaDB** üzerinde saklanarak, semantik olarak benzer gereksinimlerin bulunmasını sağlar.

```python
emb = genai.embed_content(model="models/text-embedding-004", content=text)
collection.add(ids, documents, embeddings)
```

💡 Bu yapı sayesinde sistem, kullanıcıdan gelen sorguyla benzer gereksinimleri “anlamsal yakınlık” ile bulabilir.

---

## 🧠 BABOK Uyumlu Gereksinim Üretimi

Model, **BABOK (Business Analysis Body of Knowledge)** standardına uygun olarak aşağıdaki alanları otomatik doldurur:

| Alan                       | Açıklama                                        |
| -------------------------- | ----------------------------------------------- |
| **Gereksinim Türü**        | Business / Stakeholder / Solution / Transition  |
| **Doğa (F/NF)**            | Gereksinimin işlevsel olup olmadığını belirtir. |
| **Rationale (Gerekçe)**    | Gereksinimin neden gerekli olduğunu açıklar.    |
| **Business Value**         | İşe katkı veya getiriyi belirtir.               |
| **Acceptance Criteria**    | Test edilebilir kabul kriterleri.               |
| **MoSCoW Önceliği**        | Must / Should / Could / Won’t                   |
| **Impact / Effort / Risk** | 1–5 ölçeğinde sayısal değerlendirme.            |
| **Kano Kategorisi**        | Temel / Performans / Heyecan                    |
| **Cost of Delay**          | Gecikmenin mali etkisi.                         |

> Tüm çıktılar **Türkçe**, yapısal ve ölçülebilir biçimde üretilir.

---

## ⚖️ Gereksinim Önceliklendirme Modülü

Proje, gereksinimleri iki metrikle değerlendirir:

| Metrik   | Formül                                                          | Açıklama                                                         |
| -------- | --------------------------------------------------------------- | ---------------------------------------------------------------- |
| **RICE** | (Reach × Impact × Confidence) / Effort                          | Gereksinimin kullanıcı etkisi ve erişim gücüne göre puanlanması. |
| **WSJF** | (Business Value + Time Criticality + Risk Reduction) / Job Size | İş değeri, risk azaltımı ve zaman baskısını dikkate alır.        |

🔍 Anahtar sinyaller (otomatik tespit edilir):

* “güvenlik”, “şifreleme”, “kvkk” → yüksek risk
* “performans”, “ölçeklenebilirlik” → yüksek etki
* “yedekleme”, “erişilebilirlik” → zaman kritiği yüksek

📊 Çıktı örneği:

```
🧮 Önerilen Önceliklendirme
- RICE: 46.6
- WSJF: 4.2
- Risk: 5
- Impact: 4
🏷️ Gereksinim Doğası: Non-Functional
```

---

## 🔍 RAG (Retrieval-Augmented Generation) Akışı

1. **Retrieval:** Kullanıcı sorgusu embedding’e dönüştürülür, ChromaDB’den en yakın gereksinimler getirilir.
2. **Augmentation:** Bu örnekler modelin bağlamına eklenir.
3. **Generation:** Gemini modeli, BABOK yapısında gereksinim çıktısı üretir.

💡 Bu yapı sayesinde sistem, hem **veriye dayalı** (retrieval) hem **yaratıcı** (generation) bir analiz sunar.

---

## 💬 Gradio Arayüzü

Gradio, kullanıcıdan proje açıklamasını alır ve modeli tetikler.
Sonuç olarak kullanıcı **BABOK uyumlu gereksinim önerilerini** doğrudan tarayıcı üzerinden görür.

```bash
python app.py
```

**Örnek Arayüz:**

```
💬 Proje Açıklaması:
"Bankacılık uygulamasında müşteri verisi güvenliği ve işlem performansı."

📘 Çıktı:
- Gereksinim Türü: Solution
- Doğa: Non-Functional
- Gereksinim: Sistem müşteri verilerini AES-256 ile şifrelemelidir.
- Rationale: KVKK ve PCI-DSS uyumluluğu sağlamak için.
```

---

## 🧪 Test Çalışması

```python
test_query = "Günlük 5 milyon API çağrısını %99.9 başarı oranı ile işleyebilmelidir."
print(rag_response_babok(test_query))
```

📈 Model çıktısı:

* Gereksinim Doğası: Non-Functional
* RICE Skoru: 44.8
* WSJF Skoru: 5.1
* BABOK uyumlu gereksinim: “Sistem saniyede 60.000 istek işleyebilmeli, ortalama gecikme 200 ms altında olmalıdır.”

---

## 🧩 Proje Yapısı

```
📦 business-requirement-chatbot/
 ┣ 📂 data/
 ┃ ┗ Pure_Annotate_Dataset.csv
 ┣ 📂 src/
 ┃ ┣ embeddings.py
 ┃ ┣ generator.py
 ┃ ┣ prioritization.py
 ┃ ┗ ui.py
 ┣ app.py
 ┣ .env.example
 ┗ README.md
```

## 📘 Lisans ve Katkılar

Bu proje MIT Lisansı altında yayınlanmıştır.


## 🧠 Sonuç ve Değerlendirme

Bu proje, BABOK temelli iş analizi yöntemleriyle yapay zekâ teknolojilerini birleştirerek gereksinim mühendisliğini yarı otomatik hale getirmeyi başarmıştır.

RAG (Retrieval-Augmented Generation) yapısı sayesinde:

Gereksinim dokümantasyonu hızlanmış,

Tutarlılık ve izlenebilirlik sağlanmış,

LLM çıktıları bağlama duyarlı hale getirilmiştir.

Proje, gelecekte kurumsal bilgi yönetimi, proje planlama ve otomatik gereksinim izleme sistemlerinde ölçeklenebilir bir temel olarak kullanılabilir.

```
