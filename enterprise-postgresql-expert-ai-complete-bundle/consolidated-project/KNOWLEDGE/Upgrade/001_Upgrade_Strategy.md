# PostgreSQL Upgrade Strategy

## Amaç

PostgreSQL sürüm yükseltmesini veri kaybı, beklenmeyen kesinti ve geri dönüşsüz değişiklik riskini azaltarak planlamak.

## Minor ve Major Upgrade

### Minor upgrade

Örnek: 16.2 → 16.8. Aynı major seri içindedir. Genel olarak binary/package güncellemesi ve restart gerekir. Yine de release notes, extension binary uyumu ve rollback paketi kontrol edilmelidir.

### Major upgrade

Örnek: 15 → 16 veya 16 → 17. Veri dizini doğrudan hedef binary ile açılamaz. Yaygın yöntemler:

- `pg_upgrade`
- dump/restore
- logical replication ile blue/green geçiş

## Karar Matrisi

| Yöntem | Kesinti | Ek disk | Geri dönüş | Büyük veride uygunluk |
|---|---:|---:|---:|---:|
| pg_upgrade copy | Orta | Yüksek | Daha kolay | İyi |
| pg_upgrade link | Düşük | Düşük | Kritik/dikkatli | Çok iyi |
| Dump/restore | Yüksek | Orta | Kolay | Küçük/orta |
| Logical replication | Çok düşük | Yüksek | İyi | Çok iyi |

## Ön Kontroller

```bash
psql -Atc "select version();"
psql -Atc "select extname, extversion from pg_extension order by 1;"
```

Kontrol listesi:

- Hedef sürüm destekleniyor mu?
- Extension'ların hedef sürüm binary paketleri hazır mı?
- Disk kapasitesi yeterli mi?
- Backup restore testi başarıyla yapıldı mı?
- Uygulama driver ve ORM uyumlu mu?
- Collation/ICU davranışı değişiyor mu?
- Deprecated parametreler var mı?
- Patroni ve DCS bileşenleri hedef PostgreSQL sürümünü destekliyor mu?

## pg_upgrade Ön Kontrolü

```bash
/path/to/new/bin/pg_upgrade \
  --old-bindir=/path/to/old/bin \
  --new-bindir=/path/to/new/bin \
  --old-datadir=/data/old \
  --new-datadir=/data/new \
  --check
```

`--check` gerçek geçişten önce uyumsuzlukları yakalamak için zorunlu kabul edilmelidir.

## Link Mode Riski

`pg_upgrade --link`, eski ve yeni cluster dosyalarını hard link ile ilişkilendirir. Yeni cluster başlatıldıktan sonra eski cluster üzerinde yazma yapılması geri dönüşü bozabilir. Bu yöntem yalnızca açık geri dönüş planıyla kullanılmalıdır.

## Upgrade Sonrası

- Extension update
- `ANALYZE`
- Kritik sorgular için plan karşılaştırması
- Replica yeniden kurulum yöntemi
- Backup zincirini yeniden başlatma
- Monitoring dashboard ve alert doğrulaması

Örnek:

```sql
ALTER EXTENSION pg_stat_statements UPDATE;
ANALYZE;
```

## Başarı Kriterleri

- Uygulama bağlantısı başarılı
- Read/write testleri başarılı
- Kritik sorgular kabul edilebilir sürede
- Replica streaming
- Backup alınabiliyor ve doğrulanıyor
- Hata logları temiz
- Rollback penceresi kapanmadan iş onayı alınmış
