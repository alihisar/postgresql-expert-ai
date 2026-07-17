# Blue/Green ve Logical Replication ile Major Upgrade

## Amaç

Yeni PostgreSQL major sürümünde ayrı bir green cluster kurup veriyi logical replication ile taşıyarak kesintiyi azaltmak.

## Ne zaman uygundur?

- düşük kesinti hedefi
- platform veya işletim sistemi değişimi
- major sürüm geçişiyle birlikte mimari dönüşüm
- rollback için eski cluster'ı bir süre koruma ihtiyacı

## Sınırlamalar

Logical replication tüm nesneleri otomatik taşımaz. Şunlar ayrıca yönetilmelidir:

- DDL
- sequence değerleri
- roller ve yetkiler
- extension'lar
- large object davranışı
- bazı partition ve replica identity senaryoları
- unlogged tablolar

## Yüksek seviye akış

1. Green cluster'ı kur.
2. Şema ve rollerin kontrollü kopyasını oluştur.
3. Publication/subscription kur.
4. Initial sync'i tamamla.
5. Lag'i izle.
6. Cutover öncesi write freeze uygula.
7. Son lag sıfıra yaklaşınca sequence ve final delta doğrula.
8. Uygulama bağlantısını green'e geçir.
9. Smoke test ve iş doğrulaması yap.
10. Rollback penceresi boyunca blue cluster'ı koru.

## Basit örnek

Blue cluster:

```sql
CREATE PUBLICATION upgrade_pub FOR ALL TABLES;
```

Green cluster:

```sql
CREATE SUBSCRIPTION upgrade_sub
CONNECTION 'host=blue-db port=5432 dbname=app user=repl password=...'
PUBLICATION upgrade_pub;
```

Gerçek üretimde SSL, slot, ağ, parola yönetimi ve publication kapsamı ayrıca tasarlanır.

## Sequence doğrulama

Logical replication sequence değerlerini sürekli taşımayabilir. Cutover öncesi sequence'ler için özel eşitleme gerekir.

## DDL yönetimi

Logical replication DDL replikasyonu sağlamaz. Şema migration aracı veya kontrollü deployment süreci kullanılır.

## Rollback

Uygulama green'e yazmaya başladıktan sonra blue cluster otomatik olarak güncel kalmaz. Gerçek rollback için reverse replication veya kısa karar penceresi tasarlanmalıdır.

## Riskler

- eksik tablo publication kapsamı
- replica identity eksikliği
- sequence çakışması
- uzun initial copy
- yüksek WAL üretimi
- slot nedeniyle disk büyümesi
- cutover sonrası iki tarafa yazma

## Doğrulama

```sql
SELECT * FROM pg_stat_subscription;
SELECT * FROM pg_replication_slots;
```

- tablo sayısı ve satır örnekleri
- checksum/iş kuralı doğrulaması
- sequence değerleri
- uygulama read/write smoke test
