> **GÜNCELLEME — DOPING KEŞİF (2026-10-23):** Bu belge önceki ana geliştirme rehberinin isimleri güncellenmiş sürümüdür. Yeni **kod/link ile oda** özelliği için [16](16-rooms-codes-and-invite-links.md)–[20](20-product-decisions-and-change-log.md) bölümleri **önceliklidir**. Eski metinde “yalnız aynı Butik grubu”, “yalnız davet” gibi ifadeler geçiyorsa yalnız **Butik arkadaşını seç** akışını anlatır; kod/link akışına uygulanmaz. `kesif_room`, `kesif_room_join_request` ve `kesif_session.room_id` eklenmeden geliştirmeye başlamayın.

# Doping Keşif — Proje Ana Dokümanı ve Codex Geliştirme Rehberi

> **Durum:** Tasarım / uygulama öncesi \| **Sürüm:** 1.0 \|
> **Platform:** Doping Hafıza Web  
> **Stack (önerilen):** React + TypeScript + Vite + Tailwind CSS; Java
> 21 + Spring Boot 3.x; PostgreSQL 16+; Spring Security + JWT; REST +
> STOMP/WebSocket; Flyway; springdoc-openapi/Swagger UI; Docker Compose;
> JUnit 5 + Testcontainers.  
> **Kaynak:** Daha önce hazırlanan *Doping Keşif Backend Teknik Tasarım
> Dokümanı* ve bu projede netleştirilmiş ürün kuralları. Yüklenen
> `doping-pulse-main.zip`, **sadece dokümanların organizasyon biçimi
> için örnektir**; Pulse'a ait karar motoru, Kafka, mikro-müdahale ve
> diğer alan kuralları Keşif'e aktarılmayacaktır.

## 0. Codex için temel çalışma talimatı

Bu dosya projenin **ana gereksinim ve uygulama planıdır**. Kod yazmadan
önce tamamını oku. Bir adımı bitirmeden sonraki adıma geçme. Özellikle
`## 15. Geliştirme aşamaları` bölümündeki kontrol listelerini uygula.

1.  **Önce plan ve mimari:** Mevcut repository'yi incele, bulunan kodu
    silme/yeniden yazma; eksikleri ve önerilen dosyaları özetle.
2.  **Önce doküman, sonra migration, sonra backend, sonra frontend
    entegrasyonu.** Tek komutla her şeyi üretmeye çalışma.
3.  **Netleşmemiş iş kararlarını uydurma.** `KARAR BEKLİYOR` alanlarını
    konfigüre edilebilir tasarla, varsayılan değerleri açıkça işaretle.
4.  **Üretim entegrasyonu varmış gibi davranma.** Doping Hafıza kimlik
    doğrulama, Butik grup ve soru bankası için önce port/adapter
    arayüzleri ve yalnızca `local` profilinde çalışan mock adapter'lar
    yaz.
5.  **Veritabanı şeması Flyway ile yönetilsin.** `ddl-auto=validate`;
    `create`/`update` kullanma. Migration geri alınması ve veritabanı
    yedeklemesi planlansın.
6.  **Güvenlik zorunlu:** Yetkilendirme sadece frontend'de yapılmaz;
    JWT, grup üyeliği, maç katılımcılığı ve soru/cevap gizliliği
    backend'de doğrulanır.
7.  **Gerçek zamanlı doğruluk zorunlu:** Aynı turdaki iki cevap yarış
    koşullarında tutarlı biçimde işlenmeli; çift cevap, çift ödül ve
    çift puzzle açılması engellenmeli.
8.  **Her fazın sonunda:** Değişen dosyaları, çalıştırılan komutları,
    test sonucunu, bilinen riskleri ve sıradaki fazı raporla. Test
    edilmemiş bir özelliği tamamlandı diye işaretleme.
9.  **Kod kalitesi:** DTO validation, merkezi exception handling,
    transaction sınırları, structured logging, testler, açık
    isimlendirme ve okunabilir modüler yapı.
10. **Dil:** Kod ve API adları İngilizce; README, karar kayıtları ve
    kullanıcıya dönük açıklamalar Türkçe olabilir.

## 1. Proje özeti

**Doping Keşif**, aynı **Doping Butik grubundaki** iki öğrencinin
eşzamanlı 10 soruluk bir oyunda birlikte puzzle parçaları açtığı,
rekabetten çok ortak ilerlemeyi öne çıkaran eğitsel bir deneyimdir.

- Her oyun **tam 2 öğrenci** ile oynanır; rastgele eşleşme yoktur.
- İki öğrenci **aynı soruyu aynı anda** görür; cevaplar bağımsızdır.
- **10 soru** vardır; her doğru cevap **1 parça**, her tur en fazla **2
  parça** kazandırır.
- Ortak oyun puzzle'ı **20 parça**dır. Sorular ekranında gizli görsel
  açılmaz; tur sonundaki ayrı puzzle/ilerleme ekranında gösterilir.
- Her oyun yeni bir **ortak oturum ilerlemesi** başlatır; her öğrencinin
  **kişisel koleksiyonu kalıcıdır**.
- Ders/sınıf/konu seçilir; **manuel zorluk seviyesi seçimi yoktur**.
- Oyun içi sohbet, rastgele oyuncu, liderlik tablosu, rekabetçi skor
  sıralaması yoktur.
- Web arayüzü **açık/beyaz tema**, Doping Hafıza'nın mevcut **sol
  sidebar yapısı ve markası** ile uyumlu olmalıdır.

### 1.1 MVP kapsamı

Giriş/kimlik doğrulama; Butik grup arkadaşlarını listeleme; puzzle
seçme; sınıf/ders/konu seçimi; davet; lobi/hazır olma; 10 turluk
eşzamanlı soru akışı; doğru cevap değerlendirme; puzzle açılması; sonuç;
XP ve kişisel koleksiyon; oyun geçmişi; yeniden bağlanma; temel
yönetim/test verisi.

### 1.2 MVP dışı

3+ oyuncu, rastgele eşleşme, chat, sesli/video görüşme, turnuva, global
leaderboard, mobil uygulamaya özel ekranlar, gerçek ödeme, gerçek push
altyapısı, Kafka/mikroservis zorunluluğu.

### 1.3 Başarı kriterleri

- Grup dışından bir oyuncu davet edilemez.
- Sadece davetli oyuncu daveti kabul edebilir.
- Bir maçta tam 2 oyuncu ve 10 sabit soru bulunur.
- Soru sırası her iki istemcide aynıdır; cevap anahtarı soru
  gösterilirken gönderilmez.
- Aynı oyuncunun aynı tura ikinci cevabı puan kazandırmaz.
- İki cevap gelince (veya tanımlı timeout kuralı gerçekleşince) tur
  **bir kez** kapanır.
- Tur başına 0/1/2, maç başına en çok 20 ortak parça açılır.
- Oyun sonucu, XP ve koleksiyon güncellemeleri mükerrer isteklerde
  değişmez.
- Yetkisiz erişim ve başkasının maçına WebSocket aboneliği engellenir.
- Geçici bağlantı kopması sonrası istemci sunucu durumundan devam
  edebilir.

## 2. Roller ve erişim

