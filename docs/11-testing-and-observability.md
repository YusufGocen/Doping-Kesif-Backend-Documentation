# 11. Hata senaryoları, test ve performans

## 11.1 İşlevsel test matrisi

| **Test**                       | **Beklenen sonuç**                  |
|--------------------------------|-------------------------------------|
| İki doğru yanıt                | 2 parça açılır; 2 doğru kaydedilir  |
| Bir doğru / bir yanlış         | 1 parça açılır                      |
| İki yanlış                     | 0 parça açılır                      |
| Yanıt gelmeden süre dolması    | TIMEOUT; tekrar yanıt kabul edilmez |
| Aynı yanıt iki kez             | Tek yanıt/tek puan                  |
| İki eşzamanlı yanıt            | Tur bir kez resolve edilir          |
| Butik yolunda grup dışı davet                | 403, davet oluşturulmaz             |
| Davet çift kabul               | Tek session oluşturulur             |
| 11\. soru isteği               | Oyun 10. turda biter                |
| 21\. parça açma                | CHECK ve servis kuralı engeller     |
| Maç tekrar finish              | XP ve koleksiyon tekrar artmaz      |
| WebSocket kopması              | Reconnect snapshot doğru            |
| Yetkisiz subscribe             | Subscription reddedilir             |
| Soru bankasında 10’dan az soru | Başlatma durur; tutarlı hata        |
| Yanlış seçenek anahtarı        | 400/422, cevap yazılmaz             |
| Koleksiyon 18/20 + 6 ödül      | 20/20 üst sınırı korunur            |

## 11.2 Test katmanları

- Unit: GameEngine, PuzzleUnlockPolicy, InvitationPolicy, XpPolicy, QuestionSelectionPolicy.

- Repository/integration: Testcontainers PostgreSQL ile unique/FK/check/transaction doğrulaması.

- API: MockMvc veya RestAssured; JWT, validation, HTTP kodları, IDOR.

- WebSocket: iki eşzamanlı istemci, doğru event sırası, reconnect ve authorization.

- Concurrency: aynı anda iki cevap, double ready, double accept, double finish.

- E2E: iki test öğrencisi, aynı Butik grubu, 10 tur, puzzle ve koleksiyon.

- Load: gerçek kullanıcı kapasitesi belli olduktan sonra hedef eşzamanlı maç sayısına göre senaryo.

## 11.3 Gözlemlenebilirlik

| **Metrik**                                     | **Amaç**                     |
|------------------------------------------------|------------------------------|
| kesif_invitations_created_total               | Davet hacmi                  |
| kesif_sessions_started_total / finished_total | Tamamlama oranı              |
| kesif_round_resolution_latency_ms             | Tur kapanış gecikmesi        |
| kesif_ws_connections_active                   | Aktif bağlantılar            |
| kesif_reconnect_total                         | Bağlantı istikrarı           |
| kesif_answer_conflicts_total                  | Çift yanıt / yarış durumları |
| kesif_question_pool_insufficient_total        | İçerik kapasitesi            |
| kesif_outbox_pending_total                    | Olay yayını gecikmesi        |

Log örnek anahtarları: traceId, sessionId, roundNo, eventType, durationMs, resultCode. Kişisel veri ve soru cevap anahtarları loglara yazılmaz.

## 11.4 Kod/link katılımı ek test matrisi

| Test | Beklenen |
|---|---|
| Geçerli kod + oturum açmış kullanıcı | Oda katılımı veya onay talebi |
| Yanlış / süresi geçmiş kod | 404/410 veya güvenli genel hata; sızıntı yok |
| Link ile giriş yapan oturumsuz kullanıcı | Login'e yönlendirilir; dönüşte token güvenli biçimde doğrulanır |
| Aynı link ile iki ayrı kullanıcı aynı anda katılır | 2 kişilik limit; yalnız biri kabul edilir |
| Oda sahibi kendi koduyla ikinci oyuncu olmaya çalışır | 409 `ALREADY_ROOM_MEMBER` |
| Başka öğrencinin oda durumunu sorgulama | 403/404 |
| Link token brute force | Rate-limit + alarm |
| Host reddi / oda kapatması | Yeni katılım engellenir, istemciler bilgilendirilir |
| Davet linki farklı Butik grubundan kullanıcıya gider | Karar A13'e göre host onayı veya ret |
| Oyun sırasında bağlantı kesilir | Yetkili reconnect; session verisi korunur |
