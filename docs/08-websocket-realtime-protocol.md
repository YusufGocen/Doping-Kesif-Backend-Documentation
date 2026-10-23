# 8. WebSocket protokolü

## 8.1 Bağlantı ve abonelik

ÖNERİ: WebSocket handshake sırasında JWT/oturum doğrulanır; STOMP CONNECT de ayrıca doğrulanır. İstemci /ws/kesif üzerinden bağlanır. /user/queue/kesif kişisel olaylar için, /topic/kesif/sessions/{sessionId} maç olayları için kullanılır. Subscription authorization zorunludur; yalnız maç katılımcıları ilgili topic’e abone olabilir.

## 8.2 İstemci → sunucu komutları

| **Destination**                  | **Payload**                         | **Yetki / işlem**                   |
|----------------------------------|-------------------------------------|-------------------------------------|
| /app/kesif/invitations/send     | invitee, group, puzzle, topic       | Grup doğrulama; REST ile alternatif |
| /app/kesif/sessions/{id}/ready  | {ready:true,clientRequestId}        | Yalnız katılımcı                    |
| /app/kesif/sessions/{id}/answer | {roundId,optionKey,clientRequestId} | Yalnız katılımcı, tek yanıt         |
| /app/kesif/sessions/{id}/sync   | {lastEventId}                       | Reconnect snapshot / kaçan olaylar  |

## 8.3 Sunucu → istemci olayları

| **Event type**      | **İçerik**                                     | **Alıcı**                      |
|---------------------|------------------------------------------------|--------------------------------|
| INVITATION_RECEIVED | invitationId, sender, expiresAt                | Davetli                        |
| INVITATION_ACCEPTED | invitationId, sessionId                        | Davet eden                     |
| PLAYER_READY        | sessionId, playerId, ready                     | İki oyuncu                     |
| GAME_STARTED        | sessionId, totalRounds=10                      | İki oyuncu                     |
| QUESTION_STARTED    | roundNo, questionView, expiresAt               | İki oyuncu                     |
| ANSWER_RECEIVED     | roundNo, playerId, answered=true (seçenek yok) | İki oyuncu                     |
| ROUND_REVEALED      | correctOption, result, newPieceIndices         | İki oyuncu                     |
| ROUND_COMPLETED     | roundNo, nextRoundNo                           | İki oyuncu                     |
| GAME_FINISHED       | result, collection, xp                         | Oyuncu bazlı özel alanlar ayrı |
| PLAYER_DISCONNECTED | playerId, reconnectUntil                       | İki oyuncu                     |
| GAME_ABORTED        | reasonCode                                     | İki oyuncu                     |

Önemli: Oyuncunun kişisel XP ve koleksiyon bilgisi ortak topic’te paylaşılmamalı; kişisel user queue üzerinden gönderilmeli. Olaylara eventId, sessionId, serverTime ve sequenceNumber eklenmeli; istemci sıralama/tekrar kontrolü yapabilmelidir.

## 8.4 Gerçek zamanlı güvenilirlik

- Veritabanı transaction’ı commit olmadan WebSocket olayı yayınlanmaz.

- Kritik olaylar için outbox tablosu veya transaction-after-commit publish yaklaşımı kullanılır; üretimde outbox daha güvenilir.

- Client reconnect sonrası GET /sessions/{id} veya sync komutuyla sunucunun güncel snapshot’ı alınır.

- İstemcinin saati değil, expires_at ve sunucu saati geçerlidir.

- Tek instance prototipte STOMP simple broker kullanılabilir; çoklu instance için broker relay ve sticky-session/dağıtık oturum tasarımı gerekir.

## 8.5 Oda ve link katılımı olayları

Yeni topic/komut sözleşmesi: [17](17-room-api-and-websocket-contracts.md). `ROOM_JOIN_REQUESTED`, `ROOM_JOIN_APPROVED`, `ROOM_JOIN_REJECTED`, `ROOM_PARTICIPANT_JOINED`, `ROOM_READY`, `ROOM_EXPIRED` ve `ROOM_CLOSED` olayları önerilir. **Yalnızca yetkili oda katılımcıları** oda olaylarına abone olabilir. Ham oda kodu ve link token'ı topic payload'larına yazılmaz.