| Rol                      | Yapabilecekleri                                                                        | Sınırlar                                                |
|--------------------------|----------------------------------------------------------------------------------------|---------------------------------------------------------|
| `STUDENT`                | Görevleri görme, grup arkadaşına davet, kendi maçına katılma, cevap, koleksiyon/geçmiş | Başkasının grubunu, cevabını veya özel verisini göremez |
| `ADMIN` (yerel prototip) | Test soruları ve puzzle yönetimi, test verisi yükleme                                  | Üretimde kurumun mevcut yetki modeliyle eşleştirilmeli  |
| `SYSTEM`                 | Süre aşımı, maç sonlandırma, ödül yazma, outbox yayınlama                              | İç servis/worker yetkisi                                |

**Öğrenci kimliği:** JWT `sub` claim'i güvenilir kullanıcı ID'sine
eşlenir. İstemciden gönderilen `studentId` kimlik kaynağı olarak
kullanılmaz.

## 3. Kullanıcı akışı

1.  Öğrenci Doping Hafıza oturumuyla Keşif ekranına girer.
2.  Puzzle görevi (Uzay Macerası, Tarih Yolculuğu, Okyanus Keşfi,
    Gizemli Orman) seçer.
3.  Sınıf (5–8), ders, konu seçer. Zorluk sistem tarafından belirlenir.
4.  Doping Butik grubundaki uygun arkadaş listesini görür ve **bir
    kişiyi** davet eder.
5.  Davetli öğrenci kabul/reddeder; kabulden sonra iki oyuncu lobide
    hazır olur.
6.  Backend aynı 10 soruluk **değişmez soru setini** seçip maçı
    başlatır.
7.  Her turda iki öğrenci aynı soruyu görür; kendi cevabını gönderir.
8.  Her iki cevap tamamlanınca sunucu sonuç ve açılacak parça bilgisini
    üretir.
9.  İstemci **ayrı puzzle ekranını** gösterir; iki oyuncu ilerlemeyi
    aynı görür.
10. Her iki oyuncu ilerlemeye hazır olduğunda veya yapılandırılmış süre
    dolunca sıradaki soru açılır.
11. 10. turdan sonra maç biter; kişisel koleksiyon, XP ve geçmiş
        güncellenir.

### 3.1 Ekran listesi (frontend)

`KeşifHomePage`, `PuzzleSelectionPage`, `KeşifSetupPage`,
`FriendSelectionPage`, `KeşifLobbyPage`, `KeşifQuestionPage`,
`PuzzleRevealPage`, `KeşifResultPage`, `CollectionPage`,
`KeşifHistoryPage`.

### 3.2 UI ilkeleri

- Var olan Doping Hafıza sidebar'ı ve logo alanı korunmalı; açık tema ve
  erişilebilir kontrast kullanılmalı.
- İki oyuncunun adları/avatarları sade biçimde gösterilmeli; kişisel
  veri asgari düzeyde tutulmalı.
- Soru ekranında görseli erkenden açan bir önizleme bulunmamalı.
- Yükleme, davet bekleme, rakip cevabı bekleme, bağlantı koptu ve tekrar
  bağlanılıyor durumları açıkça gösterilmeli.
- Responsive web; klavye kullanımı, focus ve ekran okuyucu etiketleri
  düşünülmeli.

## 4. Teknik mimari

``` mermaid
flowchart LR
    FE[React + TypeScript Web] -->|HTTPS REST / JWT| API[Spring Boot API]
    FE <-->|WSS STOMP / JWT| WS[Spring WebSocket]
    API --> DOMAIN[Keşif Domain Services]
    WS --> DOMAIN
    DOMAIN --> DB[(PostgreSQL)]
    DOMAIN --> AUTH[Identity Provider Port]
    DOMAIN --> BUTIK[Butik Group Port]
    DOMAIN --> Q[Question Bank Port]
    DOMAIN --> OUT[Outbox / Events]
    AUTH -. local mock / future adapter .- EXT[Mevcut Doping Sistemleri]
    BUTIK -. local mock / future adapter .- EXT
    Q -. local mock / future adapter .- EXT
```

**Tercih:** İlk aşamada **modüler monolit**. REST ve WebSocket aynı
Spring Boot uygulamasında. Tek instance ile başlayıp çoklu instance
ihtiyacında broker/Redis ve dağıtık oturum yönetimi ayrıca tasarlanacak.
Gereksiz Kafka, Kubernetes veya mikroservis kurulumu yok.

### 4.1 Backend bağımlılıkları

- Spring Web, Spring WebSocket, Spring Security, OAuth2 Resource Server
  veya güvenilir JWT doğrulama çözümü
- Spring Data JPA, Bean Validation, PostgreSQL Driver, Flyway
- springdoc-openapi Starter WebMVC UI (Swagger)
- Spring Boot Actuator, Micrometer (temel metrikler)
- JUnit 5, Spring Boot Test, Testcontainers PostgreSQL, Mockito
- Lombok **isteğe bağlı**; domain modelinde açıklığı bozacak otomatik
  eşitlik/hashcode kullanılmamalı

### 4.2 Güvenlik / JWT

- Üretimde **mevcut Doping Hafıza kimlik sağlayıcısı** araştırılır;
  varsa JWT issuer, audience, imza/JWKS ve claim sözleşmesiyle
  entegrasyon yapılır.
- **Yerel geliştirme için** `/api/v1/dev/auth/token` yalnızca `local`
  profilinde test kullanıcılarına JWT üretir; production'da kapalıdır.
- Access token kısa ömürlü; refresh token akışı ancak kurumsal auth
  sistemiyle uyumluysa eklenir. Prototipte gerçek üretim token/secret
  kullanılmaz.
- `Authorization: Bearer <token>`; `ROLE_STUDENT`, `ROLE_ADMIN`; CORS
  origin allowlist; HTTPS/WSS.
- WebSocket `CONNECT` sırasında token doğrulanır; **SUBSCRIBE** ve
  **SEND** işlemlerinde oturum katılımcılığı tekrar denetlenir.
- JWT, parola, tam soru cevap anahtarı ve çocuk verileri loglanmaz.

### 4.3 Swagger/OpenAPI

- `GET /swagger-ui/index.html`, `GET /v3/api-docs` (springdoc sürümüne
  göre doğrula).
- Tüm REST endpoint'leri `@Operation`, `@ApiResponse`,
  `@SecurityRequirement`, request/response schema örnekleri ile
  belgelenir.
- Swagger `bearerAuth` security scheme içerir. Dev ortamında test
  edilebilir, production'da erişim politikası ayrıca belirlenir.
- Hata formatı Spring `ProblemDetail` / RFC 9457 uyumlu tek biçimde
  tanımlanır.

### 4.4 Proje dizin yapısı

