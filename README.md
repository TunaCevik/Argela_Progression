# CDR (Call Detail Record) İşleme Sistemi

## 🚀 Proje Hakkında

Bu proje, Telekomünikasyon şirketleri için **Çağrı Detay Kayıtlarını (CDR)** işleyen, maliyetlendiren, depolayan ve sorgulanabilir hale getiren bir backend sistemidir. Sistem, gelen telefon ve SMS çağrısı kayıtlarını alır, ilgili tarife planlarına göre maliyetlerini hesaplar ve kalıcı olarak veritabanına kaydeder. Ayrıca, geçmiş çağrı kayıtlarını esnek filtreleme seçenekleriyle sorgulama yeteneği sunar.

Sistem, modern ve katmanlı bir mimariyle tasarlanmış olup, **SOLID prensiplerine** uygunluk ve yüksek düzeyde genişletilebilirlik hedeflenmiştir.

## ✨ Temel Özellikler

- **Telefon Çağrısı İşleme:** Gelen telefon çağrısı CDR'larını ayrıştırır, maliyetini hesaplar ve kaydeder.
- **SMS Çağrısı İşleme:** Gelen SMS çağrısı CDR'larını ayrıştırır, maliyetini hesaplar ve kaydeder.
- **Başarısız Çağrı Yönetimi:** Başarısız olan çağrıları kaydeder ve maliyetlendirme süreçlerini buna göre yönetir (genellikle maliyet sıfır).
- **Esnek CDR Sorgulama:** Başlangıç/bitiş zamanı, numara, çağrı tipi ve sonuç gibi çeşitli kriterlere göre geçmiş çağrı kayıtlarını filtreleme ve sorgulama imkanı.
- **Tarife Planı Yönetimi:** Sesli ve SMS çağrıları için farklı tarife planlarını destekler ve maliyet hesaplamalarında bu planları kullanır.

## 🛠️ Teknolojiler

- **Programlama Dili:** Java 21
- **Web Çatısı:** Quarkus 3.20.2 (API endpointleri için)
- **Veritabanı:** PostgreSQL
- **Veritabanı Konteynerizasyonu:** Docker
- **ORM (Object-Relational Mapping):** Hibernate / JPA (Panache ile)
- **Bina Aracı:** Gradle
- **Modelleme:** UML (Unified Modeling Language) - Class & Sequence Diagrams
- **API Test Aracı:** Thunder Client (VS Code Eklentisi)

## 🏗️ Mimari Tasarım

Proje, katmanlı bir mimariye sahiptir:

1.  **Controller Katmanı:** Dış dünyadan gelen HTTP isteklerini karşılar, doğrular ve Servis katmanına yönlendirir. RESTful API prensiplerine uygun olarak tasarlanmıştır.
2.  **Service Katmanı:** İş mantığını içerir. Controller'dan gelen talepleri işler, gerekli hesaplamaları (örn. maliyet) yapar ve Repository katmanıyla etkileşime girerek veritabanı işlemlerini koordine eder.
3.  **Repository Katmanı:** Veritabanı ile doğrudan iletişim kurar. CRUD (Create, Read, Update, Delete) operasyonlarını fiziksel olarak yürütür ve Hibernate/JPA aracılığıyla Java nesnelerini veritabanı kayıtlarına eşler.
4.  **Domain/Model Katmanı:** Projenin iş varlıklarını (Call, VoiceCallRatePlan, SmsCallRatePlan vb.) ve bunların ilişkilerini temsil eden POJO (Plain Old Java Object) sınıflarını içerir. Bu katman, iş mantığının çekirdeğidir ve framework bağımsızlığı hedefler.

### UML Diyagramları

Projenin temel yapıları ve dinamik davranışları UML diyagramları ile modellenmiştir:

- **Sınıf Diyagramı (Class Diagram):** Sistemdeki ana varlıkları (`Call`, `VoiceCallRatePlan`, `SMSCallRatePlan` vb.) ve aralarındaki statik ilişkileri (kalıtım, ilişkilendirme, gerçekleştirme) gösterir.
- **Sıra Diyagramları (Sequence Diagrams):** Belirli senaryoların (örn. Telefon Çağrısı İşleme, SMS Çağrısı İşleme, CDR Sorgulama, Başarısız Çağrı İşleme) zaman içindeki mesaj akışını ve nesneler arası etkileşimleri görselleştirir.

## 🗄️ Veritabanı Şeması (Hibernate / JPA Mappings)

Proje, PostgreSQL üzerinde tablolarını **Hibernate'in JPA anotasyonları aracılığıyla otomatik olarak oluşturacaktır.** Aşağıdaki şema, her tablonun hangi Java tipleriyle eşleştiğini göstermektedir:

