# 10. Güvenlik, gizlilik ve veri saklama

## 10.1 Kimlik ve erişim

- Her REST ve STOMP komutunda kullanıcı kimliği sunucu oturumundan elde edilir; client-supplied playerId güvenilmez.

- **Butik arkadaşı davetinde** davet öncesi ve kabul sırasında iki öğrencinin aynı aktif Butik grubunda olduğu doğrulanır. **Kod/link katılımında** grup üyeliği yerine oda kodu/linki, kimlik doğrulama, oda kapasitesi, erişim süresi ve (etkinse) oda sahibi onayı kontrol edilir.

- Session, invitation, history ve collection okumalarında object-level authorization uygulanır (IDOR önleme).

- Soruların doğru cevapları yalnız backend’de tutulur; REVEAL öncesi response, WebSocket veya browser state’e gönderilmez.

- Davet gönderimi ve cevap gönderimi için rate limit; eşzamanlı aktif oyun sınırı önerilir.

- Token süre sonu, yeniden bağlantı ve WebSocket subscribe yetkilendirmesi test edilir.

## 10.2 Çocuk kullanıcı verileri

Öğrenciler reşit olmayabileceği için veri minimizasyonu esas alınır. Profilde yalnız dış kullanıcı ID ve gerekiyorsa gösterim adı tutulur; telefon, e-posta, doğum tarihi gibi alanlar Keşif için kopyalanmaz. Grup listesi yalnız uygun öğrencilere gösterilir. Sohbet özelliği yoktur. KVKK ve kurumun çocuk kullanıcı politikalarına göre saklama süresi, log maskeleme, erişim rolleri ve silme/anonymize akışı hukuk ve güvenlik ekipleriyle belirlenmelidir.

## 10.3 Saklama, izleme ve gizlilik

AÇIK: Maç geçmişi ve cevap detaylarının kaç gün/ay tutulacağı belirlenmeli. Öneri: operasyon loglarında token, tam soru cevap anahtarı ve kişisel veri tutulmaması; requestId/traceId ile korelasyon. Erişim denetimi, TLS, DB şifreleme/backup, secret manager ve least privilege uygulanmalı.

## 10.4 Oda kodu ve davet bağlantısı güvenliği

- **Kod/link JWT değildir; JWT'nin yerine geçmez.** Her iki öğrencinin de kimliği doğrulanmalıdır.
- Kod kısa ve tahmin edilebilir olabileceği için rate-limit, deneme sayacı, süre sınırı ve tercihen host onayı zorunlu güvenlik kontrolleridir.
- Davet linki token'ı kriptografik güçlü ve tahmin edilemez olmalı; DB'de HMAC/hash olarak saklanmalı; loglara, analitik event'lerine ve referer'a sızmamalıdır.
- Link URL'si yalnızca kullanıcı tarafından açılan hedef origin'de işlenmeli; yönlendirme parametresi ile açık yönlendirme yapılmamalıdır.
- Odaya katılım hakkı, **başka öğrencilerin profil veya soru cevap verilerine erişim** vermez.
- Oda sahibi dış gruptan katılımcıyı kabul etmeden önce yalnız minimum gerekli profil bilgisini görür.
- Gerekli ekip kararları: grup dışı öğrenci kabulü, link geçerliliği, oda sahibi onayı ve veri saklama süresi.
