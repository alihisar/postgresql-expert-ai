# PostgreSQL Minor Upgrade

## Tanım

Minor upgrade aynı major sürüm ailesi içindeki düzeltme sürümüne geçiştir. Genellikle katalog formatı değişmez; yine de binary değişimi, servis restartı ve extension/packaging etkileri vardır.

## Neden önemlidir?

Minor sürümler güvenlik, veri bütünlüğü, crash ve replication düzeltmeleri içerebilir. "Sadece patch" diyerek plansız uygulanmamalıdır.

## Ön koşullar

- güncel ve doğrulanmış yedek
- restore testinin geçmiş olması
- release note incelemesi
- extension ve işletim sistemi paket uyumluluğu
- HA topolojisi ve restart sırası
- rollback yöntemi
- bakım penceresi veya rolling plan

## Standalone yaklaşım

1. Servis durumunu ve versiyonu kaydet.
2. Yedek ve restore kanıtını doğrula.
3. Paketleri güncelle.
4. PostgreSQL'i kontrollü restart et.
5. log, extension, bağlantı ve temel sorguları doğrula.

## Patroni yaklaşımı

Genel prensip:

1. Replica'ları tek tek güncelle.
2. Her node sonrası cluster sağlığını doğrula.
3. Uygun zamanda switchover yap.
4. Eski leader'ı güncelle.
5. Gerekirse rolü geri taşı.

Patroni'nin kendisi ile PostgreSQL paket yükseltmesi aynı değişiklik değildir; ikisinin sürüm notları ayrı değerlendirilir.

## Riskler

- yanlış repository'den beklenmeyen major paket kurulması
- extension shared library uyumsuzluğu
- restart sonrası eski config ile açılmama
- standby'ın yanlış binary ile başlaması
- HAProxy veya VIP katmanında trafik kesintisi

## Öncesi kontroller

```sql
SELECT version();
SELECT extname, extversion FROM pg_extension ORDER BY 1;
SELECT datname, datallowconn FROM pg_database ORDER BY 1;
SELECT * FROM pg_stat_replication;
```

```bash
patronictl list
systemctl status postgresql
journalctl -u postgresql --since '-30 min'
```

## Sonrası kontroller

```sql
SELECT version();
SELECT now();
SELECT count(*) FROM pg_stat_activity;
```

- uygulama smoke test
- replication lag
- WAL archive
- backup agent
- monitoring agent
- connection pool

## Rollback

Minor upgrade rollback, paket yöneticisi ve binary uyumluluğuna bağlıdır. Eski binary'ye dönmek teorik olarak mümkün görünse bile release note ve packaging davranışı doğrulanmadan uygulanmamalıdır.

## Sonuç

Minor upgrade küçük kapsamlı olabilir; fakat üretimde değişiklik yönetimi, doğrulama ve HA sırası olmadan güvenli değildir.
