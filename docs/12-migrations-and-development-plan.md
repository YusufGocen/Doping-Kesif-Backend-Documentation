# 12. Migration, geliştirme planı ve açık kararlar

## 12.1 Flyway migration sırası

22. V1\_\_kesif_schema_and_catalog.sql — schema, subject, topic, puzzle, question (prototip).

23. V2\_\_kesif_invitation_and_session.sql — invitation, session, session_player.

24. V3\_\_kesif_round_and_answer.sql — round, answer, unique/index/check.

25. V4\_\_kesif_collection_and_rewards.sql — collection, piece, XP ledger.

26. V5\_\_kesif_outbox.sql — güvenilir olay yayını.

27. V6\_\_kesif_seed_dev.sql — yalnız local/dev ortamı için örnek öğrenci, grup, soru ve puzzle verileri.

## 12.2 Kodlama iş paketleri

| **Sıra** | **İş paketi**                            | **Çıktı / kabul**                      |
|----------|------------------------------------------|----------------------------------------|
| 1        | Spring Boot iskeleti, PostgreSQL, Flyway | Uygulama ayağa kalkar; migration geçer |
| 2        | Catalog + puzzle + soru seed             | GET katalog endpoint’leri              |
| 3        | Identity / Butik adapter                 | Test grubu ve yetki kontrolü           |
| 4        | Invitation + lobby                       | Davet kabul/ret ve ready               |
| 5        | Session + 10 soru snapshot               | Aynı soru sırası ve gizli cevap        |
| 6        | Answer + round engine                    | Eşzamanlı güvenli cevap işleme         |
| 7        | WebSocket olayları                       | İki tarayıcı senkron oynar             |
| 8        | Puzzle + koleksiyon + XP                 | Mükerrersiz final güncellemesi         |
| 9        | Reconnect + outbox + güvenlik            | Kesinti ve yetkisiz erişim testleri    |
| 10       | E2E + performans + Swagger               | Demo ve teknik teslim                  |

## 12.3 API ve DB teslim dosyaları

- README.md — proje kurulumu, mimari ve çalıştırma.

- docs/architecture.md — mimari, sınırlar, state machine.

- docs/database.md + ERD — tablo sözlüğü, indeks, ilişki.

- docs/api.yaml — OpenAPI 3.1 REST sözleşmesi.

- docs/websocket.md — topic, komut, event payload ve sıra.

- docs/game-rules.md — cevap, puzzle, koleksiyon, XP politikası.

- src/main/resources/db/migration/\*.sql — Flyway scriptleri.

- docker-compose.yml — local PostgreSQL ve servisler.

- tests/ — unit, integration, concurrency, E2E senaryoları.

## 12.4 Açık karar listesi (onay gerektirir)

| **No** | **Konu**                                              | **Öneri / karar sahibi** |
|--------|-------------------------------------------------------|--------------------------|
| A01    | Gerçek Auth/JWT sözleşmesi                            | Platform backend ekibi   |
| A02    | Butik grup üyeliği ve arkadaş listesi API’si          | Butik ekibi              |
| A03    | Soru bankası API ve lisans/kullanım kuralı            | İçerik + backend         |
| A04    | Soru başına süre ve timeout davranışı                 | Ürün ekibi               |
| A05    | REVEAL sonrası otomatik ilerleme veya iki taraf onayı | Ürün + UX                |
| A06    | Bağlantı kopunca bekleme / maçı sonlandırma           | Ürün + backend           |
| A07    | XP değerleri ve mevcut ödül sistemi                   | Ürün + gamification      |
| A08    | Kişisel koleksiyon parçası kazanım algoritması        | Ürün + backend           |
| A09    | Maç ve cevap verisi saklama süresi                    | Güvenlik + hukuk         |
| A10    | Soru medyası ve puzzle asset erişimi                  | Frontend + içerik        |
| A11    | Eşzamanlı kullanıcı kapasitesi / deployment           | Altyapı + backend        |
| A12    | Aynı anda aktif davet/maç limiti                      | Ürün + backend           |

## 12.5 Definition of Done

Kod review, migration geri dönüş planı, Swagger sözleşmesi, iki oyunculu E2E demo, testlerde kritik race condition kapsamı, yetki ve güvenlik kontrolleri, temel metrikler, README ve açık kararların kapatılması tamamlandığında MVP backend teslim edilir.

## 12.6 Yeniden adlandırma ve oda/link için ek iş paketleri

**Önceki ürün adı:** Doping Battle. **Güncel ürün adı:** Doping Keşif. Bu paket, sıfırdan geliştirilecek proje için Java paket adını `com.doping.kesif`, PostgreSQL şemasını `kesif`, tabloları `kesif_*`, REST prefix'ini `/api/v1/kesif`, WebSocket'i `/ws/kesif` olarak tanımlar. **Mevcut çalışan veritabanı varsa isimleri otomatik değiştirmeyin; migration ve veri taşıma planı çıkarın.**

| Aşama | İş | Kabul |
|---|---|---|
| A | Auth ve Doping Butik entegrasyon sözleşmelerini doğrula | JWT ve grup ID formatları belgeli |
| B | Oda entity + Flyway V7 taslağı | Oda oluşturma ve TTL testleri |
| C | Kod/link token servisi ve rate-limit | Tahmin/tekrar deneme testleri |
| D | Oda katılımı ve isteğe bağlı host onayı | Kapasite yarış testi |
| E | Butik daveti ve oda akışını birleştir | Her iki yol aynı oyun motoruna çıkar |
| F | React orta alan ekranları | Sidebar/üst/sağ paneller değişmeden çalışır |
| G | WebSocket oda olayları + reconnect | İki tarayıcı senkronize |
| H | Swagger ve E2E demo | Kod, link, Butik ve oyun testleri |

Yeni açık kararlar **A13–A18**: A13 grup dışından kod/link ile katılım; A14 oda sahibi onayı; A15 oda/kod/link TTL; A16 kod biçimi ve paylaşım; A17 link ile girişte oturum açma yönlendirmesi; A18 oda kapanması/yeniden giriş ve aynı anda oda limiti.
