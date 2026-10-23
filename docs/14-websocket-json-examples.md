# Ek B. Örnek WebSocket JSON olayları

```json
{

"eventId": "<uuid>", "type": "QUESTION_STARTED",

"sessionId": "<uuid>", "sequenceNumber": 11,

"serverTime": "2026-10-20T14:00:00Z",

"data": {

"roundNo": 1, "totalRounds": 10,

"question": { "id": "<uuid>", "body": "...",

"options": [{"key":"A","text":"..."},{"key":"B","text":"..."}] },

"expiresAt": "2026-10-20T14:01:00Z"

}

}

{

"type": "ROUND_REVEALED", "sessionId": "<uuid>",

"data": { "roundNo": 1, "correctOptionKey": "B",

"players": [{"playerId":"p1","correct":true},

{"playerId":"p2","correct":false}],

"newlyUnlockedPieceIndices": [7], "sharedUnlockedCount": 1 }

}
```
