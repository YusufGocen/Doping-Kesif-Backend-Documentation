# 1. Kapsam, roller ve kabul kriterleri

## 1.1 Ürün tanımı

Doping Keşif, Doping Hafıza web platformuna entegre edilecek, Doping Butik grubundan davetle veya oda kodu/bağlantısı üzerinden eşleşen iki öğrencinin eş zamanlı olarak aynı 10 soruyu çözdüğü, doğru cevaplarla ortak 20 parçalık puzzle görselini açtığı iş birlikçi öğrenme deneyimidir. Öğrenciler birbirine karşı değil, ortak puzzle hedefi için oynar.

## 1.2 Sabit ürün kuralları

- KESİN — 5–8. sınıflar; ders ve uygun konu seçimi. Kullanıcıya ayrı zorluk seçimi gösterilmez.

- KESİN — İki giriş yöntemi vardır: (A) aynı Doping Butik grubundan arkadaş daveti; (B) oda kodu veya davet bağlantısı ile katılım. Kod/link ile grup dışından katılımın koşulları **A13 açık kararıdır**. Rastgele eşleşme yoktur.

- KESİN — Oyun iki kişiliktir; davet, kabul ve hazır olma akışı vardır.

- KESİN — İki oyuncu aynı anda aynı soruyu görür ve birbirinden bağımsız yanıtlar.

- KESİN — Bir oyun 10 sorudur; oyuncu başına en fazla 10 doğru cevap, toplam en fazla 20 açılan puzzle parçası.

- KESİN — Puzzle soru ekranında görünmez; cevap aşaması tamamlandıktan sonraki puzzle/sonuç ekranında parçalar açılır.

- KESİN — Sohbet ve rekabetçi liderlik tablosu yoktur; kişisel puzzle koleksiyonu bulunur.

- KESİN — Doping Hafıza mevcut beyaz temalı web arayüzü ve sol menüsü korunur (frontend gereksinimi).

## 1.3 Roller ve yetkiler

| **Rol**                       | **İzinler**                                                                                         | **Sınırlar**                                                                              |
|-------------------------------|-----------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| Öğrenci                       | Kendi grup arkadaşlarını görür, davet oluşturur/yanıtlar, kod/link ile oda oluşturur veya katılır, aktif oyunu oynar, koleksiyonunu inceler | Başka grubun verisini, sorunun doğru cevabını veya diğer oyuncunun cevabını erken göremez |
| Keşif backend                | Butik davetinde grup uygunluğu; kod/link katılımında oda erişim kuralları doğrular, soruları seçer, oyunu yönetir, cevapları puanlar                            | Kimlik bilgisini istemcinin gönderdiği userId’den güvenerek almaz                         |
| İçerik yöneticisi (opsiyonel) | Soru/puzzle yayınlama ve pasife alma                                                                | MVP’de yönetim ekranı zorunlu değildir                                                    |
| Mevcut Doping servisleri      | Kimlik, Butik üyeliği ve soru içerikleri için otorite                                               | Gerçek API sözleşmeleri henüz bilinmiyor                                                  |

## 1.4 MVP / kapsam dışı

MVP: grup arkadaşı listeleme, davet, lobi, soru seti oluşturma, 10 tur, cevap doğrulama, 20 parçalık puzzle, sonuç, XP/koleksiyon (karar verilirse), geçmiş, reconnect, yetki kontrolü. Kapsam dışı: rastgele eşleşme, sohbet, ses, 3+ oyuncu, sıralama tablosu, mobil-native uygulama.

## 1.5 Kabul kriterleri

- Butik arkadaşı davetinde aynı aktif grup üyeliği doğrulanır. Kod/link ile katılımda geçerli oda erişimi, oturum açmış kullanıcı, kapasite ve (etkinse) oda sahibi onayı doğrulanır.

- Her turda iki öğrenci aynı questionId ve seçenek sırasını alır; doğru cevap gönderilmez.

- Her oyuncu aynı turda en fazla bir geçerli yanıt verebilir; yinelenen istek ekstra puan oluşturmaz.

- İki yanıt veya sunucu zaman aşımı sonrasında tur kapanır; doğru sayısı kadar (0–2) parça açılır.

- 10 tur sonunda maç FINISHED olur; toplam açılan parça 0–20 arasında kalır.

- Koleksiyon ve ödül işlemleri aynı maç için tekrar yürütüldüğünde mükerrer kayıt oluşmaz.

- Yeniden bağlanan öğrenci yalnızca yetkili olduğu aktif maçın güncel durumunu görür.