``` text
doping-kesif/
├── README.md                         # Bu dosya (ana sözleşme)
├── AGENTS.md                         # İsteğe bağlı Codex çalışma kuralları
├── .env.example                      # Sır içermeyen örnek değişkenler
├── .gitignore
├── docker-compose.yml               # PostgreSQL, opsiyonel pgAdmin
├── docs/
│   ├── architecture.md               # Bileşenler, entegrasyon portları
│   ├── database-schema.md            # Tam tablo sözlüğü + indeksler
│   ├── er-diagram.md                 # Mermaid ER diyagramı
│   ├── api-specification.md          # REST request/response/hatalar
│   ├── websocket-protocol.md         # Mesaj şemaları ve güvenlik
│   ├── game-rules-and-state.md       # Durum makineleri ve algoritmalar
│   ├── security-and-privacy.md       # JWT, rol, çocuk verisi
│   ├── test-plan.md                  # Test matrisi
│   ├── decisions.md                  # Onay bekleyen kararlar (ADR)
│   └── openapi.yaml                  # OpenAPI 3.x sözleşmesi
├── backend/
│   ├── pom.xml
│   └── src/
│       ├── main/java/com/doping/kesif/
│       │   ├── KeşifApplication.java
│       │   ├── config/              # Security, JWT, OpenAPI, WebSocket
│       │   ├── auth/                # CurrentUser, identity port
│       │   ├── boutique/            # Butik group port + adapters
│       │   ├── catalog/             # Grade, subject, topic, question
│       │   ├── puzzle/              # Puzzle and collection
│       │   ├── invitation/          # Invite and lobby
│       │   ├── game/                # Session, round, answer, engine
│       │   ├── reward/              # XP ledger
│       │   ├── realtime/            # STOMP controllers/events
│       │   ├── common/              # Error, response, auditing
│       │   └── outbox/              # Optional reliable event relay
│       ├── main/resources/
│       │   ├── application.yml
│       │   ├── application-local.yml
│       │   └── db/migration/        # V1__, V2__ ...
│       └── test/                     # Unit/integration/security tests
└── frontend/
    ├── package.json
    ├── src/
    │   ├── app/                     # Router, providers, auth
    │   ├── components/              # Shared UI / sidebar adapter
    │   ├── features/kesif/         # Pages, components, hooks
    │   ├── features/collection/
    │   ├── services/api/            # Axios/fetch, typed contracts
    │   ├── services/realtime/       # STOMP client, reconnect
    │   ├── types/
    │   └── styles/
    └── tests/
```

## 5. PostgreSQL veri modeli

**Şema:** `public` veya `kesif` şeması; kurumsal DBA ile
kesinleştirilecek. Aşağıdaki `kesif_*` tabloları Keşif'e ait veri
modelidir. Gerçek öğrenci/grup kimlikleri **harici sistem referansı**
olabilir; entegrasyon netleşmeden kurumsal tablolarla FK zorlaması
yapılmaz.

**Tip standartları:** PK `UUID`; zaman `TIMESTAMPTZ`; miktar `INTEGER`;
durum `VARCHAR(32)` + `CHECK` veya uygulama enum eşlemesi; dış ID
`VARCHAR(128)` (kurumsal ID formatı netleşince güncellenir); JSONB
sadece esnek payload/snapshot için. `created_at`, `updated_at`,
gerektiğinde `version BIGINT`.

### 5.1 Tablo envanteri ve kolonlar

| Tablo                      | Önemli kolonlar                                                                                                                                                                                  | Amaç / ilişki                                                                           |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| `kesif_player_profile`    | `id UUID PK`, `external_student_id VARCHAR(128) UNIQUE`, `display_name`, `grade_level SMALLINT`, `created_at`, `updated_at`                                                                      | Öğrenci için asgari Keşif profili; harici kimlik kaynağına referans                    |
| `kesif_subject`           | `id UUID PK`, `code UNIQUE`, `name`, `active`                                                                                                                                                    | Matematik vb. ders kataloğu                                                             |
| `kesif_topic`             | `id UUID PK`, `subject_id FK`, `grade_level SMALLINT`, `code`, `name`, `active`                                                                                                                  | Ders + sınıf düzeyi altında konu                                                        |
| `kesif_question`          | `id UUID PK`, `external_question_id NULL`, `topic_id FK`, `body TEXT`, `difficulty SMALLINT`, `active`, `created_at`                                                                             | Yerel/mock soru veya harici soru bankası referansı; doğru seçenek istemciye gönderilmez |
| `kesif_question_option`   | `id UUID PK`, `question_id FK`, `option_key`, `body TEXT`, `is_correct BOOLEAN`                                                                                                                  | Seçenekler; soru başına tek doğru seçenek kısıtı                                        |
| `kesif_puzzle`            | `id UUID PK`, `code UNIQUE`, `title`, `image_asset_key`, `piece_count DEFAULT 20`, `active`                                                                                                      | Puzzle kataloğu; görselin kullanım hakkı doğrulanmalı                                   |
| `kesif_invitation`        | `id UUID PK`, `inviter_id FK`, `invitee_id FK`, `boutique_group_ref`, `status`, `expires_at`, `created_at`, `responded_at`                                                                       | Aynı grup içi davet ve süresi                                                           |
| `kesif_session`           | `id UUID PK`, `invitation_id FK UNIQUE`, `puzzle_id FK`, `grade_level`, `subject_id FK`, `topic_id FK NULL`, `status`, `current_round_no`, `revealed_count`, `started_at`, `ended_at`, `version` | Tek bir 10 soruluk maçın otoriter durumu                                                |
| `kesif_session_player`    | `id UUID PK`, `session_id FK`, `player_id FK`, `slot SMALLINT`, `ready`, `correct_count`, `connected_at`, `last_seen_at`                                                                         | Maçın iki katılımcısı; `UNIQUE(session_id,player_id)` ve `UNIQUE(session_id,slot)`      |
| `kesif_round`             | `id UUID PK`, `session_id FK`, `round_no SMALLINT`, `question_id FK`, `status`, `opened_at`, `deadline_at`, `closed_at`, `version`                                                               | 1–10 numaralı tur ve seçilmiş soru snapshot/ref                                         |
| `kesif_round_answer`      | `id UUID PK`, `round_id FK`, `player_id FK`, `selected_option_id FK NULL`, `is_correct`, `submitted_at`, `timed_out`                                                                             | Her oyuncu/tur için en çok bir cevap: `UNIQUE(round_id,player_id)`                      |
| `kesif_piece_unlock`      | `id UUID PK`, `session_id FK`, `round_id FK`, `player_id FK`, `piece_index SMALLINT`, `created_at`                                                                                               | Doğru cevapla açılan ortak parçanın değişmez kaydı; `UNIQUE(session_id,piece_index)`    |
| `kesif_player_collection` | `id UUID PK`, `player_id FK`, `puzzle_id FK`, `unlocked_count`, `completed_at`, `version`                                                                                                        | Öğrencinin kalıcı puzzle ilerlemesi; `UNIQUE(player_id,puzzle_id)`                      |
| `kesif_collection_piece`  | `id UUID PK`, `collection_id FK`, `piece_index SMALLINT`, `source_session_id FK`, `created_at`                                                                                                   | Kişisel koleksiyon parçalarının tekil kazanımı; `UNIQUE(collection_id,piece_index)`     |
| `kesif_xp_ledger`         | `id UUID PK`, `player_id FK`, `session_id FK`, `event_type`, `amount`, `dedupe_key UNIQUE`, `created_at`                                                                                         | Append-only XP muhasebesi, tekrar ödül engeli                                           |
| `kesif_outbox_event`      | `id UUID PK`, `aggregate_type`, `aggregate_id`, `event_type`, `payload JSONB`, `created_at`, `published_at`, `attempts`                                                                          | İsteğe bağlı güvenilir olay yayınlama ve retry                                          |

