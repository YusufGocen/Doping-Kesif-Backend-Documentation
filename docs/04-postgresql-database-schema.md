# 4. PostgreSQL veri modeli ve tablo sözlüğü

Şema önerisi: kesif. PK olarak UUID, created_at/updated_at için timestamptz (UTC), durumlar için PostgreSQL CHECK veya varchar + uygulama enum önerilir. Harici kullanıcı ve grup ID türü mevcut sistemle uyumlu seçilmelidir; aşağıda örnek olarak UUID kullanılmıştır.

## 4.1 Tablo envanteri

| **Tablo**                     | **Amaç**                                    | **Kardinalite / not**    |
|-------------------------------|---------------------------------------------|--------------------------|
| kesif_player_profile         | Keşif oyuncu referansı, XP ve metadata     | 1 öğrenci / 1 kayıt      |
| kesif_group_membership_cache | Prototip veya kısa süreli üyelik snapshot’ı | opsiyonel; otorite değil |
| kesif_subject                | Ders kataloğu                               | ders                     |
| kesif_topic                  | Ders/konu kataloğu                          | konu                     |
| kesif_question               | Yerel soru havuzu / snapshot kaynağı        | prototip veya adaptör    |
| kesif_question_option        | Soru seçenekleri                            | soruya N seçenek         |
| kesif_puzzle                 | Puzzle teması ve 20 parça asset’i           | puzzle                   |
| kesif_invitation             | Arkadaş daveti ve süresi                    | davet                    |
| kesif_session                | Maç ve durum / puzzle açılımı               | maç                      |
| kesif_session_player         | Maç oyuncusu, ready ve skor                 | maç başına tam 2         |
| kesif_round                  | 1–10 tur, soru snapshot’ı ve zamanları      | maç başına 10            |
| kesif_round_answer           | Oyuncu cevabı, doğruluk, idempotency        | tur başına max 2         |
| kesif_piece_unlock           | Açılan puzzle parçası kayıtları             | maçta max 20             |
| kesif_player_collection      | Oyuncu bazında puzzle ilerlemesi            | oyuncu × puzzle          |
| kesif_collection_piece       | Koleksiyonda kalıcı kazanılan parça         | oyuncu × puzzle × parça  |
| kesif_xp_ledger              | XP hareketleri / mükerrer önleme            | ödül olayı               |
| kesif_outbox_event           | Commit sonrası güvenilir olay yayını        | olay kuyruğu             |

## 4.2 Temel tablo kolonları

### kesif_player_profile

| **Kolon**               | **Tip / kısıt**               | **Açıklama**                       |
|-------------------------|-------------------------------|------------------------------------|
| id                      | uuid PK                       | Keşif iç ID                       |
| external_user_id        | varchar(128) UNIQUE NOT NULL  | Doping userId                      |
| display_name            | varchar(100) NULL             | Gösterim; gerekmedikçe cache yapma |
| xp_total                | integer DEFAULT 0 CHECK \>= 0 | Türetilmiş toplam                  |
| created_at / updated_at | timestamptz NOT NULL          | Denetim                            |

### kesif_subject

| **Kolon** | **Tip / kısıt**      | **Açıklama**     |
|-----------|----------------------|------------------|
| id        | uuid PK              | Ders             |
| code      | varchar(32) UNIQUE   | MATH, TURKISH... |
| name      | varchar(100)         | Görünen ad       |
| active    | boolean DEFAULT true | Seçilebilirlik   |

### kesif_topic

| **Kolon**  | **Tip / kısıt**     | **Açıklama** |
|------------|---------------------|--------------|
| id         | uuid PK             | Konu         |
| subject_id | uuid FK → subject   | Ders         |
| grade      | smallint CHECK 5..8 | Sınıf        |
| code       | varchar(64)         | Konu kodu    |
| name       | varchar(150)        | Konu adı     |
| active     | boolean             | Aktiflik     |

### kesif_question

