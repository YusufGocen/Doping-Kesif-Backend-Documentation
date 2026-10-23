# 20. Ürün Kararları, İsim Değişikliği ve Sürüm Notları

## 20.1 Kaynak ve değişiklik özeti

Bu paket, kullanıcının gönderdiği **Doping Battle Backend Dokümantasyon Projesi ZIP** dosyasındaki 17 Markdown belge ve kök yapı temel alınarak düzenlenmiştir. Eski teknik içeriğin PostgreSQL/Java/REST/WebSocket/algoritma/test ayrıntıları korunmuş; **Doping Keşif** adı, yeni UI ve **oda kodu / davet linki** özelliğiyle uyumlu genişletilmiştir.

| Alan | Önce | Şimdi |
|---|---|---|
| Ürün adı | Doping Battle | **Doping Keşif** |
| Oyun yaklaşımı | Ortak puzzle, yarış yok | Aynı, iş birlikçi |
| Katılım | Yalnız Butik grup arkadaşı | **Butik arkadaşı + kod/link ile oda** |
| Ekran | Mor Battle prototipleri | Mevcut Doping Hafıza beyaz/mavi dashboard içinde orta panel |
| Oda | Davet + session ağırlıklı | Yeni `kesif_room` + `kesif_room_join_request` |
| REST | `/api/v1/battle` | `/api/v1/kesif` (öneri) |
| WebSocket | `/ws/battle` | `/ws/kesif` (öneri) |
| PostgreSQL | `battle` şeması, `battle_*` | `kesif` şeması, `kesif_*` (sıfırdan kurulum önerisi) |
| Java | `com.doping.battle` | `com.doping.kesif` (öneri) |
| Tablolar | 17 | **19 önerilen** (17 eski + 2 oda tablosu) |

## 20.2 Sabit kalan kurallar

- 5–8. sınıf, ders ve konu seçimi; manuel zorluk seçimi yok.
- İki oyuncu, aynı 10 soru, bağımsız cevaplar, iki cevap veya timeout sonrası sonuç.
- Doğru cevap başına 1 parça; 20 parçalık ortak puzzle.
- Kişisel koleksiyon, oyun geçmişi ve XP (XP katsayıları onaya tabi).
- Rastgele eşleştirme, sohbet, liderlik tablosu yok.
- JWT ile kimlik doğrulama, sunucu tarafı doğru cevap, WebSocket ile eşzamanlılık.

## 20.3 Açık kararlar (ekip onayı gerekli)

| ID | Soru | Geçici öneri |
|---|---|---|
| A13 | Kod/link ile farklı Butik grubundan öğrenci katılabilir mi? | Evet, **host onayıyla**; ürün ekibi onaylamalı |
| A14 | Oda sahibinin katılım onayı zorunlu mu? | Dış gruptan katılımda evet |
| A15 | Oda/kod/link ne kadar geçerli? | Örn. 30 dk; ürün kararı |
| A16 | Kod uzunluğu ve link yenileme politikası? | 6–8 karakter, CSPRNG; güvenlik kararı |
| A17 | Link açan giriş yapmamış öğrenci ne görür? | Login → güvenli geri dönüş |
| A18 | Aynı öğrencinin aktif oda sayısı, host çıkışı ve kapanma? | Tek aktif oda; ürün kararı |
| A19 | Puzzle soru ekranında mini önizleme gösterilecek mi? | Sonuç ekranında tam açılım, soru ekranında yalnız ilerleme |
| A20 | Oda üyeliği ayrı tablo mu, join request ile mi tutulacak? | MVP'de join request; ölçek için ayrı üyelik tablosu değerlendirilir |

## 20.4 Kodlamadan önce uyumluluk kontrolü

- [ ] İsim ve klasörler Keşif olarak doğrulandı.
- [ ] Grup dışı katılım için A13 onaylandı.
- [ ] Oda erişim/TTL/onay kararları netleşti.
- [ ] PostgreSQL 19 tablo envanteri ve FK'ler gözden geçirildi.
- [ ] Mevcut Doping kimlik/Butik/soru bankası API sözleşmeleri alındı.
- [ ] REST ve WebSocket mesajları frontend ekranlarıyla eşlendi.
- [ ] JWT/Swagger güvenlik kuralları netleştirildi.
- [ ] İki tarayıcıyla kod, link, Butik ve 10 tur E2E test senaryoları onaylandı.