### `calls` Tablosu

Tüm telefon ve SMS çağrısı kayıtlarının ortak ve türe özel niteliklerini içeren ana tablo. `call_type` sütunu kaydın türünü belirtir.

| Java Tipi       | Veritabanı Sütun Adı    | Kısıtlamalar                     | Açıklama                                                |
| :-------------- | :---------------------- | :------------------------------- | :------------------------------------------------------ |
| `Long`          | `id`                    | `PRIMARY KEY`                    | Benzersiz çağrı kaydı ID'si                             |
| `String`        | `call_type`             | `NOT NULL`                       | `'VOICE_CALL'` veya `'SMS_CALL'`                        |
| `LocalDateTime` | `start_time`            | `NOT NULL`                       | Çağrının başlama zamanı                                 |
| `String`        | `imsi`                  | `NOT NULL`                       | Abonenin Uluslararası Mobil Abone Kimliği               |
| `String`        | `imei`                  |                                  | Abonenin Uluslararası Mobil Ekipman Kimliği             |
| `Long`          | `cell_id`               |                                  | Çağrının yapıldığı hücre kulesi ID'si                   |
| `Long`          | `lac_id`                |                                  | Çağrının yapıldığı konum alanı kodu ID'si               |
| `String`        | `a_number`              | `NOT NULL`                       | Arayan numara                                           |
| `String`        | `b_number`              | `NOT NULL`                       | Aranan numara                                           |
| `String`        | `direction`             | `NOT NULL`                       | Çağrı yönü (`'MO'` - Mobil Giden, `'MT'` - Mobil Gelen) |
| `String`        | `result`                | `NOT NULL`                       | Çağrı sonucu (`'SUCCESS'`, `'FAIL'`)                    |
| `Double`        | `cost`                  | `NOT NULL` (varsayılan 0)        | Çağrının hesaplanan maliyeti                            |
| `LocalDateTime` | `end_time`              |                                  | Telefon çağrısı bitiş zamanı                            |
| `Long`          | `setup_duration`        |                                  | Kurulum süresi (milisaniye)                             |
| `Long`          | `conversation_duration` |                                  | Konuşma süresi (milisaniye)                             |
| `Integer`       | `message_length`        |                                  | SMS mesaj uzunluğu                                      |
| `Long`          | `voice_rate_plan_id`    | `FK -> voice_call_rate_plans.id` | Telefon çağrısı için tarife planı ID'si                 |
| `Long`          | `sms_rate_plan_id`      | `FK -> sms_call_rate_plans.id`   | SMS çağrısı için tarife planı ID'si                     |
| `LocalDateTime` | `created_at`            |                                  | Kaydın oluşturulma zamanı                               |
| `LocalDateTime` | `updated_at`            |                                  | Kaydın son güncellenme zamanı                           |

### `voice_call_rate_plans` Tablosu

Sesli arama tarife planlarını saklar.

| Java Tipi       | Veritabanı Sütun Adı | Kısıtlamalar                | Açıklama                   |
| :-------------- | :------------------- | :-------------------------- | :------------------------- |
| `Long`          | `id`                 | `PRIMARY KEY`               | Tarife planı ID'si         |
| `String`        | `plan_name`          | `NOT NULL`, `UNIQUE`        | Tarife planının adı        |
| `Double`        | `rate_per_second`    | `NOT NULL`                  | Saniye başına ücret        |
| `Boolean`       | `is_active`          | `NOT NULL` (`DEFAULT TRUE`) | Planın aktif olup olmadığı |
| `LocalDateTime` | `created_at`         |                             |                            |
| `LocalDateTime` | `updated_at`         |                             |                            |

### `sms_call_rate_plans` Tablosu

SMS tarife planlarını saklar.

| Java Tipi       | Veritabanı Sütun Adı | Kısıtlamalar                | Açıklama                   |
| :-------------- | :------------------- | :-------------------------- | :------------------------- |
| `Long`          | `id`                 | `PRIMARY KEY`               | Tarife planı ID'si         |
| `String`        | `plan_name`          | `NOT NULL`, `UNIQUE`        | Tarife planının adı        |
| `Double`        | `rate_per_segment`   | `NOT NULL`                  | SMS segmenti başına ücret  |
| `Boolean`       | `is_active`          | `NOT NULL` (`DEFAULT TRUE`) | Planın aktif olup olmadığı |
| `LocalDateTime` | `created_at`         |                             |                            |
| `LocalDateTime` | `updated_at`         |                             |                            |