**Not:** Butik grupları Keşif'ın sahip olduğu tablo olarak
**varsayılmıyor**. Yerel geliştirmede `MockBoutiqueGroupGateway`
fixture'ları kullanılacak. Gerekirse yalnızca `local` test şemasında
`mock_boutique_group` ve `mock_boutique_membership` eklenebilir; üretim
domain tablolarına karıştırılmayacak.

### 5.2 ER diyagramı

``` mermaid
erDiagram
    KESIF_PLAYER_PROFILE ||--o{ KESIF_INVITATION : inviter_or_invitee
    KESIF_PLAYER_PROFILE ||--o{ KESIF_SESSION_PLAYER : participates
    KESIF_PLAYER_PROFILE ||--o{ KESIF_PLAYER_COLLECTION : owns
    KESIF_PLAYER_PROFILE ||--o{ KESIF_XP_LEDGER : earns
    KESIF_SUBJECT ||--o{ KESIF_TOPIC : contains
    KESIF_TOPIC ||--o{ KESIF_QUESTION : has
    KESIF_QUESTION ||--o{ KESIF_QUESTION_OPTION : offers
    KESIF_INVITATION ||--o| KESIF_SESSION : creates
    KESIF_PUZZLE ||--o{ KESIF_SESSION : used_by
    KESIF_PUZZLE ||--o{ KESIF_PLAYER_COLLECTION : collected_as
    KESIF_SESSION ||--|{ KESIF_SESSION_PLAYER : includes
    KESIF_SESSION ||--|{ KESIF_ROUND : has
    KESIF_ROUND }o--|| KESIF_QUESTION : asks
    KESIF_ROUND ||--o{ KESIF_ROUND_ANSWER : receives
    KESIF_ROUND ||--o{ KESIF_PIECE_UNLOCK : awards
    KESIF_SESSION ||--o{ KESIF_XP_LEDGER : grants
    KESIF_PLAYER_COLLECTION ||--o{ KESIF_COLLECTION_PIECE : consists_of
```

### 5.3 İndeks ve constraint kuralları

- `kesif_player_profile(external_student_id)` UNIQUE.
- `kesif_topic(subject_id, grade_level, code)` UNIQUE.
- `kesif_question_option(question_id, option_key)` UNIQUE; **tek doğru
  seçenek** için PostgreSQL partial unique index:
  `(question_id) WHERE is_correct = true` (ayrıca en az bir doğru
  seçenek validasyonu).
- `kesif_invitation(invitee_id,status,expires_at)` ve
  `(inviter_id,status)` indeksleri.
- `kesif_session(status,started_at)`,
  `kesif_session_player(player_id,session_id)`.
- `kesif_round(session_id,round_no)` UNIQUE,
  `round_no BETWEEN 1 AND 10`.
- `kesif_round_answer(round_id,player_id)` UNIQUE.
- `kesif_piece_unlock(session_id,piece_index)` UNIQUE,
  `piece_index BETWEEN 1 AND 20`.
- `kesif_player_collection(player_id,puzzle_id)` UNIQUE,
  `unlocked_count BETWEEN 0 AND 20`.
- `kesif_collection_piece(collection_id,piece_index)` UNIQUE.
- `kesif_xp_ledger(dedupe_key)` UNIQUE.
- `kesif_session_player(slot)` değeri 1 veya 2; **tam iki katılımcı**
  uygulama transaction'ı ile garanti edilir.
- Cevaplanan `selected_option_id` değerinin turun `question_id` değerine
  ait olduğu **servis katmanında** doğrulanır (gerekirse DB composite FK
  ile güçlendirilir).
- `version` optimistic locking için `@Version` ile kullanılır; kritik
  tur kapanışlarında row-level locking/atomik update seçimi test edilir.

### 5.4 Migration planı

`V1__create_catalog_and_player_tables.sql` →
`V2__create_invitation_and_session.sql` →
`V3__create_rounds_answers_and_piece_unlocks.sql` →
`V4__create_collections_and_xp.sql` →
`V5__create_outbox_and_indexes.sql` →
`V6__seed_local_reference_catalog.sql` (**seed yalnızca local/test**).

Migration dosyaları idempotent uygulama başlatma beklentisine uygun
olmalı; prod tabloları otomatik drop edilmemeli. Şema değişikliği için
yeni migration oluşturulmalı, geçmiş migration değiştirilmemeli.

## 6. Domain modelleri ve Java sınıfları

| Katman     | Örnek sınıflar                                                                                                                                                   | Sorumluluk                       |
|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------|
| Entity     | `KeşifSession`, `KeşifRound`, `KeşifRoundAnswer`, `KeşifInvitation`, `KeşifPuzzle`, `PlayerCollection`, `XpLedgerEntry`                                     | PostgreSQL/JPA persist modeli    |
| Enum       | `InvitationStatus`, `SessionStatus`, `RoundStatus`, `XpEventType`                                                                                                | Tip güvenli durumlar             |
| DTO        | `CreateInvitationRequest`, `InvitationResponse`, `AnswerRequest`, `RoundViewResponse`, `GameStateResponse`, `GameResultResponse`                                 | REST ve WebSocket dış sözleşmesi |
| Repository | `KeşifSessionRepository`, `KeşifRoundRepository`, `KeşifAnswerRepository`, ...                                                                                | JPA sorguları ve lock işlemleri  |
| Service    | `InvitationService`, `MatchmakingPolicy`, `GameSessionService`, `RoundService`, `AnswerEvaluationService`, `PuzzleService`, `CollectionService`, `RewardService` | İş kuralları ve transaction      |
| Gateway    | `IdentityGateway`, `BoutiqueGroupGateway`, `QuestionBankGateway`, `AssetGateway`                                                                                 | Harici sistem soyutlaması        |
| Adapter    | `MockBoutiqueGroupGateway`, `HttpBoutiqueGroupGateway`                                                                                                           | Yerel ve üretim entegrasyonları  |
| Controller | `InvitationController`, `KeşifSessionController`, `CatalogController`, `CollectionController`                                                                   | HTTP sözleşmesi                  |
| Realtime   | `KeşifStompController`, `KeşifEventPublisher`, `StompAuthorizationInterceptor`                                                                                 | STOMP komutları/olayları         |

**JPA kuralları:** Entity doğrudan API'den dönülmez. DTO mapper
kullanılır. Lazy loading ve N+1 sorgular kontrol edilir.
`@Transactional` servis katmanında tutulur. JPA entity'lerinde `@Data`
kullanımı ve döngüsel `toString()` engellenir.

## 7. Durum makineleri ve oyun algoritması

### 7.1 Davet

`PENDING → ACCEPTED | REJECTED | EXPIRED | CANCELLED`.

- Kabul yalnızca davetli oyuncudan; iptal yalnızca davet sahibinden.
- Davet kabulünde grup üyeliği **yeniden doğrulanır**.
- Süresi geçmiş davet kabul edilemez; yinelenen kabul ikinci maç
  üretmez.

### 7.2 Maç

`WAITING_FOR_PLAYERS → READY → IN_PROGRESS → COMPLETED` veya
`CANCELLED / ABANDONED`.

