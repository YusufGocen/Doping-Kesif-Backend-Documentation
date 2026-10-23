# 2. Oyun akışı ve durum makineleri

## 2.1 Uçtan uca kullanıcı akışı

1.  Kimlik doğrulama → öğrenci görev/sınıf/ders/konu seçer; Butik arkadaşı daveti veya kod/link ile oda yöntemini belirler.

2.  Butik yönteminde uygun arkadaşı seçip davet gönder; kod/link yönteminde oda oluşturup kodu/linki paylaş veya mevcut odaya katıl.

3.  Butik daveti kabul edilir veya geçerli kod/link ile katılım tamamlanır; iki öğrenci lobiye girer.

4.  İki oyuncu READY olur → sunucu 10 soruyu seçip oyun snapshot’ını kaydeder.

5.  Sunucu ilk soruyu iki oyuncuya gönderir; her oyuncu kendi cevabını verir.

6.  Her iki cevap gelince (veya süre dolunca) doğru cevap ve açılan parça bilgisi açıklanır.

7.  Puzzle sonuç aşaması gösterilir → sonraki soru; toplam 10 tur.

8.  Final hesaplamaları tek sefer yapılır → maç geçmişi, koleksiyon ve varsa XP güncellenir.

## 2.2 Davet durumları

> PENDING → ACCEPTED → (lobi oluşturuldu)
>
> PENDING → REJECTED \| CANCELLED \| EXPIRED

ÖNERİ: Davetler 2 dakika sonra EXPIRED olabilir. Davet kabulü, davetli öğrenci tarafından ve geçerlilik süresi içinde yapılmalıdır. Davet kabulü tek transaction’da yürütülmeli; aynı davetin çift kabulü engellenmelidir.

## 2.3 Maç durumları

> WAITING_FOR_PLAYERS → READY → IN_PROGRESS → FINISHED
>
> ↘ CANCELLED ↘ ABORTED

WAITING_FOR_PLAYERS: lobi açıldı. READY: iki oyuncu hazır, soru seti sabitlendi. IN_PROGRESS: turlar işleniyor. FINISHED: 10 tur tamamlandı. CANCELLED: başlamadan iptal. ABORTED: kalıcı bağlantı/sistem sorunu. ÖNERİ: kullanıcı çıkışında önce reconnect grace period uygulanır.

## 2.4 Tur durumları

> PENDING → QUESTION_OPEN → REVEAL → COMPLETED
>
> ↘ TIMEOUT → REVEAL

ÖNERİ: Her tur için sunucu kaynaklı expires_at tutulur; varsayılan 60 saniye yalnızca örnek değerdir. Bir oyuncu cevap verip diğeri vermezse süre bitiminde eksik cevap yanlış/boş kabul edilir. Doğru cevap REVEAL durumundan önce gönderilmez.

## 2.5 Parça açılma kuralı

| **A yanıtı** | **B yanıtı** | **Açılan parça** | **Toplam değişim** |
|--------------|--------------|------------------|--------------------|
| Doğru        | Doğru        | 2                | +2                 |
| Doğru        | Yanlış/boş   | 1                | +1                 |
| Yanlış/boş   | Doğru        | 1                | +1                 |
| Yanlış/boş   | Yanlış/boş   | 0                | +0                 |

*KARAR / NOT: Parçaların hangi indekslerle açılacağı deterministik ve kalıcı olmalı. Öneri: maç başında 1–20 parça indekslerini rastgele karıştırıp maç kaydına yazmak; her doğru cevapta sıradaki kapalı parçayı açmak.*
