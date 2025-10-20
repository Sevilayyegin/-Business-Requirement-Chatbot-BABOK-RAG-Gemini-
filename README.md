# 🔹 BABOK + RAG + Gemini Tabanlı Business Requirement Chatbot

Gemini LLM + ChromaDB + Gradio altyapısıyla çalışan bir Business Requirement Chatbot

---
## 🎯 Projenin Amacı

Bu proje, iş analizi sürecinde **gereksinimlerin tanımlanması, sınıflandırılması ve önceliklendirilmesini otomatikleştirmek** için geliştirilmiştir.  
Geleneksel olarak saatler süren gereksinim dokümantasyonu artık birkaç saniye içinde otomatik üretilmektedir.

### Sistem Özellikleri:
- 🧠 **Veri temelli (RAG destekli)**
- 📘 **Uluslararası standartlara uygun (BABOK)**
- ⚙️ **Ölçeklenebilir (ChromaDB + Gemini)**
- 💬 **Kullanıcı dostu (Gradio arayüzü)**

Yapay zekâ ve veri tabanı tekniklerini bir araya getirerek sistem:
- Gereksinimin **Functional** veya **Non-Functional** olduğunu tahmin eder,  
- **BABOK** standartlarına göre gereksinim dokümanı üretir,  
- **RICE** ve **WSJF** metrikleriyle önceliklendirme yapar,  
- Etkileşimli bir **Gradio** arayüzü üzerinden kullanıcıya sunar.  

Kısaca, bu chatbot bir iş analistinin yaptığı “gereksinim çıkarımı, analizi ve dokümantasyon” sürecini kısmen otomatikleştirir.

---

## 🧩 Mimarî Bileşenler

![architecture](Business%20Requirement%20Chatbot.png)

> 💡 Şema: Kullanıcı → Embedding (Gemini) → ChromaDB (RAG) → LLM (Gemini) → BABOK Uyumlu Gereksinim → Önceliklendirme (RICE / WSJF) → Gradio Arayüzü


## 🧩 Sistem Katmanları ve Kullanılan Teknolojiler

| Katman | Teknoloji | Açıklama |
|:----------------------|:---------------------------|:--------------------------------------------------------------------------------------------|
| 🤖 **Gemini (LLM)** | Google Gemini (1.5 Flash / Pro) | Gereksinimleri anlamlandırır ve BABOK uyumlu metin üretir. |
| 🧠 **PURE Annotate Dataset** | Kaggle (Açık veri kümesi) | Modelin eğitildiği veya örnekleme yaptığı kamuya açık gereksinim verisidir. |
| 📊 **ChromaDB** | Vektör Veritabanı | Gereksinim verilerini vektör biçiminde depolar, benzerlik araması sağlar. |
| 🔁 **RAG Pipeline** | text-embedding-004 + Gemini | Sorgudan bilgi getirir (**Retrieval**) ve Gemini ile çıktı üretir (**Generation**). |
| 🧰 **Framework** | Python 3.10+, pandas, dotenv, gradio | Modelin altyapısını ve veri işleme akışını sağlar. |
| ⚖️ **Önceliklendirme** | RICE & WSJF Modelleri | Gereksinimlerin iş değerine göre sıralanmasını sağlar. |
| 💬 **Gradio Arayüzü** | Gradio | Kullanıcı dostu bir web arayüzü sunar. |

---

## 🚀 Kurulum ve Gereksinimler

### 1️⃣ Gerekli Paketleri Yükle
```bash
pip install -q chromadb sentence-transformers google-generativeai python-dotenv gradio pandas
pip install -q sentence-transformers==2.2.2 transformers==4.41.2 huggingface_hub==0.22.2
````

> 💡 Colab veya Kaggle kullanıyorsan ayrıca:
>
> ```bash
> !pip install -q --upgrade "google-generativeai>=0.7.2"
> ```

---

### 2️⃣ Ortam Değişkeni ve API Anahtarı

`.env` dosyanı oluştur:

```bash
GOOGLE_API_KEY="your_api_key_here"
```

Proje içinde:

```python
from dotenv import load_dotenv
load_dotenv()
import google.generativeai as genai
genai.configure(api_key=os.getenv("GOOGLE_API_KEY"))
```

3️⃣ Veri Kümesini Ekleyin

📊 Proje, PURE Annotate Dataset (Kaggle) verisini kullanır:
Kaggle Dataset Linki → [PURE Annotate Dataset](https://www.kaggle.com/datasets/computerscience3/public-requirementspure-dataset)

Dosyayı indirdikten sonra proje dizinine şu şekilde yerleştirin:
```bash
/data/Pure_Annotate_Dataset.csv
````
---

