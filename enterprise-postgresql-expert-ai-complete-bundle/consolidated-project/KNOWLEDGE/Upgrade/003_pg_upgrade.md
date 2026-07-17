# pg_upgrade ile Major Sürüm Yükseltme

## Amaç

`pg_upgrade`, eski PostgreSQL major sürümünün veri dizinini yeni major sürüm formatına hızlı biçimde taşımak için kullanılır. Logical dump/restore'a göre çok daha kısa sürebilir; ancak extension, binary, tablespace ve rollback planı dikkatle hazırlanmalıdır.

## Modlar

### Copy mode

Yeni cluster dosyaları kopyalanır. Daha fazla disk ve süre gerekir, fakat eski cluster veri dosyaları bağımsız kalır.

### Link mode

Hard link kullanır. Hızlı ve düşük disk tüketimlidir; yeni cluster başlatıldıktan sonra eski cluster dosyaları güvenli rollback kaynağı olarak görülemez.

### Clone/reflink benzeri seçenekler

Dosya sistemi ve sürüm desteğine bağlıdır. Ortamda gerçekten desteklendiği doğrulanmalıdır.

## Temel süreç

1. Envanter çıkar.
2. Yeni binary ve extension paketlerini kur.
3. Yeni boş cluster oluştur.
4. Eski ve yeni config farklarını değerlendir.
5. `pg_upgrade --check` çalıştır.
6. Uygulamayı durdur veya write freeze uygula.
7. Eski cluster'ı temiz kapat.
8. Gerçek upgrade'i çalıştır.
9. Yeni cluster'ı başlat.
10. istatistikleri üret.
11. extension'ları yükselt.
12. uygulama ve veri doğrulaması yap.
13. eski cluster temizliğini ancak kabul sonrası yap.

## Kritik kontroller

- locale ve collation uyumluluğu
- encoding
- tablespace erişimi
- extension binary'leri
- preload library'ler
- replication slot ve logical decoding tasarımı
- PostGIS gibi extension zincirleri
- `postgresql.conf`, `pg_hba.conf`, SSL dosyaları
- backup/restore ve rollback

## Check örneği

```bash
/usr/pgsql-new/bin/pg_upgrade \
  --check \
  --old-bindir=/usr/pgsql-old/bin \
  --new-bindir=/usr/pgsql-new/bin \
  --old-datadir=/var/lib/pgsql/old/data \
  --new-datadir=/var/lib/pgsql/new/data
```

Yollar dağıtıma göre değişir.

## İstatistikler

Upgrade sonrası optimizer istatistiklerinin yeniden oluşturulması gerekir. `pg_upgrade` tarafından oluşturulan analiz yardımcı scriptleri veya kontrollü `vacuumdb --analyze-in-stages` yaklaşımı kullanılabilir.

## Extension yükseltme

```sql
SELECT extname, extversion FROM pg_extension ORDER BY 1;
```

Extension için yeni sürüm destekleniyorsa:

```sql
ALTER EXTENSION extension_name UPDATE;
```

Bu komut körlemesine toplu çalıştırılmaz; her extension'ın sürüm zinciri ve bağımlılıkları doğrulanır.

## Rollback ilkesi

- Copy mode: eski cluster bağımsız tutulduysa daha kolay geri dönüş sağlar.
- Link mode: yeni cluster başladıktan sonra eski veri dizinine dönmek güvenli olmayabilir.
- Uygulama yazma yaptıysa mantıksal divergence oluşur.

Rollback, upgrade komutundan önce yazılı ve test edilmiş olmalıdır.

## HA ortamı

`pg_upgrade` physical replica'ları otomatik olarak dönüştürmez. Replica stratejisi ayrıca planlanır:

- yeni base backup
- rsync tabanlı hızlandırma
- blue/green
- logical replication

Patroni cluster'da DCS, scope, bootstrap ve member state dikkatle yönetilir.

## Doğrulama

```sql
SELECT version();
SELECT count(*) FROM pg_database;
SELECT extname, extversion FROM pg_extension ORDER BY 1;
```

- satır sayısı örnekleri
- kritik iş sorguları
- sequence değerleri
- cron/job sistemleri
- logical replication
- backup
- monitoring

## Sık yapılan hatalar

- yalnızca `pg_upgrade --check` başarılı diye risk kalmadığını sanmak
- extension paketlerini unutmak
- eski config'i doğrudan kopyalamak
- link mode rollback riskini göz ardı etmek
- uygulama ve sequence doğrulaması yapmamak
