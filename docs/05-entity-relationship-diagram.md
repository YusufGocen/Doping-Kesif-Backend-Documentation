# 5. ER diyagramı ve ilişkiler

> kesif_player_profile 1 ── N kesif_invitation (inviter/invitee)
>
> kesif_invitation 1 ── 0..1 kesif_session
>
> kesif_session 1 ── 2 kesif_session_player N ── 1 kesif_player_profile
>
> kesif_subject 1 ── N kesif_topic 1 ── N kesif_question 1 ── N kesif_question_option
>
> kesif_puzzle 1 ── N kesif_session
>
> kesif_session 1 ── 10 kesif_round 1 ── 0..2 kesif_round_answer
>
> kesif_session 1 ── 0..20 kesif_piece_unlock
>
> kesif_player_profile 1 ── N kesif_player_collection N ── 1 kesif_puzzle
>
> kesif_player_collection 1 ── 0..20 kesif_collection_piece
>
> kesif_player_profile 1 ── N kesif_xp_ledger

Fiziksel ERD’de session_player tablosu oyuncu-maç N:N ilişkisini çözer. Round_answer hem round hem player’a bağlıdır; cevabı veren oyuncunun o session’a katıldığının servis katmanında ayrıca kontrol edilmesi gerekir. Invitation → Session birebir ve opsiyonel ilişki olarak tasarlanır.

## 5.1 Görsel ER ilişki diyagramı (Mermaid)

Aşağıdaki şema, yukarıdaki ilişkilerin görsel özetidir. Kolon tanımları için 4. bölüme bakınız.

```mermaid
erDiagram
    kesif_player_profile ||--o{ kesif_invitation : "davet eder/alır"
    kesif_invitation ||--o| kesif_session : "oturum oluşturur"
    kesif_player_profile ||--o{ kesif_session_player : "katılır"
    kesif_session ||--|{ kesif_session_player : "2 oyuncu"
    kesif_subject ||--o{ kesif_topic : "konuları"
    kesif_topic ||--o{ kesif_question : "soruları"
    kesif_question ||--o{ kesif_question_option : "seçenekleri"
    kesif_puzzle ||--o{ kesif_session : "oynanır"
    kesif_session ||--|{ kesif_round : "10 tur"
    kesif_round ||--o{ kesif_round_answer : "0-2 cevap"
    kesif_session ||--o{ kesif_piece_unlock : "açılan parçalar"
    kesif_player_profile ||--o{ kesif_player_collection : "koleksiyonları"
    kesif_puzzle ||--o{ kesif_player_collection : "ait olduğu puzzle"
    kesif_player_collection ||--o{ kesif_collection_piece : "parçaları"
    kesif_player_profile ||--o{ kesif_xp_ledger : "XP hareketleri"
```

## 5.2 Mermaidsiz şema okuma

İlişkilerde 1:N = bir kayıttan çok kayıt; 1:2 = bir maça tam iki katılımcı (uygulama invariant’ı); 1:10 = maçta on tur; 0..2 = her turda en fazla iki yanıt; 0..20 = en fazla yirmi puzzle parçası. Bu sayılar PostgreSQL üzerinde yalnız FK ile değil, transaction ve testlerle korunur.

## 5.3 Yeni oda ilişkileri

```mermaid
erDiagram
    kesif_player_profile ||--o{ kesif_room : owns
    kesif_room ||--o{ kesif_room_join_request : receives
    kesif_player_profile ||--o{ kesif_room_join_request : requests
    kesif_room ||--o| kesif_session : starts
    kesif_session ||--|{ kesif_session_player : includes
    kesif_player_profile ||--o{ kesif_session_player : participates
    kesif_invitation o|--o| kesif_session : optional_source
```

Yeni iki tablo dahil tam genişletme: [19 — SQL/ER](19-room-sql-and-er-diagram.md).
