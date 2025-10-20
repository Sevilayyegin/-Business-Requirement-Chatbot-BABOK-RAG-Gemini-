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

🚀 TEST ÇALIŞTIRILIYOR...
📝 Sorgu: Günlük 5 milyon API çağrısını %99.9 başarı ile işleyebilmelidir.

📘 Örnek Çıktı:

Kullanıcının proje açıklamasına dayanarak, BABOK standartlarına uygun gereksinim önerileri aşağıda listelenmiştir:

────────────────────────────
**Gereksinim Önerisi 1**

**Gereksinim Türü:** Solution  
**Gereksinim Doğası (F/NF):** Non-Functional  
**Gereksinim:** Sistem, günlük ortalama 5 milyon API çağrısını %99.9 başarı oranıyla işleyebilmelidir.  
**Gerekçe (Rationale):** Belirtilen başarı oranı ve çağrı sayısı, sistemin kabul edilebilir performans ve güvenilirlik düzeyini tanımlar.  
**İş Değeri (Business Value):** Kullanıcı memnuniyetini ve sistem güvenilirliğini artırır, iş sürekliliğini sağlar.  
**Paydaşlar (Stakeholders):** Kullanıcılar, İşletme, Sistem Yöneticileri, Geliştirme Ekibi, Destek Ekibi  
**Kabul Kriterleri (Acceptance Criteria):**
- Sistem, 24 saatlik test süresince 5 milyon API çağrısını %99.9 başarıyla tamamlamalıdır.  
- Başarısız çağrılar loglanmalı ve hata nedenleri analiz edilebilir olmalıdır.  
- Testler, gerçek kullanım senaryolarına göre gerçekleştirilmelidir.  
**MoSCoW:** Must (temel performans gereksinimi)  
**Impact:** 5 **Effort:** 4 **Risk:** 3  
**Kano Sınıfı:** Performans (Başarı oranı yükseldikçe kullanıcı memnuniyeti artar)  
**Cost of Delay:** Yüksek — başarısızlık, hizmet kesintisi ve itibar kaybına yol açabilir.  

────────────────────────────
**Gereksinim Önerisi 2**

**Gereksinim Türü:** Solution  
**Gereksinim Doğası (F/NF):** Non-Functional  
**Gereksinim:** Sistem, API çağrılarında ortalama yanıt süresini 200 milisaniyenin altında tutmalıdır.  
**Gerekçe:** Düşük gecikme süreleri kullanıcı deneyimini iyileştirir ve sistem performansını artırır.  
**İş Değeri:** Kullanıcı memnuniyetini artırır, sistemin rekabet gücünü yükseltir.  
**Paydaşlar:** Kullanıcılar, İşletme, Sistem Yöneticileri, Geliştirme Ekibi  
**Kabul Kriterleri:**
- Farklı API uç noktalarında yapılan testlerde ortalama yanıt süresi 200 ms altında olmalıdır.  
- En yüksek yanıt süresi 500 ms’yi aşmamalıdır.  
- Testler farklı yük seviyelerinde (ör. 1000 RPS, 5000 RPS) yapılmalıdır.  
**MoSCoW:** Should (önemli performans kriteri)  
**Impact:** 4 **Effort:** 3 **Risk:** 2  
**Kano Sınıfı:** Performans  
**Cost of Delay:** Orta — yavaş yanıt süreleri kullanıcıların sistemi terk etmesine neden olabilir.  

────────────────────────────
**Gereksinim Önerisi 3**

**Gereksinim Türü:** Solution  
**Gereksinim Doğası:** Non-Functional  
**Gereksinim:** Sistem, beklenen yük artışlarını karşılamak için yatayda ölçeklenebilir olmalıdır.  
**Gerekçe:** Artan kullanıcı sayısına ve iş hacmine göre sistemin esnek biçimde büyümesini sağlar.  
**İş Değeri:** İş sürekliliğini destekler, ani yük artışlarına karşı dayanıklılığı artırır, uzun vadeli maliyetleri düşürür.  
**Paydaşlar:** İşletme, Sistem Yöneticileri, Geliştirme Ekibi  
**Kabul Kriterleri:**
- Yeni sunucular eklendiğinde sistem performansı otomatik artmalıdır.  
- Ölçeklendirme kesinti süresi 5 dakikayı geçmemelidir.  
- Ölçeklendirme testleri 5 milyon ve 10 milyon çağrı yüklerinde yapılmalıdır.  
**MoSCoW:** Should (sürdürülebilirlik için önemli)  
**Impact:** 4 **Effort:** 4 **Risk:** 3  
**Kano Sınıfı:** Performans  
**Cost of Delay:** Orta — ölçeklenebilirlik eksikliği, büyüme potansiyelini sınırlar.  

────────────────────────────
**Gereksinim Önerisi 4**

