# 7. REST API sözleşmeleri

Önerilen prefix: /api/v1/kesif. **Oda oluşturma, kod/link katılımı ve oda sahibi onayı API’leri için [16-rooms-codes-and-invite-links.md](16-rooms-codes-and-invite-links.md) ve [17-room-api-and-websocket-contracts.md](17-room-api-and-websocket-contracts.md) esas alınır.** Tüm endpoint’ler kimlik doğrulaması ister; /actuator/health ve admin yönetimi ayrı güvenlik kurallarına tabidir. Sunucu öğrenci kimliğini token’dan çözer.

| **Metot** | **Yol**                               | **İşlev**                       | **Başarı** |
|-----------|---------------------------------------|---------------------------------|------------|
| GET       | /catalog/grades                       | 5,6,7,8                         | 200        |
| GET       | /catalog/subjects?grade=8             | Dersler                         | 200        |
| GET       | /catalog/topics?grade=8&subjectId=... | Konular                         | 200        |
| GET       | /puzzles                              | Aktif puzzle listesi            | 200        |
| GET       | /groups                               | Öğrencinin uygun Butik grupları | 200        |
| GET       | /groups/{groupId}/friends             | Aynı gruptaki uygun arkadaşlar  | 200        |
| POST      | /invitations                          | Davet oluştur                   | 201        |
| GET       | /invitations/incoming                 | Bekleyen davetler               | 200        |
| POST      | /invitations/{id}/accept              | Davet kabul; maç aç             | 200        |
| POST      | /invitations/{id}/reject              | Davet reddet                    | 200        |
| POST      | /invitations/{id}/cancel              | Gönderen iptal                  | 200        |
| GET       | /sessions/{id}                        | Maç durumu / reconnect snapshot | 200        |
| POST      | /sessions/{id}/ready                  | Hazır ol                        | 200        |
| POST      | /sessions/{id}/answers                | Cevap gönder (REST fallback)    | 200/202    |
| GET       | /sessions/{id}/result                 | Final sonucu                    | 200        |
| GET       | /me/collections                       | Kişisel puzzle koleksiyonu      | 200        |
| GET       | /me/history?page=0&size=20            | Maç geçmişi                     | 200        |

## 7.1 Davet oluşturma örneği

> POST /api/v1/kesif/invitations
>
> Authorization: Bearer \<JWT\>
>
> { "inviteeExternalUserId": "usr_202", "groupExternalId": "grp_11",
>
> "puzzleId": "\<uuid\>", "topicId": "\<uuid\>" }
>
> → 201 { "invitationId": "\<uuid\>", "status": "PENDING",
>
> "expiresAt": "2026-10-20T14:02:00Z" }

Kontroller: davet eden != davet edilen; her ikisi aynı aktif Butik grubunda; puzzle/konu aktif; davetli uygun; davet limitleri aşılmadı; aynı ikili için yinelenen aktif davet yok.

## 7.2 Cevap gönderme örneği

> POST /api/v1/kesif/sessions/{sessionId}/answers
>
> { "roundId": "\<uuid\>", "optionKey": "B",
>
> "clientRequestId": "\<uuid\>" }
>
> → 202 { "accepted": true, "roundId": "\<uuid\>", "waitingForPartner": true }

Sunucu doğru/yanlış bilgisini iki yanıt tamamlanmadan cevap HTTP yanıtına koymaz. Aynı clientRequestId ile tekrar gelen istek aynı sonucu döndürür. Farklı optionKey ile ikinci cevap 409 döner.

## 7.3 Hata formatı ve kodlar

> { "timestamp": "2026-10-20T14:00:00Z", "status": 409,
>
> "code": "ANSWER_ALREADY_SUBMITTED",
>
> "message": "Bu tur için cevap zaten kaydedildi.",
>
> "traceId": "..." }

| **HTTP** | **Kod**                                           | **Durum**                                                 |
|----------|---------------------------------------------------|-----------------------------------------------------------|
| 400      | VALIDATION_ERROR                                  | Eksik veya geçersiz parametre                             |
| 401      | UNAUTHENTICATED                                   | Token yok/geçersiz                                        |
| 403      | NOT_SAME_GROUP / NOT_SESSION_PLAYER               | Grup veya maç yetkisi yok                                 |
| 404      | SESSION_NOT_FOUND                                 | Kayıt bulunamadı (yetkisiz durumda 404 tercih edilebilir) |
| 409      | INVITATION_NOT_PENDING / ANSWER_ALREADY_SUBMITTED | Durum çatışması                                           |
| 410      | INVITATION_EXPIRED / ROUND_CLOSED                 | Süre doldu                                                |
| 429      | RATE_LIMITED                                      | Davet/cevap sınırı                                        |
| 503      | QUESTION_BANK_UNAVAILABLE                         | Bağımlılık geçici hatası                                  |
