# Ek A. Örnek SQL DDL (çekirdek tablolar)

Aşağıdaki SQL örnek taslaktır; tüm kolonlar ve indeksler üretim migration’larında tamamlanmalıdır. Üretim ortamında dış kullanıcı ID tipi ve mevcut servis adaptörleri doğrulanmalıdır.

```sql
CREATE SCHEMA IF NOT EXISTS kesif;

CREATE TABLE kesif.kesif_session (

id uuid PRIMARY KEY, invitation_id uuid UNIQUE,

group_external_id varchar(128) NOT NULL,

puzzle_id uuid NOT NULL, topic_id uuid NOT NULL,

status varchar(24) NOT NULL,

current_round smallint NOT NULL DEFAULT 0 CHECK (current_round BETWEEN 0 AND 10),

unlocked_count smallint NOT NULL DEFAULT 0 CHECK (unlocked_count BETWEEN 0 AND 20),

piece_order jsonb NOT NULL,

version bigint NOT NULL DEFAULT 0,

started_at timestamptz, finished_at timestamptz,

created_at timestamptz NOT NULL DEFAULT now(),

updated_at timestamptz NOT NULL DEFAULT now()

);

CREATE TABLE kesif.kesif_round (

id uuid PRIMARY KEY, session_id uuid NOT NULL REFERENCES kesif.kesif_session(id),

round_no smallint NOT NULL CHECK (round_no BETWEEN 1 AND 10),

question_snapshot jsonb NOT NULL,

correct_option_key varchar(8) NOT NULL,

status varchar(20) NOT NULL, opened_at timestamptz,

expires_at timestamptz, resolved_at timestamptz,

UNIQUE (session_id, round_no)

);

CREATE TABLE kesif.kesif_round_answer (

id uuid PRIMARY KEY, round_id uuid NOT NULL REFERENCES kesif.kesif_round(id),

player_id uuid NOT NULL, option_key varchar(8),

is_correct boolean NOT NULL, source varchar(12) NOT NULL,

client_request_id uuid, answered_at timestamptz NOT NULL DEFAULT now(),

UNIQUE (round_id, player_id)

);

CREATE UNIQUE INDEX uq_kesif_answer_request

ON kesif.kesif_round_answer(player_id, client_request_id)

WHERE client_request_id IS NOT NULL;
```