**Gereksinim Türü:** Solution  
**Gereksinim Doğası:** Non-Functional  
**Gereksinim:** Sistem, oluşabilecek hatalar için anlamlı hata mesajları üretmeli ve loglamalıdır.  
**Gerekçe:** Hata durumlarının hızlı tespit edilmesini ve sistem kararlılığının korunmasını sağlar.  
**İş Değeri:** Arıza sürelerini azaltır, destek maliyetlerini düşürür, güven artırır.  
**Paydaşlar:** Geliştirme Ekibi, Destek Ekibi, Sistem Yöneticileri  
**Kabul Kriterleri:**
- Hata mesajları neden ve çözüm önerisi içermelidir.  
- Mesajlar açık, anlaşılır olmalı; teknik jargon minimumda tutulmalıdır.  
- Tüm hatalar zaman damgası ve kaynak bilgisiyle loglanmalıdır.  
- Loglar merkezi sistemde toplanmalı ve analiz edilebilir olmalıdır.  
**MoSCoW:** Must (bakım ve izlenebilirlik için zorunlu)  
**Impact:** 5 **Effort:** 2 **Risk:** 1  
**Kano Sınıfı:** Temel (olmazsa olmaz nitelikte)  
**Cost of Delay:** Yüksek — hataların loglanmaması sistem arızalarına yol açabilir.  

────────────────────────────
**Gereksinim Önerisi 5**

**Gereksinim Türü:** Solution  
**Gereksinim Doğası:** Functional  
**Gereksinim:** Sistem, API kullanım istatistiklerini (çağrı sayısı, yanıt süresi, hata oranı vb.) gerçek zamanlı izleyebilmeli ve raporlayabilmelidir.  
**Gerekçe:** Sürekli performans takibi ve erken anormallik tespiti sağlar.  
**İş Değeri:** Performans optimizasyonu, güvenlik tehditlerinin erken tespiti, daha bilinçli karar alma.  
**Paydaşlar:** İşletme, Sistem Yöneticileri, Geliştirme ve Güvenlik Ekibi  
**Kabul Kriterleri:**
- Sistem metrikleri (çağrı, gecikme, hata oranı) gerçek zamanlı izlenmelidir.  
- Veriler grafik ve tablo formatında görselleştirilmelidir.  
- Eşik aşımlarında uyarılar (e-posta, SMS) gönderilmelidir.  
- Günlük, haftalık, aylık raporlar üretilebilmelidir.  
- Veriler güvenli biçimde saklanmalı ve yetkisiz erişim engellenmelidir.  
**MoSCoW:** Should (izleme ve optimizasyon için önemli)  
**Impact:** 4 **Effort:** 3 **Risk:** 2  
**Kano Sınıfı:** Performans  
**Cost of Delay:** Orta — sorunların geç fark edilmesi mali kayıplara neden olabilir.  


────────────────────────────
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

Bu proje, yapay zekâ destekli gereksinim mühendisliği alanında BABOK (Business Analysis Body of Knowledge) standartlarının sistematik biçimde uygulanabileceğini göstermiştir.

Geleneksel olarak iş analistlerinin yürüttüğü gereksinim tanımlama, sınıflandırma ve önceliklendirme süreçleri; bu proje ile LLM tabanlı otomasyon ve veri odaklı analiz yaklaşımıyla birleştirilmiştir.

Başlıca Bulgular:

- RAG (Retrieval-Augmented Generation) mimarisi, gereksinimlerin yalnızca metinsel olarak değil, bağlamsal ve semantik yakınlığa göre analiz edilmesini sağlamıştır.
- Gemini LLM modeli, kullanıcının doğal dilde ifade ettiği proje açıklamalarını BABOK uyumlu gereksinim alanlarına dönüştürebilmiştir.
- Üretilen çıktılar, Business / Stakeholder / Solution / Transition gereksinim türleriyle uyumlu ve izlenebilir niteliktedir.

Katma Değer:

Bu sistem, iş analizi disiplininde:

- Gereksinim kalitesi ve tutarlılığının artırılmasına,
- Önceliklendirme kararlarının nesnelleştirilmesine,
- Kurumsal bilgi tabanlarının ölçeklenebilir ve yeniden kullanılabilir hâle getirilmesine katkı sağlamaktadır.

Akademik ve Endüstriyel Perspektif:

Proje, gereksinim mühendisliği ve LLM tabanlı bilgi yönetimi arasında köprü kurarak, gelecekte:

- Otomatik gereksinim doğrulama,
- Tutarlılık ve bağımlılık analizi,
- Kurumsal bilgi tabanı entegrasyonu
konularında genişletilebilir bir temel sunmaktadır.

Sonuç olarak, proje BABOK prensipleri, RAG yaklaşımı ve Gemini LLM birleşimiyle, yapay zekânın iş analizi süreçlerinde stratejik bir ortak olarak kullanılabileceğini başarıyla ortaya koymuştur.

```