- Sadece iki katılımcı hazır olunca oyun başlar.
- Maç başladıktan sonra soru listesi değişmez; 10 soru bir defada
  seçilip `kesif_round` içine yazılır.
- Soru havuzunda 10 uygun soru yoksa maç başlamaz; kullanıcıya anlaşılır
  hata verilir.

### 7.3 Tur

`PENDING → OPEN → WAITING_FOR_ANSWERS → REVEAL → CLOSED`.

- Soru ve seçenekler iki oyuncuya sunulur; **doğru cevap işareti
  gösterilmez**.
- Her oyuncunun ilk geçerli cevabı kabul edilir; cevap değiştirilemez.
- İki cevap geldiğinde veya deadline işlendiğinde değerlendirme yapılır.
- Doğru cevap başına bir benzersiz parça açılır. `piece_index` 1–20
  aralığında olmalı; aynı indeks tekrar açılamaz.
- REVEAL ayrı ekran/olaydır; ardından sıradaki tur açılır.
- 10. tur kapanınca tek transaction içinde sonuç ve ödül kayıtları
      tamamlanır.

### 7.4 Cevap yarışı ve idempotency

1.  Kimlik ve maç katılımcılığı kontrol edilir.
2.  `roundId` ve `optionId` geçerliliği doğrulanır.
3.  Transaction başlar; ilgili tur satırı kilitlenir
    (`SELECT ... FOR UPDATE`) **veya** version CAS ile güncellenir.
4.  `UNIQUE(round_id, player_id)` sayesinde tekrar cevap puan
    kazandıramaz; tekrarda önceki sonuç döndürülür veya `409` (karar
    kaydıyla sabitlenir).
5.  İki cevap tamamlandıysa tur **bir kez** kapatılır, parça kayıtları
    atomik yazılır.
6.  İşlem commit edildikten sonra ilgili WebSocket olayları yayınlanır;
    kritik olay kaybı için outbox opsiyonu değerlendirilir.
7.  Tekrarlı completion isteğinde XP `dedupe_key` ile tekrar yazılmaz.

### 7.5 Koleksiyon parçası kazanımı — önerilen deterministik kural

Oturumda açılan `piece_index` değerleri her oyuncunun koleksiyonunda
**henüz bulunmayan parçalara** eşlenir. Örneğin koleksiyonda 1–12 varsa,
yeni kazanılan 6 ortak parça koleksiyona 13–18 olarak eklenir. Bu,
**önerilen MVP kuralıdır**; görseldeki parça kimlikleriyle birebir aynı
kalması isteniyorsa ekip kararı gerekir. Tamamlanmış koleksiyon 20/20'de
kalır; tekrar ödül verilmez.

### 7.6 XP — önerilen, yapılandırılabilir

- Maçı tamamlama: `+20 XP`
- Öğrencinin her doğru cevabı: `+5 XP`
- Koleksiyonu ilk kez tamamlama: `+30 XP`

**KARAR BEKLİYOR:** XP'nin kurumsal ödül sistemine entegrasyonu ve
değerlerin onayı. Puanlar sunucuda hesaplanır; istemciden XP miktarı
alınmaz.

## 8. REST API sözleşmesi (taslak v1)

**Base path:** `/api/v1` \| **Kimlik:** Bearer JWT (local token üretimi
hariç) \| **Body:** JSON.

| Method | Endpoint                                   | Açıklama                                          | Rol       |
|--------|--------------------------------------------|---------------------------------------------------|-----------|
| `GET`  | `/me`                                      | Oturum açmış öğrenci özeti                        | STUDENT   |
| `GET`  | `/catalog/grades`                          | Desteklenen sınıflar                              | STUDENT   |
| `GET`  | `/catalog/subjects?grade=8`                | Sınıfa uygun dersler                              | STUDENT   |
| `GET`  | `/catalog/topics?grade=8&subjectId=...`    | Konular                                           | STUDENT   |
| `GET`  | `/puzzles`                                 | Puzzle görevleri                                  | STUDENT   |
| `GET`  | `/boutique/friends`                        | Uygun aynı grup arkadaşları                       | STUDENT   |
| `POST` | `/invitations`                             | Davet oluştur                                     | STUDENT   |
| `GET`  | `/invitations/incoming`                    | Gelen davetler                                    | STUDENT   |
| `POST` | `/invitations/{id}/accept`                 | Daveti kabul et                                   | STUDENT   |
| `POST` | `/invitations/{id}/reject`                 | Daveti reddet                                     | STUDENT   |
| `POST` | `/invitations/{id}/cancel`                 | Daveti iptal et                                   | STUDENT   |
| `GET`  | `/sessions/{id}`                           | Maçın yetkili oyuncuya göre görünümü              | Katılımcı |
| `POST` | `/sessions/{id}/ready`                     | Oyuncu hazır                                      | Katılımcı |
| `POST` | `/sessions/{id}/rounds/{roundNo}/answers`  | Cevap gönder (otoriter işlem)                     | Katılımcı |
| `POST` | `/sessions/{id}/rounds/{roundNo}/continue` | Reveal sonrası hazır/ilerle                       | Katılımcı |
| `GET`  | `/sessions/{id}/result`                    | Maç sonucu                                        | Katılımcı |
| `GET`  | `/me/collections`                          | Kişisel koleksiyon                                | STUDENT   |
| `GET`  | `/me/history`                              | Oyun geçmişi                                      | STUDENT   |
| `GET`  | `/health`                                  | Health/readiness (actuator üzerinden de olabilir) | Policy    |

**İlkeleri:** `201 Created` oluşturma, `200 OK` okuma, `204 No Content`
uygun eylem, `400` validasyon, `401` kimlik, `403` yetki, `404` görünür
olmayan kaynak, `409` durum çakışması, `422` alan kuralı, `429` rate
limit. Yanıtların tam şeması `docs/openapi.yaml` dosyasında tek
kaynaktan tanımlanacak.

### 8.1 Örnek davet

``` http
POST /api/v1/invitations
Authorization: Bearer <access_token>
Content-Type: application/json
```

``` json
{
  "inviteeId": "8fda913d-735b-4d5a-93f7-449e08a52b4d",
  "puzzleId": "c5ed8418-a046-452e-8c6a-2d89a2a11923",
  "gradeLevel": 8,
  "subjectId": "d4a3c1bd-5ef1-4e10-bbc4-72390a2e762a",
  "topicId": null
}
```

`inviterId` **istemciden alınmaz**; JWT'den bulunur. Backend davetlinin
aynı Butik grubunda olduğunu doğrular.

### 8.2 Örnek cevap

``` http
POST /api/v1/sessions/{sessionId}/rounds/3/answers
Authorization: Bearer <access_token>
Content-Type: application/json
```

``` json
{
  "optionId": "bcbb0378-43e6-4f21-8d31-51224ebac93f"
}
```

Cevap gönderme yanıtı diğer oyuncunun cevabını veya doğru şıkkı **erken
açıklamaz**. Değerlendirme tur kapanınca ayrı olayla duyurulur.

### 8.3 Hata örneği

``` json
{
  "type": "https://example.invalid/problems/kesif-not-participant",
  "title": "Forbidden",
  "status": 403,
  "detail": "Bu oyun oturumuna erişim yetkiniz yok.",
  "instance": "/api/v1/sessions/123",
  "code": "KESIF_NOT_PARTICIPANT"
}
```

