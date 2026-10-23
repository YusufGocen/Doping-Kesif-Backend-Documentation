# 17. Oda REST API ve WebSocket Sözleşmeleri

> Prefix: `/api/v1/kesif` — öneri. OpenAPI/Swagger sözleşmesi geliştirme sırasında gerçek DTO'larla eşleşmelidir.

## 17.1 Yeni REST endpoint'leri

| Metot | Yol | Açıklama | Başarı | Yetki |
|---|---|---|---|---|
| `POST` | `/rooms` | Oda oluştur, paylaşım kodu/linki üret | 201 | Giriş yapmış öğrenci |
| `POST` | `/rooms/join-by-code` | Kod ile katılım veya onay talebi | 200/202 | Giriş yapmış öğrenci |
| `POST` | `/rooms/join-by-link` | Opaque link token ile katılım veya onay talebi | 200/202 | Giriş yapmış öğrenci |
| `GET` | `/rooms/{roomId}` | Yetkili katılımcı için oda snapshot | 200 | Host / onaylı oyuncu |
| `GET` | `/rooms/{roomId}/join-requests` | Bekleyen talepler | 200 | Host |
| `POST` | `/rooms/{roomId}/join-requests/{requestId}/approve` | Talebi onayla | 200 | Host |
| `POST` | `/rooms/{roomId}/join-requests/{requestId}/reject` | Talebi reddet | 200 | Host |
| `POST` | `/rooms/{roomId}/cancel` | Odayı iptal et | 200 | Host |
| `POST` | `/rooms/{roomId}/rotate-invite` | Kod/link yenile (opsiyonel) | 200 | Host |
| `GET` | `/me/rooms/active` | Öğrencinin aktif oda durumu | 200 | Kendisi |

**Eski 17 REST endpoint'i** aynen korunur; oda yolunda `sessions` işlemleri `roomId` ile bağlanır. Kod/link için ayrı bir kimlik doğrulama endpoint'i açılmaz; platform JWT/SSO kullanılır.

## 17.2 Örnek oda oluşturma

```http
POST /api/v1/kesif/rooms
Authorization: Bearer <access-token>
Content-Type: application/json
```
```json
{
  "puzzleId": "11111111-1111-4111-8111-111111111111",
  "topicId": "22222222-2222-4222-8222-222222222222",
  "joinPolicy": "HOST_APPROVAL"
}
```
```json
{
  "roomId": "33333333-3333-4333-8333-333333333333",
  "status": "WAITING",
  "joinCode": "DK7A3Q",
  "inviteUrl": "https://<onayli-origin>/kesif/katil?token=<opaque-token>",
  "expiresAt": "2026-10-23T15:00:00Z",
  "maxPlayers": 2
}
```

`inviteUrl` örnek rotadır; gerçek origin platform ekibiyle belirlenir. Ham kod/token yanıtını loglamayın; bu yanıt yalnız yetkili oda sahibine verilir.

## 17.3 Kod ile katılma

```http
POST /api/v1/kesif/rooms/join-by-code
Authorization: Bearer <access-token>
```
```json
{"code":"DK7A3Q","clientRequestId":"44444444-4444-4444-8444-444444444444"}
```
```json
{"roomId":"33333333-3333-4333-8333-333333333333","joinStatus":"PENDING_APPROVAL","message":"Oda sahibinin onayı bekleniyor."}
```

Host onayı kapalıysa `joinStatus=APPROVED` olabilir. Grup dışı katılım politikası A13'e bağlıdır.

## 17.4 Link ile katılma

```http
POST /api/v1/kesif/rooms/join-by-link
Authorization: Bearer <access-token>
```
```json
{"token":"<opaque-token>","clientRequestId":"55555555-5555-4555-8555-555555555555"}
```

Token geçersiz/süresi geçmişse hassas ayrıntı sızdırmayan hata döndürülür. Login dönüşü token'ın istemcide güvenli ve kısa süreli tutulması gerekir; URL token'ı üçüncü taraf izleme servislerine gönderilmez.

## 17.5 Oda olayları

| Olay | Alıcı | Veri (özet) |
|---|---|---|
| `ROOM_CREATED` | Host | `roomId,status,expiresAt` |
| `ROOM_JOIN_REQUESTED` | Host | `roomId,requestId,requesterPublicProfile` |
| `ROOM_JOIN_APPROVED` | Katılımcı + host | `roomId,playerPublicProfile` |
| `ROOM_JOIN_REJECTED` | Talep eden | `roomId,reasonCode` |
| `ROOM_PARTICIPANT_JOINED` | Host + katılımcı | `roomId,playerPublicProfile` |
| `ROOM_READY` | Host + katılımcı | `roomId,sessionId` |
| `ROOM_EXPIRED` | Host + ilgili kullanıcı | `roomId` |
| `ROOM_CLOSED` | Host + ilgili kullanıcı | `roomId,reason` |

Önerilen hedefler: `/user/queue/kesif` (kişisel), `/topic/kesif/rooms/{roomId}` (oda). Abonelik yalnız host ve onaylı katılımcıya açık; onay bekleyen öğrenciye kişisel olaylar gönderilir. Ham kod/token hiçbir WebSocket olayında yer almaz.

## 17.6 Hata sözleşmesi

| HTTP | Kod | Ne zaman? |
|---|---|---|
| 401 | `UNAUTHENTICATED` | Giriş yapılmadı |
| 403/404 | `ROOM_ACCESS_DENIED` | Yetkisiz oda erişimi |
| 404/410 | `ROOM_CODE_INVALID_OR_EXPIRED` | Kod bulunamadı veya süresi doldu |
| 409 | `ROOM_FULL` | İkinci oyuncu zaten var |
| 409 | `ALREADY_ROOM_MEMBER` | Kendi odasına tekrar katılım |
| 409 | `JOIN_REQUEST_ALREADY_PENDING` | Aynı bekleyen talep |
| 409 | `ROOM_NOT_JOINABLE` | Oyun başladı/oda kapandı |
| 429 | `ROOM_JOIN_RATE_LIMITED` | Çok fazla deneme |

## 17.7 OpenAPI/Swagger

- Springdoc OpenAPI ile `Rooms` tag'i ve request/response şemaları üretilecek.
- Bearer JWT security scheme tüm oda API'lerinde tanımlanacak.
- Swagger UI üzerinden demo için test JWT kullanımı, üretim sırlarının saklanması ve örnek kodların sahte olduğu açıklanacak.
- WebSocket sözleşmeleri Swagger kapsamı dışında; bu dosya ve olay şemaları ayrıca sürümlenecek.