| **Kolon**            | **Tip / kısıt**   | **Açıklama**                |
|----------------------|-------------------|-----------------------------|
| id                   | uuid PK           | Soru                        |
| external_question_id | varchar(128) NULL | Soru bankası ID             |
| topic_id             | uuid FK → topic   | Konu                        |
| grade                | smallint          | Sınıf                       |
| body_json            | jsonb NOT NULL    | Soru metni, medya           |
| difficulty_tag       | varchar(20) NULL  | İç seçim; UI’da gösterilmez |
| status               | varchar(20)       | PUBLISHED/ARCHIVED          |
| version              | integer           | İçerik sürümü               |

### kesif_question_option

| **Kolon**   | **Tip / kısıt**          | **Açıklama**      |
|-------------|--------------------------|-------------------|
| id          | uuid PK                  | Seçenek           |
| question_id | uuid FK → question       | Soru              |
| option_key  | varchar(8)               | A/B/C/D vb.       |
| body_json   | jsonb                    | Seçenek içerik    |
| is_correct  | boolean NOT NULL         | Yalnızca sunucuda |
| UNIQUE      | (question_id,option_key) | Tekrarsız seçenek |

### kesif_puzzle

| **Kolon**   | **Tip / kısıt**               | **Açıklama**      |
|-------------|-------------------------------|-------------------|
| id          | uuid PK                       | Puzzle            |
| code        | varchar(64) UNIQUE            | SPACE, HISTORY... |
| title       | varchar(120)                  | Başlık            |
| asset_key   | varchar(255)                  | CDN anahtarı      |
| piece_count | smallint DEFAULT 20 CHECK =20 | MVP 20            |
| active      | boolean                       | Yayın durumu      |

### kesif_invitation

| **Kolon**                 | **Tip / kısıt**          | **Açıklama**         |
|---------------------------|--------------------------|----------------------|
| id                        | uuid PK                  | Davet                |
| inviter_id / invitee_id   | uuid FK → player_profile | İki farklı öğrenci   |
| group_external_id         | varchar(128)             | Butik grup referansı |
| puzzle_id / topic_id      | uuid FK                  | Seçimler             |
| status                    | varchar(20)              | PENDING/ACCEPTED/... |
| expires_at                | timestamptz              | Davet süresi         |
| session_id                | uuid FK NULL             | Kabul edilince maç   |
| created_at / responded_at | timestamptz              | Zamanlar             |

### kesif_session

| **Kolon**                | **Tip / kısıt**      | **Açıklama**        |
|--------------------------|----------------------|---------------------|
| id                       | uuid PK              | Maç                 |
| invitation_id            | uuid UNIQUE FK       | Kaynak davet        |
| group_external_id        | varchar(128)         | Yetki sınırı        |
| puzzle_id / topic_id     | uuid FK              | Seçimler            |
| status                   | varchar(24)          | WAITING.../FINISHED |
| current_round            | smallint CHECK 0..10 | Aktif sıra          |
| unlocked_count           | smallint CHECK 0..20 | Ortak ilerleme      |
| piece_order              | jsonb NOT NULL       | 1..20 karışık sıra  |
| started_at / finished_at | timestamptz NULL     | Zamanlar            |
| version                  | bigint NOT NULL      | Optimistic lock     |

### kesif_session_player

| **Kolon**                      | **Tip / kısıt**        | **Açıklama** |
|--------------------------------|------------------------|--------------|
| id                             | uuid PK                | Katılım      |
| session_id                     | uuid FK                | Maç          |
| player_id                      | uuid FK                | Oyuncu       |
| ready                          | boolean                | Hazır        |
| correct_count                  | smallint DEFAULT 0     | Doğru sayısı |
| connected_at / disconnected_at | timestamptz            | Bağlantı     |
| UNIQUE                         | (session_id,player_id) | Tek katılım  |

### kesif_round

