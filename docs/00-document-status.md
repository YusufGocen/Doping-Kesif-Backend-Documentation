# Doküman Hakkında



# Dokümanın statüsü

Bu doküman, daha önce kararlaştırılmış oyun kurallarını teknik tasarıma dönüştürür. Doping Hafıza / Doping Butik servislerinin gerçek şemaları, kimlik doğrulama yöntemi ve soru bankası sözleşmeleri paylaşılmadığı için entegrasyon ayrıntıları öneri niteliğindedir. Kodlamadan önce ilgili ekip tarafından doğrulanmalıdır.

| **Etiket** | **Anlam**                                                 |
|------------|-----------------------------------------------------------|
| KESİN      | Önceki ürün görüşmelerinde netleştirilen oyun gereksinimi |
| ÖNERİ      | Backend geliştirmesi için önerilen uygulama tercihi       |
| AÇIK       | Ürün / mevcut platform ekibiyle karara bağlanacak konu    |

# İçindekiler

1\. Kapsam, roller ve kabul kriterleri

2\. Oyun akışı ve durum makineleri

3\. Mimari ve entegrasyon sınırları

4\. PostgreSQL veri modeli ve tablo sözlüğü

5\. ER diyagramı ve ilişkiler

6\. Java domain modeli ve katmanlar

7\. REST API sözleşmeleri

8\. WebSocket protokolü

9\. Oyun motoru ve transaction kuralları

10\. Güvenlik, gizlilik ve veri saklama

11\. Hata senaryoları, test ve performans

12\. Migration, geliştirme planı ve açık kararlar

Ek A. Örnek SQL DDL

Ek B. Örnek JSON mesajları

## 0.1 Güncel kapsam (23 Ekim 2026)

Projenin güncel adı **Doping Keşif**. Yeni ürün ekranlarında **Butik arkadaş daveti** yanında **oda kodu ve davet linkiyle katılım** vardır. Önceki yalnız Butik grubu şartı artık sadece doğrudan Butik davet yöntemi için geçerlidir. Kod/link ile farklı gruptan katılım **A13 onayına** tabidir. Teknik tasarım için [16–20](16-rooms-codes-and-invite-links.md) belgeleri güncel önceliklidir.