## 📊 Veri Seti: PURE Annotate Dataset

Model, Kaggle’daki **PURE Annotate Dataset** üzerinde eğitilmiştir.
Her satır bir gereksinimi temsil eder:

* Functional → “System shall export reports in PDF.”
* Non-Functional → “System shall encrypt user data at rest.”

Etiketleme Mantığı
- NFR_boolean = 0: Bu gereksinim işlevsel (Functional Requirement) olarak yorumlanır.
- NFR_boolean = 1: Bu gereksinim işlevsel olmayan (Non-Functional Requirement) olup kalite, güvenlik, performans gibi unsurları kapsar.

---

## 🔡 Embedding Aşaması (Gemini)

Metinler anlamca karşılaştırılabilir hale getirilir.
Bu işlem, her gereksinim cümlesini sayısal bir **embedding vektörüne** dönüştürür.

```python
emb = genai.embed_content(model="models/text-embedding-004", content=text)
```

---

## 🧱 ChromaDB Knowledge Base

Gereksinim embedding’leri **ChromaDB** üzerinde saklanır.
Bu yapı, semantik benzerlik aramaları yapılmasını sağlar.

```python
collection = client.create_collection(name="requirements_kb", metadata={"hnsw:space": "cosine"})
collection.add(ids=kb["id"].tolist(), documents=kb["text"].tolist(), embeddings=kb_embeddings)
```

---

## 🧠 BABOK Uyumlu Prompt Tasarımı

Model, aşağıdaki şablona göre çıktı üretir:

| Alan                                    | Açıklama                                       |
| --------------------------------------- | ---------------------------------------------- |
| **Gereksinim Türü**                     | Business / Stakeholder / Solution / Transition |
| **Doğa**                                | Functional veya Non-Functional                 |
| **Rationale**                           | Gerekçe                                        |
| **Business Value**                      | Katma değer                                    |
| **Acceptance Criteria**                 | Ölçülebilir kabul kriterleri                   |
| **MoSCoW**, **Kano**, **Cost of Delay** | Önceliklendirme metrikleri                     |

> Model yanıtlarını Türkçe ve BABOK formatında üretir.

---

## 📘 BABOK Uyumlu Çıktı Formatı

Her yanıt, **BABOK (Business Analysis Body of Knowledge)** rehberine göre yapılandırılmış gereksinim alanlarını içerir.

---

| Alan | Açıklama / Örnek |
|:------------------------------|:---------------------------------------------------------------|
| **Gereksinim Türü** | Solution |
| **Gereksinim Doğası (F/NF)** | Non-Functional |
| **Gereksinim** | Sistem tüm müşteri verilerini **AES-256** ile şifrelemelidir. |
| **Rationale (Gerekçe)** | Veri gizliliği ve regülasyon uyumu için. |
| **Business Value** | Yüksek |
| **Stakeholders (Paydaşlar)** | Güvenlik Ekibi, BT, Uyumluluk |
| **Acceptance Criteria (Kabul Kriteri)** | Tüm verilerin **KVKK** ve **PCI-DSS** standartlarına uygun olarak şifrelenmesi. |
| **MoSCoW Önceliği** | Must *(Regülasyon gereği)* |
| **Impact / Effort / Risk** | Impact: 4 • Effort: 3 • Risk: 5 |
| **Kano Kategorisi** | Temel Gereksinim *(Zorunlu güvenlik önlemi)* |
| **Cost of Delay (Gecikme Maliyeti)** | Veri sızıntısı riski → **Çok yüksek maliyet** |

---

💡 **Not:**  
Bu format, RAG (Retrieval-Augmented Generation) yaklaşımıyla elde edilen gereksinimlerin,  
**BABOK** standardında izlenebilir ve ölçülebilir hale getirilmesini sağlar.

---

## 📊 Önceliklendirme Modülleri

Proje iki farklı metrik kullanarak öncelik belirler:

| Metrik | Formül | Amaç |
|:--------|:--------------------------------------------------------------|:---------------------------------------------------------------|
| **RICE** | (Reach × Impact × Confidence) / Effort | Kullanıcı erişimi, etkisi ve güven düzeyine göre öncelik puanı hesaplar. |
| **WSJF** | (Business Value + Time Criticality + Risk Reduction) / Job Size | Ekonomik değer, zaman baskısı ve risk azaltmayı dikkate alarak iş sırasını belirler. |

---

