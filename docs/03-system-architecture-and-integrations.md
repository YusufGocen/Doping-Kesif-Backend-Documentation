# 3. Mimari ve entegrasyon sınırları

## 3.1 Önerilen teknoloji seti

| **Katman**     | **Teknoloji / sorumluluk**                                                    |
|----------------|-------------------------------------------------------------------------------|
| İstemci        | React + TypeScript; REST çağrıları ve STOMP/WebSocket abonelikleri            |
| Uygulama       | Java 21 + Spring Boot; Controller → Service → Repository                      |
| Kalıcı veri    | PostgreSQL; Flyway migration; JPA/Hibernate                                   |
| Gerçek zamanlı | Spring WebSocket + STOMP; ilk prototip için simple broker                     |
| Güvenlik       | Spring Security + mevcut JWT/SSO doğrulama adaptörü                           |
| Test           | JUnit 5, Mockito, Testcontainers (PostgreSQL), WebSocket entegrasyon testleri |
| Dağıtım        | Docker, ortam değişkenleri, log/metric/trace; altyapı ekibi kararı            |

ÖNERİ: Başlangıçta tek Spring Boot servis (modüler monolit). Birden fazla instance veya yüksek eşzamanlılık gerektiğinde ortak broker (RabbitMQ/Redis pub-sub vb.) ve paylaşımlı oyun durumu değerlendirilir. Tek node simple broker çoklu instance senaryosu için yeterli kabul edilmemelidir.

## 3.2 Bileşen akışı

> React Web → REST /api/v1/kesif/\* → Spring Security → Application Services
>
> ↘ WebSocket /ws/kesif → KeşifGateway → GameEngine
>
> GameEngine → PostgreSQL (oturum, tur, yanıt, koleksiyon)
>
> Adapters → Doping Auth / Doping Butik / Soru Bankası (sözleşmeler AÇIK)

## 3.3 Mevcut sistemlerle entegrasyon

| **Dış bağımlılık** | **Beklenen bilgi**                                              | **Prototip yaklaşımı**                      |
|--------------------|-----------------------------------------------------------------|---------------------------------------------|
| Kimlik doğrulama   | Token doğrulama, userId, rol, oturum süresi                     | Test JWT / mock identity adapter            |
| Doping Butik       | Grup ID, aktif üyelik, arkadaş listesi, yetki                   | Yerel test grupları; adapter arayüzü        |
| Soru bankası       | Soru ID, ders/konu/sınıf, seçenekler, doğru yanıt, yayın durumu | Yerel seed soru verisi; adapter arayüzü     |
| Görsel / CDN       | Puzzle asset ID, URL, kullanım izni                             | Test asset metadata                         |
| Ödül sistemi       | Mevcut XP veya rozet entegrasyonu                               | Önce Keşif-local ledger; entegrasyon sonra |

Kritik ilke: Üretimde mevcut Doping kullanıcı, grup ve soru tablolarının doğrudan kopyalarını oluşturmak yerine, otorite servislerden veri alınmalı. Keşif DB’de yalnızca dış kimlik referansları ve oyuna ait snapshot/transaction kayıtları tutulmalı. Yerel students/groups tabloları yalnızca prototip içindir.

## 3.4 Veri tutarlılığı yaklaşımı

Maçın soru sırası, seçenek dizilimi, doğru cevap snapshot’ı ve puanlama kuralı oyun başlarken sabitlenir. Böylece soru bankası daha sonra güncellense bile geçmiş maçlar deterministik kalır. Doğru cevap snapshot’ı hiçbir oyuncu API’sinde REVEAL öncesi açığa çıkarılmaz.

## 3.5 Yeni ürün akışı: Keşif odası ve katılım yolları

- UI yalnız **Doping Hafıza ana dashboard'unun orta alanını** değiştirir; sol sidebar, üst araç çubuğu, sağ takvim ve promosyon paneli korunur.
- Akış: görev/sınıf/ders/konu → **Arkadaş Seç (Butik)** veya **Oda Oluştur / Odaya Katıl (kod/link)** → lobi → 10 tur → puzzle açılımı → sonuç/koleksiyon.
- **Keşif Room Service**: kod üretimi, link token üretimi, 2 kişilik kapasite, oda sahibi onayı, TTL, tekrar katılım, yarış koşulları. Oda ile gerçek oyun oturumu farklı entity'dir.
- Doping kimliği mevcut servis ile doğrulanır. Kod/link **kimlik doğrulamasının yerine geçmez**.
- Ayrıntı: [16 — Oda tasarımı](16-rooms-codes-and-invite-links.md), [17 — API ve olaylar](17-room-api-and-websocket-contracts.md), [18 — UI sözleşmesi](18-ui-flow-and-integration.md).