`example.invalid` örnek placeholder'dır; production alan adı değildir.

## 9. WebSocket / STOMP tasarımı

- Endpoint: `/ws/kesif` (WSS production).
- Client application prefix: `/app`.
- Öğrenciye özel olaylar: `/user/queue/kesif`.
- Maç olayları: `/topic/kesif/sessions/{sessionId}` **ancak katılımcı
  kontrolü yapılan özel subscription interceptor ile**; daha güvenli
  alternatif: tüm olayları iki oyuncunun user queue'suna göndermek.
- Yetki doğrulaması `CONNECT`, `SUBSCRIBE`, `SEND` üzerinde yapılır.
- Soru gönderimi sırasında `correctOptionId`, `isCorrect` ve doğru cevap
  açıklayıcı metadata asla bulunmaz.

| Sunucu olayı          | Ne zaman                | Önemli alanlar                                                 |
|-----------------------|-------------------------|----------------------------------------------------------------|
| `INVITATION_RECEIVED` | Davet gönderildi        | `invitationId`, `inviterDisplayName`, `expiresAt`              |
| `INVITATION_UPDATED`  | Kabul/ret/iptal         | `invitationId`, `status`                                       |
| `PLAYER_READY`        | Oyuncu hazır            | `sessionId`, `playerId`, `ready`                               |
| `SESSION_STARTED`     | İki oyuncu hazır        | `sessionId`, `totalRounds`                                     |
| `ROUND_STARTED`       | Yeni soru açıldı        | `sessionId`, `roundNo`, `question`, `deadlineAt`               |
| `ANSWER_RECEIVED`     | Bir oyuncu cevap verdi  | `roundNo`, `answeredPlayerSlot` (cevabın içeriği yok)          |
| `ROUND_REVEALED`      | Tur sonuçlandı          | `roundNo`, `playerResults`, `newPieceIndices`, `revealedCount` |
| `NEXT_ROUND_READY`    | Reveal sonrası ilerleme | `roundNo`, `readySlots`                                        |
| `SESSION_COMPLETED`   | 10 tur tamamlandı       | `sessionId`, `resultAvailable`                                 |
| `SESSION_ABORTED`     | İptal/terk              | `reasonCode`                                                   |

**Event envelope:** `eventId`, `eventType`, `sessionId`, `sequence`,
`occurredAt`, `payload`. `sequence` ile istemci eski/çift olayı
atlayabilir. Yeniden bağlantıda `GET /sessions/{id}` otoriter
snapshot'tır; WebSocket tek veri kaynağı değildir.

``` json
{
  "eventId": "ba905839-c64d-45c0-83d9-15c13d5fbba2",
  "eventType": "ROUND_REVEALED",
  "sessionId": "7e1dcf5e-b3a9-4a6b-8a83-17936c343917",
  "sequence": 12,
  "occurredAt": "2026-10-21T12:00:00Z",
  "payload": {
    "roundNo": 3,
    "newPieceIndices": [5, 6],
    "revealedCount": 6
  }
}
```

## 10. Frontend uygulama planı

### 10.1 Teknolojiler

React + TypeScript + Vite + Tailwind CSS; React Router; TanStack Query
(öneri); Axios veya fetch; `@stomp/stompjs` ve SockJS yalnızca sunucu
altyapısı gerektiriyorsa; React Hook Form + Zod (formlar için öneri);
Vitest + React Testing Library; Playwright (E2E).

### 10.2 Veri akışı

- REST: katalog, davet listesi, oturum snapshot, geçmiş, koleksiyon.
- WebSocket: davet ve maç içi anlık olaylar.
- Tekil otorite **backend**; frontend cevap doğrulamaz, XP hesaplamaz,
  parça indeksini belirlemez.
- `useKeşifSession(sessionId)` hook'u REST snapshot + WebSocket event
  reducer kullanır.
- Refresh/reconnect sonrası REST snapshot alınır, ardından olay
  aboneliği yenilenir.
- Token tercihen mevcut Doping Hafıza auth katmanından; demo modda
  yalnızca local token. Üretim token'ı localStorage'da saklama kararı
  güvenlik ekibiyle değerlendirilir; XSS riskine dikkat.

### 10.3 Bileşenler

`KeşifLayout`, `KeşifSidebarAdapter`, `PuzzleCard`, `SubjectSelector`,
`TopicSelector`, `FriendCard`, `InvitationDialog`, `LobbyPlayerCard`,
`QuestionCard`, `AnswerOption`, `AnswerStatus`, `PuzzleGrid20`,
`RevealAnimation`, `ResultSummary`, `CollectionProgress`,
`ConnectionBanner`, `ErrorState`.

### 10.4 Frontend kabul testleri

- İki tarayıcı penceresi/iki test öğrencisiyle canlı maç oynanır.
- Davet ve kabul anlık görünür; geçersiz davet hata gösterir.
- Aynı sorunun iki oyuncuda aynı sıra ile açıldığı doğrulanır.
- Cevap sonrası butonlar kilitlenir; bekleme durumunda doğru şık
  açıklanmaz.
- Tur kapandığında ayrı puzzle ekranı açılır; 0/1/2 parça doğru işlenir.
- Refresh/reconnect sonrası aynı oturum korunur.
- Sonuç ve kişisel koleksiyon yeniden yüklemede aynı kalır.

## 11. Güvenlik, çocuk verileri ve gizlilik

- Kişisel veriler asgari tutulur; gerçek ad yerine izin verilen display
  name kullanılabilir.
- Gruplar ve katılımcılık sunucuda kontrol edilir; istemci `groupId`
  veya `playerId` değiştirerek yetki kazanamaz.
- IDOR testleri: `/sessions/{otherSessionId}`, koleksiyon, davet ve
  WebSocket topic erişimi.
- Soruların doğru cevapları yalnızca tur sonuçlandığında açıklanır; JPA
  entity serialization kullanılmaz.
- SQL injection için parametreli sorgular/JPA; XSS için React varsayılan
  escaping ve güvenilir asset URL'leri.
- Davet spam'i, answer spam'i, brute-force ve WS connection flood için
  rate limit.
- API/WS bağlantılarında TLS; loglarda token ve gereksiz öğrenci verisi
  maskelenir.
- Silme, saklama süresi, veli onayı ve KVKK gereklilikleri kurumun
  hukuk/güvenlik ekibiyle netleştirilmeli.
- JWT secret'ları `.env` / secrets manager üzerinden; `.env`, private
  key ve gerçek token Git'e eklenmez.

## 12. Test stratejisi

| Katman         | Araç                      | Zorunlu test                                                 |
|----------------|---------------------------|--------------------------------------------------------------|
| Unit           | JUnit 5, Mockito          | 0/1/2 parça, 10 tur, XP, koleksiyon, durum geçişleri         |
| DB Integration | Testcontainers PostgreSQL | Flyway, unique constraint, FK, transaction rollback          |
| Security       | Spring Security Test      | 401/403, grup dışı davet, başkasının maçına erişim           |
| Concurrency    | Integration tests         | İki eşzamanlı cevap, tekrar cevap, çift ödül, timeout yarışı |
| REST Contract  | MockMvc + OpenAPI         | DTO validasyonu, hata formatı, endpoint sözleşmesi           |
| WebSocket      | STOMP integration         | Yetkisiz subscribe, duplicate event, reconnect               |
| Frontend       | Vitest/RTL                | Durum ekranları, puzzle parçası sayısı, error states         |
| E2E            | Playwright                | İki öğrencinin 10 soruluk uçtan uca oyunu                    |

