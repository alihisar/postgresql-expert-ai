# Split-Brain Response Playbook

## Amaç

Aynı cluster'a ait birden fazla PostgreSQL düğümünün yazılabilir primary hâle gelmesi durumunda veri kaybını sınırlamak.

## İlk 5 Dakika

1. Uygulama yazma trafiğini durdur veya tek güvenilir endpoint'e indir.
2. Şüpheli eski primary'yi ağdan izole et.
3. HAProxy backendlerini doğrula.
4. VIP sahipliğini kontrol et.
5. Patroni ve DCS liderliğini doğrula.

## Kanıt Toplama

Her düğümde:

```sql
SELECT pg_is_in_recovery();
SELECT pg_current_wal_lsn();
SELECT timeline_id FROM pg_control_checkpoint();
```

`pg_control_checkpoint()` erişimi sürüme/izinlere bağlı olabilir; alternatif olarak `pg_controldata` kullan.

```bash
pg_controldata "$PGDATA"
patronictl list
patronictl history
consul operator raft list-peers
```

## Karar

Korunacak veri kolu iş sahibiyle belirlenir. En yüksek LSN tek başına her zaman doğru iş verisini garanti etmez; iki tarafta bağımsız commitler olabilir.

## Kurtarma

- Seçilen primary dışındaki düğümler yazmaya kapatılır.
- Diverged node için `pg_rewind` uygunluğu kontrol edilir.
- Uygun değilse base backup/reinit yapılır.
- HAProxy ve VIP yalnızca seçilen primary yoluna açılır.

## Yasaklar

- Her iki primary'yi aynı anda uygulamaya açma
- Timeline analizi yapmadan rewind/reinit
- WAL dosyalarını elle kopyalama/silme
- DCS state'i rastgele temizleme