| **Kolon**              | **Tip / kısıt**       | **Açıklama**                     |
|------------------------|-----------------------|----------------------------------|
| id                     | uuid PK               | Tur                              |
| session_id             | uuid FK               | Maç                              |
| round_no               | smallint CHECK 1..10  | Tur sırası                       |
| question_external_id   | varchar(128) NULL     | Kaynak soru                      |
| question_snapshot      | jsonb NOT NULL        | Görünen soru + seçenekler        |
| correct_option_key     | varchar(8) NOT NULL   | Gizli doğru cevap                |
| status                 | varchar(20)           | PENDING/QUESTION_OPEN/REVEAL/... |
| opened_at / expires_at | timestamptz           | Zaman                            |
| resolved_at            | timestamptz NULL      | Kapanış                          |
| UNIQUE                 | (session_id,round_no) | 10 benzersiz tur                 |

### kesif_round_answer

| **Kolon**         | **Tip / kısıt**      | **Açıklama**          |
|-------------------|----------------------|-----------------------|
| id                | uuid PK              | Yanıt                 |
| round_id          | uuid FK              | Tur                   |
| player_id         | uuid FK              | Oyuncu                |
| option_key        | varchar(8) NULL      | Boş/timeout için null |
| is_correct        | boolean NOT NULL     | Sunucu hesaplar       |
| source            | varchar(12)          | SUBMITTED/TIMEOUT     |
| client_request_id | uuid NULL            | İdempotency           |
| answered_at       | timestamptz          | Zaman                 |
| UNIQUE            | (round_id,player_id) | Bir yanıt/oyuncu/tur  |

### kesif_piece_unlock

| **Kolon**             | **Tip / kısıt**          | **Açıklama**       |
|-----------------------|--------------------------|--------------------|
| id                    | uuid PK                  | Açılma             |
| session_id / round_id | uuid FK                  | Kaynak maç/tur     |
| piece_index           | smallint CHECK 1..20     | Parça numarası     |
| earned_by_player_id   | uuid FK                  | Doğru cevap sahibi |
| created_at            | timestamptz              | Zaman              |
| UNIQUE                | (session_id,piece_index) | Bir parça bir kez  |

### kesif_player_collection

| **Kolon**             | **Tip / kısıt**       | **Açıklama**   |
|-----------------------|-----------------------|----------------|
| id                    | uuid PK               | Koleksiyon     |
| player_id / puzzle_id | uuid FK               | Oyuncu/puzzle  |
| unlocked_count        | smallint CHECK 0..20  | Kişisel toplam |
| completed_at          | timestamptz NULL      | Tamamlanma     |
| version               | bigint                | Eşzamanlılık   |
| UNIQUE                | (player_id,puzzle_id) | Tek koleksiyon |

### kesif_collection_piece

| **Kolon**         | **Tip / kısıt**             | **Açıklama**       |
|-------------------|-----------------------------|--------------------|
| id                | uuid PK                     | Parça kazanımı     |
| collection_id     | uuid FK                     | Koleksiyon         |
| piece_index       | smallint CHECK 1..20        | Kalıcı parça       |
| source_session_id | uuid FK                     | Kaynak maç         |
| UNIQUE            | (collection_id,piece_index) | Mükerrer parça yok |

### kesif_xp_ledger

| **Kolon**  | **Tip / kısıt**     | **Açıklama**              |
|------------|---------------------|---------------------------|
| id         | uuid PK             | XP hareketi               |
| player_id  | uuid FK             | Öğrenci                   |
| session_id | uuid FK NULL        | Kaynak maç                |
| event_type | varchar(40)         | GAME_COMPLETE/CORRECT/... |
| points     | integer CHECK \>0   | XP                        |
| dedupe_key | varchar(200) UNIQUE | Mükerrer önleme           |
| created_at | timestamptz         | Zaman                     |

### kesif_outbox_event

| **Kolon**                     | **Tip / kısıt**    | **Açıklama**       |
|-------------------------------|--------------------|--------------------|
| id                            | uuid PK            | Olay               |
| aggregate_type / aggregate_id | varchar(40) / uuid | Maç veya davet     |
| event_type                    | varchar(80)        | Event adı          |
| payload                       | jsonb NOT NULL     | Yayın içeriği      |
| created_at / published_at     | timestamptz        | Olay yaşam döngüsü |
| retry_count                   | integer DEFAULT 0  | Yeniden deneme     |

