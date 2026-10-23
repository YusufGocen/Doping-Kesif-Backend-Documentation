# 9. Oyun motoru ve transaction kuralları

## 9.1 Başlatma algoritması

9.  İki oyuncu READY olunca session satırını pessimistic lock veya version kontrolüyle kilitle.

10. İki oyuncunun aktif grup üyeliğini yeniden doğrula; puzzle ve konu aktif mi kontrol et.

11. Seçilen sınıf/ders/konudan yayınlanmış en az 10 uygun soru bulun; yoksa maçı başlatma.

12. 10 benzersiz soruyu seç; seçenek sırasını ve doğru cevabı sunucu snapshot’ına sabitle.

13. 1–20 puzzle parçası sırasını karıştırıp piece_order’a kaydet.

14. 10 kesif_round satırını oluştur; session.status=IN_PROGRESS; round 1’i QUESTION_OPEN yap.

15. Commit sonrası GAME_STARTED ve QUESTION_STARTED olaylarını yayınla.

## 9.2 Cevap kabul algoritması

> submitAnswer(sessionId, roundId, optionKey, requestId, currentUser):
>
> authenticate + session membership check
>
> transaction: lock current round/session
>
> ensure status == QUESTION_OPEN and serverNow \< expiresAt
>
> if same player answer exists: return idempotent or 409
>
> validate optionKey against stored snapshot
>
> compute isCorrect using server-only correctOptionKey
>
> INSERT answer with UNIQUE(roundId, playerId)
>
> if both answers exist: resolveRoundOnce()
>
> commit → publish events

## 9.3 Tur kapatma algoritması

> resolveRoundOnce(round):
>
> lock round and ensure status == QUESTION_OPEN
>
> create TIMEOUT answers for missing players if expired
>
> correctCount = count(is_correct == true) // 0..2
>
> unlock next correctCount unused indices from session.piece_order
>
> insert PieceUnlock rows with UNIQUE(session,pieceIndex)
>
> update player correctCount and session.unlockedCount
>
> round.status = REVEAL; resolvedAt = now
>
> emit ROUND_REVEALED after commit
>
> after reveal delay/ack: mark COMPLETED; open next round or finish

REVEAL ekranı için sunucu kontrollü kısa bekleme veya iki oyuncunun “devam” onayı seçenekleri vardır. Bu ürün kararı netleşmelidir; yalnız istemci timer’ına güvenilmemelidir.

## 9.4 Maç bitirme ve idempotent ödül

16. 10 tur COMPLETED olduktan sonra session lock alınır; zaten FINISHED ise mevcut sonucu dön.

17. Maçta açılan benzersiz parçaları ve oyuncu doğru sayılarını veritabanından yeniden hesapla.

18. Her oyuncu için koleksiyon kilidi al; yeni parça kazanımlarını UNIQUE ile yaz, 20 üst sınırını koru.

19. ÖNERİ XP: maç tamamlama +20, kişisel doğru başına +5, koleksiyon ilk tamamlanınca +30; yalnız ürün onayıyla aktifleştir.

20. XP ledger için dedupe_key = sessionId:playerId:eventType\[:roundNo\] üret; aynı ödül iki kez yazılmaz.

21. session.status=FINISHED, finished_at doldur; outbox’a GAME_FINISHED kaydet; commit.

## 9.5 Eşzamanlılık ve transaction sınırları

İki öğrenci cevabı milisaniye arayla gönderebilir. Transaction lock, UNIQUE constraint ve idempotency birlikte kullanılmalı. Sadece Java synchronized veya uygulama belleğindeki sayaç yeterli değildir. Session/round satırları üzerinde lock order sabit tutularak deadlock riski azaltılmalıdır. Koleksiyon güncellemesi transaction içinde ve optimistic lock retry veya row lock ile yapılmalıdır.

## 9.6 Bağlantı kopması / terk etme

ÖNERİ: Bağlantı koptuğunda maç hemen sonlandırılmaz. 60–120 saniyelik reconnect süresi tanınır; aktif turun timer’ının durup durmayacağı ürün kararıdır. Grace period sonunda eksik yanıtlar TIMEOUT olarak işlenebilir veya maç ABORTED yapılabilir. Kesinti sonrası oyun snapshot’ı sunucudan tekrar alınır.

## 9.7 Oda oluşturma ve eşzamanlı katılım transaction'ı

1. Oda sahibi oturum açmış kullanıcıdır; yeni oda `WAITING` statüsünde ve `max_players=2` açılır.
2. Kod/link girişinde kullanıcı JWT'den belirlenir; erişim token'ı doğrulanır, oda süre ve kapasite kontrol edilir.
3. Aynı anda iki kişinin boş tek koltuğa katılma yarışında `SELECT ... FOR UPDATE` veya optimistic locking kullanılır; **yalnızca bir talep** kabul edilir.
4. Onay gerektiren modda `JOIN_REQUESTED` kaydı oluşturulur; host onayı olmadan ikinci koltuk dolmuş sayılmaz.
5. Oda `READY` olduğunda tek bir `kesif_session` oluşturulur (`UNIQUE(room_id)`), iki oyuncu hazır olduğunda 10 soru seçilir.
6. Link/kod tekrar kullanımı idempotent olmalıdır; aynı öğrencinin tekrar denemesi ikinci oyuncu sayılmaz.
7. Kod ve link sona erdiğinde yeni katılım reddedilir; başlamış oyun için devam politikası ayrıca tanımlanır.

Ayrıntı: [16](16-rooms-codes-and-invite-links.md), [17](17-room-api-and-websocket-contracts.md).
