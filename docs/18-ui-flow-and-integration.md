# 18. Doping Keşif — Frontend Akışı ve Dashboard Entegrasyonu

## 18.1 Tasarım sınırı

Doping Hafıza'nın **sol sidebar'ı**, **üst arama/ikon/profil alanı** ve **sağ takvim/promosyon paneli** aynen korunur. Sol menünün alt bölümüne **Doping Keşif** girişi eklenir. Yalnız **orta içerik paneli** Keşif ekranlarını gösterir. Temel renkler mevcut platforma yakın **beyaz, açık gri, mavi**; önceki mor Battle teması kullanılmaz.

## 18.2 Ekran listesi

| Adım | Ekran | Temel bileşen | Backend bağımlılığı |
|---|---|---|---|
| 1 | Keşif ana sayfası | Görev kartları, koleksiyon ilerlemesi, Oyna | `GET /puzzles`, `/me/collections` |
| 2 | Oyun ayarları | 5–8. sınıf, ders, konu (opsiyonel), görev | Katalog API'leri |
| 3 | Arkadaş / oda yöntemi | **Arkadaş Seç (Doping Butik)** veya **Kod/Link ile Oyna** | Grup API + oda API |
| 4A | Arkadaş seçimi | Aynı Butik grubundan uygun öğrenciler | `GET /groups/{id}/friends`, davet API |
| 4B | Oda oluşturma | Oda oluştur, kodu kopyala, linki paylaş | `POST /rooms` |
| 4C | Odaya katılma | Kodu gir veya davet linkini aç | `/rooms/join-by-code`, `/rooms/join-by-link` |
| 5 | Davet bekleme / lobi | İki avatar, bekliyor/hazır, iptal | Oda snapshot + WebSocket |
| 6 | Soru ekranı | Aynı soru, 4 seçenek, süre, cevap gönder | Session snapshot + cevap API |
| 7 | Cevap sonucu | İki cevap değerlendirildikten sonra puzzle parça açılımı | `ROUND_REVEALED` olayı |
| 8 | Oyun sonucu | 10 soru, açılan parça, XP | Session result API |
| 9 | Koleksiyon | Puzzle kartları, ilerleme, oyun geçmişi | `/me/collections`, `/me/history` |

**Not:** 4A–4C alternatif yollardır; tek bir öğrencinin aynı oyunda iki yolu birden tamamlaması gerekmez.

## 18.3 Önemli UI kuralları

- **Kodu gir** ile **Oda oluştur** birbirinden ayrı aksiyonlar olmalı.
- Oda sahibi kod ve davet linkini kopyalayabilir; paylaşım yalnız kullanıcının başlattığı eylemle olur.
- Linkle gelen kullanıcı login değilse önce giriş yapar; ardından oda katılımına geri döner.
- Oda sahibinin onayı etkinse katılan öğrenci **“Onay bekleniyor”** durumunu görür.
- Oyun başlamadan **“Odayı iptal et”** ve uygun **“Odadan ayrıl”** aksiyonları bulunur.
- Kod süresi doldu, oda dolu, davet reddedildi, bağlantı koptu durumlarının her biri ayrı hata/boş durum ekranıyla ele alınır.
- Sorunun doğru seçeneği iki cevap tamamlanmadan hiçbir frontend payload'ına dahil edilmez.
- Puzzle **soru ekranında zorunlu değil**; önceki tasarımda puzzle sonucu ayrı ekranda açılıyordu. Sağda canlı küçük puzzle önizlemesi varsa, doğru cevapları erken sızdırmayan bir **ilerleme önizlemesi** olmalı; bu görsel davranış UX onayına tabidir.

## 18.4 React bileşen önerileri

`KesifPage`, `PuzzleMissionCard`, `GameSettingsForm`, `PlayMethodSelector`, `BoutiqueFriendPicker`, `RoomCreateCard`, `RoomJoinByCodeForm`, `InviteLinkShare`, `RoomWaitingLobby`, `RoomApprovalPanel`, `QuestionCard`, `AnswerStatus`, `PuzzleReveal`, `GameResult`, `CollectionGrid`.

Durum yönetimi: TanStack Query (REST), WebSocket/STOMP istemcisi (canlı olaylar), sunucu snapshot'ı otorite kabul edilir. Frontend local state yalnız sunum amaçlıdır.

## 18.5 Tasarım kabul kriterleri

- Keşif seçildiğinde ana dashboard çerçevesi değişmez.
- Kod/link ve Butik arkadaş daveti aynı soru motoruna çıkar.
- İki farklı tarayıcı/hesapla lobi ve soru akışı gösterilir.
- Oda kodu/linki yalnız paylaşım için görünür; başka kullanıcıya açık odaların listesi yoktur.
- 1280px ve üzeri masaüstü düzeninde orta panel taşmaz; daha dar görünüm ayrıca test edilir.