## 4.3 İndeks, kısıt ve veri bütünlüğü

- kesif_invitation(invitee_id,status,expires_at) ve (inviter_id,status,created_at DESC) indeksleri.

- kesif_session(status,updated_at), kesif_session_player(player_id,session_id), kesif_round(session_id,round_no) indeksleri.

- kesif_round_answer için UNIQUE(round_id,player_id); idempotency için client_request_id + oyuncu bağlamında unique index.

- kesif_piece_unlock için UNIQUE(session_id,piece_index); koleksiyon parçası için UNIQUE(collection_id,piece_index).

- kesif_question_option: her soruda tek doğru cevap için filtered UNIQUE(question_id) WHERE is_correct = true; seçenek sayısı servis seviyesinde doğrulanır.

- Oyun başına tam iki oyuncu ve tam 10 tur, yalnız SQL CHECK ile garanti edilemez; oyun başlatma transaction’ında servis invariant kontrolü gerekir.

- Dış kullanıcı ID’si kimlik servisinde UUID değilse kolon tipi buna göre değiştirilir; foreign key dış veritabanına kurulmaz.

## 4.4 Koleksiyon parçası kazanım politikası (öneri)

Maç içi parça açılımı 1–20 arasından belirlenen benzersiz indekslerdir. Kişisel koleksiyona yazarken aynı parça önceden varsa tekrar eklenmez. Böylece örneğin oyuncunun 12 parçası varken bu maç 6 parça açılması kişisel koleksiyonun mutlaka 18 olacağı anlamına gelmez; çakışma olabilir. Ürün ekibi “her maç +6 ilerleme” istiyorsa kişisel koleksiyon için eksik parçaları seçen ayrı algoritma tanımlanmalıdır. Önceki örnekteki toplama kuralı ile benzersiz parça kuralı arasında bu karar netleştirilmelidir.

*KARAR / NOT: MVP için önerim: kişisel koleksiyonda kazanılmamış parçalardan sırasıyla seçim yaparak her oyun ödülünü mümkün olduğunca yeni parçalara dönüştürmek; 20/20 sınırında durmak. Böylece “12 + 6 = 18” beklentisi korunur ve duplicate oluşmaz.*

## 4.5 Doping Keşif oda genişletmesi (yeni)

Orijinal şema **17 tablo** içerir. Kod/link özelliği için önerilen **2 yeni tablo** ile toplam **19 tablo** olur:

| Yeni tablo | Görevi | Detay |
|---|---|---|
| `kesif_room` | Oda sahibi, ayarlar, katılım kodu/linki hash'leri, durum, süre ve kapasite | [16](16-rooms-codes-and-invite-links.md) |
| `kesif_room_join_request` | Katılma isteği, yöntem, onay/red ve audit | [16](16-rooms-codes-and-invite-links.md) |

**İlişki:** `kesif_room` (1) → (0..1) `kesif_session`; `kesif_room` (1) → (N) `kesif_room_join_request`; `kesif_room` → `kesif_player_profile` (host). `kesif_session.invitation_id` **NULL olabilir**, çünkü kod/link ile oluşturulan oyunda Butik daveti yoktur. `kesif_session.room_id` ise yeni akışta `NOT NULL` önerilir; eski davetler de oda üzerinden geçirilmelidir.

**Önemli:** Mevcut dosyadaki `kesif_invitation` Butik davetlerine özgüdür. Oda kodu ve link token'ını bu tablodaki `invitee_id` alanına sıkıştırmayın. Kod/link katılımı `kesif_room_join_request` üzerinden yönetilir. Kolon sözlüğü ve SQL örneği [16](16-rooms-codes-and-invite-links.md) / [19](19-room-sql-and-er-diagram.md) içindedir.