**Test matrisi örnekleri:** İki doğru=2; tek doğru=1; iki yanlış=0; 10
turdan sonra 11. tur yok; 20 üstü parça yok; çift submit tek kayıt;
davetli olmayan kabul edemez; expired davet kabul edilmez; yetersiz soru
havuzu; oyuncu kopması; son turda iki concurrent cevap; aynı XP event
iki kere işlenemez.

## 13. DevOps ve yerel çalıştırma

- `docker compose up -d` ile PostgreSQL başlar.
- `backend`: `./mvnw spring-boot:run -Dspring-boot.run.profiles=local`
  (wrapper eklendikten sonra).
- `frontend`: `npm install && npm run dev`.
- `.env.example`: `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`,
  `SPRING_DATASOURCE_URL`, `JWT_ISSUER`, `JWT_AUDIENCE`, `JWT_JWKS_URI`
  (prod), `VITE_API_BASE_URL`, `VITE_WS_URL`.
- `application-local.yml` local mock'ları etkinleştirir; prod adapter
  seçimi environment üzerinden yapılır.
- CI: backend test + frontend lint/typecheck/test + migration smoke
  test + secret scan.
- Health: Actuator liveness/readiness; temel metrikler: aktif maç, tur
  tamamlama süresi, WebSocket reconnect, hata oranı.

## 14. Dokümantasyon çıktıları

Kodlamaya başlamadan önce Codex şu dosyaları tamamlamalı ve bu ana
dosyayla tutarlı tutmalı:

- `docs/architecture.md`: context, modül sınırları, gerçek/mock
  entegrasyon.
- `docs/database-schema.md`: **her tablonun her kolonu**, tipi,
  nullability, default, PK/FK, unique/index, migration referansı.
- `docs/er-diagram.md`: Mermaid ER + ilişki açıklaması.
- `docs/api-specification.md` ve `docs/openapi.yaml`: tüm endpoint'lerin
  request/response/error şemaları, JWT scheme.
- `docs/websocket-protocol.md`: mesaj envelope, event sırası, auth ve
  reconnect.
- `docs/game-rules-and-state.md`: state machine, timeout, idempotency,
  ödül algoritmaları.
- `docs/security-and-privacy.md`: yetki matrisi, veri saklama ve çocuk
  güvenliği.
- `docs/test-plan.md`: kabul kriteri → test eşlemesi.
- `docs/decisions.md`: açık kararlar, seçilen çözüm, gerekçe, tarih.

## 15. Geliştirme aşamaları — Codex görev listesi

### Faz 0 — Repository analizi ve teknik plan

- [ ] Zip örneğini **yalnızca doküman yapısı açısından** incele; Keşif
  iş kurallarını Pulse'tan kopyalama.
- [ ] Var olan kod/asset/Figma varsa envanterini çıkar; kullanıcı
  dosyalarını koru.
- [ ] Bu README ile teknik tasarım belgesindeki kararları karşılaştır;
  farkları `docs/decisions.md` içinde listele.
- [ ] Java/Spring Boot ve Node/React sürümlerini sabitle.
- [ ] Uygulama modüllerini ve bağımlılıklarını tanımla.
- [ ] **Çıktı:** Repo ağacı, ADR listesi, uygulanabilir iş paketleri.

### Faz 1 — Ayrıntılı teknik dokümantasyon (henüz kod yok)

- [ ] `docs/architecture.md` ve entegrasyon portlarını yaz.
- [ ] 16 tablo için tam kolon sözlüğü, FK, indeks, constraints çıkar.
- [ ] ER diyagramını oluştur.
- [ ] REST ve WebSocket sözleşmelerini tanımla.
- [ ] Swagger OpenAPI YAML dosyasını hazırla.
- [ ] Oyun state machine, concurrency ve idempotency kurallarını yaz.
- [ ] Güvenlik ve test matrisini yaz.
- [ ] **Çıktı:** Ekip incelemesine hazır docs paketi; kodlamadan önce
  onay noktası.

### Faz 2 — Proje iskeleti ve local altyapı

- [ ] Spring Initializr tabanlı Maven projesi, Java 21, Spring Boot 3.x
  oluştur.
- [ ] React + TypeScript + Vite + Tailwind kur.
- [ ] Docker Compose PostgreSQL ve `.env.example` ekle.
- [ ] Local/prod profile ayrımı ve `.gitignore` yapılandır.
- [ ] Spring Security, JWT, Swagger UI, Flyway, Actuator
  bağımlılıklarını ekle.
- [ ] `/actuator/health` ve Swagger erişimini doğrula.
- [ ] **Çıktı:** Backend/frontend ayrı çalışır, DB bağlantısı
  doğrulanır.

### Faz 3 — PostgreSQL migration ve JPA modelleri

- [ ] Flyway V1–V5 migration'larını yaz.
- [ ] PK/FK, unique, check, index ve version kolonlarını oluştur.
- [ ] Entity, enum, repository ve DTO mapper katmanını yaz.
- [ ] Local seed: 2–5 öğrenci, Butik mock grup, 4 puzzle, 8. sınıf
  matematikte en az 10 soru.
- [ ] Testcontainers ile migration ve constraint testlerini çalıştır.
- [ ] **Çıktı:** Şema ve modeller çalışır; Swagger iş API'leri henüz boş
  olabilir.

### Faz 4 — Auth, katalog ve Butik entegrasyon soyutlaması

- [ ] Local JWT test kimlikleri, prod identity port ve
  SecurityFilterChain oluştur.
- [ ] `@PreAuthorize` / service authorization kontrollerini yaz.
- [ ] `BoutiqueGroupGateway` mock adapter ve gelecekteki HTTP adapter
  arayüzünü yaz.
- [ ] Katalog, puzzle, arkadaş listesi REST API'lerini tamamla.
- [ ] Swagger bearer auth ve 401/403 testleri ekle.
- [ ] **Çıktı:** Güvenli katalog ve grup arkadaşlığı doğrulaması.

### Faz 5 — Davet ve lobi backend'i

- [ ] Davet oluştur/kabul/ret/iptal akışlarını yaz.
- [ ] Davet expiry ve grup üyeliğini tekrar doğrula.
- [ ] İki oyunculu session ve ready durumlarını oluştur.
- [ ] Tekrar kabul / concurrent accept yarışlarını test et.
- [ ] **Çıktı:** İki öğrenci lobiye kadar ulaşır.

### Faz 6 — Oyun motoru ve REST cevap akışı

- [ ] Sınıf/ders/konuya uygun 10 soruyu seç ve kalıcı turlara yaz.
- [ ] Soru DTO'sundan doğru cevap bilgisini çıkar.
- [ ] Cevap gönderme ve doğrulama iş mantığını yaz.
- [ ] Tur kapanışını transaction ve unique constraint ile koru.
- [ ] 0/1/2 parça algoritmasını uygula; 20 parça sınırını koru.
- ☐
  10. turda session'ı bitir; result endpoint'ini yaz.