🔹 **RICE** modeli genellikle ürün özelliklerinin etki ve erişimine göre sıralama yaparken,  
🔹 **WSJF (Weighted Shortest Job First)** yaklaşımı ekonomik değer ve zaman kritikliğine göre optimizasyon sağlar.  

---

## 📈 Gereksinim Önceliklendirme Modülü

Gereksinimler, otomatik olarak **RICE** ve **WSJF** metriklerine göre puanlanır.

**RICE = (Reach × Impact × Confidence) / Effort**
**WSJF = (Business Value + Time Criticality + Risk Reduction) / Job Size**

Heuristik sinyaller:

* `güvenlik`, `kvkk` → yüksek risk
* `performans`, `ölçeklenebilirlik` → yüksek etki
* `yedekleme`, `erişilebilirlik` → yüksek zaman kritiği

---

## 🔍 RAG (Retrieval-Augmented Generation) Pipeline

1. **Retrieval (Getirme)** → Kullanıcı sorgusu embedding’e çevrilir ve ChromaDB’de benzer gereksinimler aranır.
2. **Augmentation (Zenginleştirme)** → Bulunan örnekler modele verilir.
3. **Generation (Üretim)** → Gemini, bağlama uygun yeni gereksinimler üretir.

```python
response = model.generate_content(prompt)
```

Bu sayede model hem veriye dayalı hem de yaratıcı içerik üretir.

---

## 💬 Gradio Arayüzü

Gradio, kullanıcıdan proje açıklamasını alıp modeli çalıştırır.
- Kullanıcı metin kutusuna proje açıklamasını girer.
- Chatbot, sorguya özel BABOK uyumlu gereksinim önerileri döndürür.

```python
demo = gr.Interface(
    fn=chatbot_interface,
    inputs=gr.Textbox(lines=4, label="💬 Proje Açıklaması"),
    outputs=gr.Markdown(label="📘 BABOK Uyumlu Gereksinim Önerileri"),
    title="Business Requirement Chatbot (BABOK + RAG)",
    description="PURE Dataset + BABOK Framework + Gemini RAG tabanlı iş analizi asistanı",
)
```

Çalıştırmak için:

```bash
python app.py
```

---

## 🎯 Örnek Sorular
"Kullanıcı cep telefonundan para transferi yapabilmeli ve IBAN kaydedebilmeli",
"Sistem en yoğun saatte ortalama 2 saniye altında yanıt vermeli",
"Tüm müşteri verileri dinamik ve durağan halde şifrelenmeli",
"Rol tabanlı erişim kontrolü uygulanmalı ve ince taneli yetkilendirme olmalı",
"Mobil uygulama WCAG 2.1 AA erişilebilirlik kriterlerini sağlamalı",
"Hata mesajları kullanıcı dostu olmalı ve teknik detay sızdırmamalı",
"Raporlama modülü PDF/CSV dışa aktarımı ve zamanlanmış e-posta gönderimini desteklemeli",
"Sistem başarısız girişlerde hesabı geçici kilitlemeli ve MFA'yı zorunlu kılmalı"

## 🧪 Test Çalışması

```python
test_query = "Günlük 5 milyon API çağrısını %99.9 başarı ile işleyebilmelidir."
print(rag_response_babok(test_query))
```

---

## 🗂️ Proje Yapısı

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

---

## 🧩 Elde Edilen Sonuçlar

Bu proje, yapay zekâ destekli gereksinim mühendisliğinde BABOK standartlarının otomatikleştirilebileceğini göstermiştir.
RAG (Retrieval-Augmented Generation) mimarisi sayesinde sistem, sadece genel metin üretmekle kalmayıp, gerçek gereksinim örneklerinden öğrenerek bağlama uygun öneriler sunabilmiştir.

Gemini modeli, kullanıcıdan gelen doğal dildeki proje açıklamalarını analiz ederek;

- Gereksinim türü (Business, Stakeholder, Solution, Transition),
- (Rationale),
- (Business Value) ve
- Acceptance Criteria)

gibi alanları sistematik biçimde üretebilmiştir.

Örnek senaryolarda, modelin oluşturduğu gereksinimler:

- Bağlama semantik olarak uygun,
- Türkçe dilinde doğal ve tutarlı,
- BABOK terminolojisine uygun bulunmuştur.

Sonuç olarak proje, LLM tabanlı gereksinim analizi ile vektör veri tabanı destekli bilgi çağırma yöntemlerini başarılı biçimde birleştirerek;
gelecekte otomatik iş analizi, gereksinim kalitesi değerlendirmesi ve kurumsal bilgi yönetimi alanlarında kullanılabilecek ölçeklenebilir bir temel oluşturmuştur.

```

