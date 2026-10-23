# 6. Java domain modeli ve katmanlar

## 6.1 Paket önerisi

> com.doping.kesif
>
> config/ SecurityConfig, WebSocketConfig, OpenApiConfig
>
> auth/ CurrentUserProvider, JwtAdapter
>
> integration/ BoutiqueGroupClient, QuestionBankClient
>
> catalog/ Subject, Topic, Puzzle + repository/service/controller
>
> invitation/ InvitationEntity, InvitationService, InvitationController
>
> game/ GameSession, SessionPlayer, GameRound, Answer
>
> GameEngine, GameStateMachine, GameRepository
>
> collection/ PlayerCollection, CollectionPiece, CollectionService
>
> reward/ XpLedger, RewardService
>
> realtime/ KeşifWsController, EventPublisher, Payloads
>
> common/ exception, error, pagination, idempotency
>
> outbox/ OutboxEvent, OutboxPublisher

## 6.2 Entity ↔ DTO ↔ service ayrımı

Entity: JPA kalıcı veri modeli. Request DTO: kullanıcıdan gelen doğrulanmış alanlar. Response DTO: istemciye güvenli biçimde dönen alanlar. Service: iş kuralları ve transaction sınırı. Repository: veritabanı erişimi. Domain event: istemcilere yayınlanacak değişiklik. JPA entity nesneleri doğrudan API’den döndürülmemelidir.

## 6.3 Ana entity sınıfları

| **Java sınıfı**  | **Temel alanlar**                                            | **İlişki / not**             |
|------------------|--------------------------------------------------------------|------------------------------|
| KeşifPlayer     | UUID id, String externalUserId, int xpTotal                  | Harici kimlik referansı      |
| Invitation       | UUID id, inviter, invitee, status, expiresAt                 | @ManyToOne Player            |
| GameSession      | UUID id, status, currentRound, unlockedCount, version        | @Version ile optimistic lock |
| SessionPlayer    | session, player, ready, correctCount                         | UNIQUE(session,player)       |
| GameRound        | session, roundNo, questionSnapshot, correctOptionKey, status | Soru snapshot gizli tutulur  |
| RoundAnswer      | round, player, optionKey, isCorrect, source                  | UNIQUE(round,player)         |
| Puzzle           | code, assetKey, pieceCount                                   | 20 sabit                     |
| PieceUnlock      | session, round, pieceIndex, earnedBy                         | Parça kaydı                  |
| PlayerCollection | player, puzzle, unlockedCount, version                       | Kişisel ilerleme             |
| CollectionPiece  | collection, pieceIndex, sourceSession                        | Benzersiz parça              |
| XpLedger         | player, session, eventType, points, dedupeKey                | Mükerrer ödül önleme         |

## 6.4 Enum taslağı

> InvitationStatus = PENDING, ACCEPTED, REJECTED, CANCELLED, EXPIRED
>
> GameStatus = WAITING_FOR_PLAYERS, READY, IN_PROGRESS, FINISHED, CANCELLED, ABORTED
>
> RoundStatus = PENDING, QUESTION_OPEN, REVEAL, COMPLETED, TIMEOUT
>
> AnswerSource = SUBMITTED, TIMEOUT

## 6.5 DTO sözleşmesi

| **DTO**                 | **Örnek alanlar**                                                                       |
|-------------------------|-----------------------------------------------------------------------------------------|
| CreateInvitationRequest | inviteeExternalUserId, groupExternalId, puzzleId, topicId                               |
| InvitationResponse      | invitationId, status, expiresAt, invitee display                                        |
| ReadyRequest            | ready=true                                                                              |
| QuestionView            | sessionId, roundNo, questionId, body, options, expiresAt; doğru cevap YOK               |
| SubmitAnswerRequest     | roundId, optionKey, clientRequestId                                                     |
| RoundRevealView         | roundNo, correctOptionKey, playerResults, newlyUnlockedPieceIndices                     |
| SessionStateView        | sessionId, status, currentRound, unlockedPieceIndices, myAnswerStatus                   |
| GameResultView          | myCorrectCount, partnerCorrectCount, sharedUnlockedCount, xpAwarded, collectionProgress |

## 6.6 Oda ve davet bağlantısı domain katmanı

| Sınıf | Tür | Sorumluluk |
|---|---|---|
| `KesifRoom` | Entity | Oda sahibi, kod/link hash, süre, durum, kapasite |
| `KesifRoomJoinRequest` | Entity | Odaya katılım talebi, onay/red, denetim |
| `RoomService` | Service | Oda oluştur, kod üret, katılma doğrula, kapat |
| `RoomJoinPolicy` | Domain policy | Auth, kapasite, süre, izin, dış grup kararı |
| `RoomTokenService` | Security service | Kriptografik rastgele token üret, HMAC/hash doğrula |
| `RoomController` | REST Controller | `/rooms` kaynakları |
| `RoomRealtimePublisher` | Application service | `ROOM_CREATED`, `JOIN_REQUESTED`, `ROOM_READY` olayları |
| `CreateRoomRequest`, `JoinRoomByCodeRequest`, `JoinRoomByLinkRequest`, `RoomSummaryResponse` | DTO | İstemci sözleşmesi |

**Asla** `roomCode`, `inviteToken` veya `correctOption` alanlarını JPA entity'sinden doğrudan JSON'a serialize etmeyin. `roomCode` yalnız oluşturma/yenileme yanıtında ve oda sahibine izinli gösterim sırasında ham biçimde iletilir; DB'de doğrulama hash'i tutulur. [16](16-rooms-codes-and-invite-links.md).