- [ ] Concurrency ve idempotency testlerini geçir.
- [ ] **Çıktı:** WebSocket olmadan bile iki öğrenci REST ile tam maç
  oynayabilir.

### Faz 7 — WebSocket gerçek zamanlı senkronizasyon

- [ ] STOMP endpoint ve auth interceptor oluştur.
- [ ] Davet, lobi, tur, reveal ve tamamlanma olaylarını yayınla.
- [ ] `eventId`/`sequence` ve reconnect snapshot akışını uygula.
- [ ] Yetkisiz topic aboneliğini engelle.
- [ ] Olayların commit sonrası yayınlanmasını sağla; outbox ihtiyacını
  değerlendir.
- [ ] **Çıktı:** İki istemcide canlı senkronizasyon.

### Faz 8 — Koleksiyon, XP ve geçmiş

- [ ] Oyundaki ortak parçaları kalıcı kişisel koleksiyona yansıt.
- [ ] Koleksiyonun 20/20 sınırını ve ilk tamamlanma ödülünü uygula.
- [ ] XP ledger ve benzersiz `dedupe_key` ekle.
- [ ] Geçmiş, sonuç ve koleksiyon endpoint'lerini tamamla.
- [ ] Tekrar işleme/ödül testlerini geçir.
- [ ] **Çıktı:** Kalıcı kişisel ilerleme.

### Faz 9 — Frontend ekranları ve entegrasyon

- [ ] Doping Hafıza beyaz tema, sidebar, layout ve routing'i kur.
- [ ] Puzzle/ders/konu/arkadaş seçim ekranlarını geliştir.
- [ ] Davet ve lobi ekranlarını REST + WS ile bağla.
- [ ] Soru ekranı, cevap bekleme, ayrı puzzle reveal ekranını geliştir.
- [ ] Sonuç, koleksiyon ve geçmiş ekranlarını tamamla.
- [ ] Loading/error/reconnect ve responsive davranışları ekle.
- [ ] **Çıktı:** İki tarayıcıda uçtan uca oyun.

### Faz 10 — Kalite, teslim ve demo

- [ ] Unit, integration, security, concurrency ve E2E testlerini
  çalıştır.
- [ ] Swagger/OpenAPI ile endpoint'leri karşılaştır.
- [ ] README kurulum adımlarını gerçek komutlarla doğrula.
- [ ] Secret scan, CORS, rate limit, logging ve health kontrolü yap.
- [ ] Demo senaryosu: iki öğrenci, 10 soru, 0/1/2 parça, koleksiyon/XP.
- [ ] Açık riskler ve üretim entegrasyonu için yapılacakları raporla.
- [ ] **Çıktı:** Tek komutla yerel çalışabilen, test edilmiş MVP.

## 16. Definition of Done (DoD)

Bir özellik ancak şu koşullarda **tamamlandı** sayılır:

1.  Gereksinim ve kabul kriteri açık.
2.  Kod proje standartlarına uygun, mevcut yapıyı bozmuyor.
3.  DB değişikliği varsa Flyway migration ve test var.
4.  REST değişikliği varsa Swagger/OpenAPI güncel.
5.  WebSocket değişikliği varsa olay sözleşmesi güncel.
6.  Yetki ve negatif senaryo testleri yazılmış.
7.  İlgili unit/integration testleri çalışmış; sonuç raporlanmış.
8.  Frontend kullanıyorsa loading/error/reconnect durumları ele alınmış.
9.  Gerçek entegrasyon yerine mock kullanılıyorsa açıkça belirtilmiş.
10. README ve docs güncel.

## 17. Ekip onayı gerektiren açık kararlar

| Kod    | Konu                               | Varsayılan/öneri                           | Onay sahibi         |
|--------|------------------------------------|--------------------------------------------|---------------------|
| ADR-01 | Doping kimlik sağlayıcısı          | Harici JWT/JWKS adapter; local mock        | Platform / güvenlik |
| ADR-02 | Doping Butik grup üyeliği          | Port/adapter, local fixture                | Butik backend       |
| ADR-03 | Soru bankası entegrasyonu          | QuestionBankGateway; local soru seed       | İçerik / backend    |
| ADR-04 | Soru süresi                        | Konfigüre edilebilir, değer **belirsiz**   | Ürün                |
| ADR-05 | Oyuncu kopması / terk              | Grace period + snapshot, süre **belirsiz** | Ürün / backend      |
| ADR-06 | Reveal'dan sonraki geçiş           | İki oyuncu hazır veya timeout              | Ürün                |
| ADR-07 | Koleksiyon parçası eşleme          | Deterministik sıradaki eksik parça         | Ürün / tasarım      |
| ADR-08 | XP miktarları                      | 20 tamamla, 5/doğru, 30 koleksiyon         | Ürün                |
| ADR-09 | Puzzle görseli ve kullanım hakkı   | Onaylı kurum asset'leri                    | Tasarım / hukuk     |
| ADR-10 | Kişisel veri saklama süresi        | Kurumsal KVKK politikası                   | Hukuk / güvenlik    |
| ADR-11 | Çoklu instance / WS broker         | MVP tek instance                           | Altyapı             |
| ADR-12 | Mevcut Doping sidebar entegrasyonu | Mevcut bileşeni yeniden kullan             | Frontend            |
| ADR-13 | 5–8. sınıf soru kapsamı            | MVP demo 8. sınıf matematik, genişleme 5–8 | Ürün / içerik       |
| ADR-14 | Timeout cevap değerlendirmesi      | Cevapsız = yanlış (öneri)                  | Ürün                |

## 18. Codex'e verilecek ilk görev metni

> Bu repository'de Doping Keşif projesini geliştireceğiz. Önce kökteki
> `README.md` dosyasını baştan sona oku. `docs/` altında mevcut
> dokümanlar varsa incele. Henüz uygulama kodu yazma. Sadece **Faz 0 ve
> Faz 1** işlerini gerçekleştir: mimari plan, tam PostgreSQL tablo
> sözlüğü ve indeksler, Mermaid ER diyagramı, Java entity/DTO
> taslakları, REST API sözleşmeleri, Swagger OpenAPI YAML, WebSocket
> protokolü, durum makineleri, JWT yetkilendirme matrisi ve test planı.
> Her kararı README ile tutarlı tut. Mevcut Doping Hafıza servislerinin
> API'lerini uydurma; adapter arayüzü ve mock planı hazırla.
> Tamamlayınca oluşturduğun dosyaları ve ekip onayı gerektiren soruları
> özetle; **Faz 2'ye geçmeden onayımı bekle.**

------------------------------------------------------------------------

**Belge notu:** Bu dosya, uygulamaya yön veren ayrıntılı ana README'dir.
Kesin kurum entegrasyonları ve ürün kararları henüz doğrulanmadığı için
ilgili maddeler açık karar olarak tutulmuştur. Önceki teknik tasarım
belgesindeki **16 çekirdek Keşif tablosu** esas alınmıştır; ek yerel
test tabloları veya yeni altyapı gereksinimleri ayrı ADR ile
eklenebilir.
