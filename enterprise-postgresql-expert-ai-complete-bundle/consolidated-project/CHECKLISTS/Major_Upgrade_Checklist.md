# Major Upgrade Checklist

## Planlama

- [ ] Kaynak ve hedef PostgreSQL sürümleri belirlendi
- [ ] Upgrade yöntemi seçildi
- [ ] Bakım penceresi onaylandı
- [ ] RTO ve RPO tanımlandı
- [ ] Rollback kararı ve son karar zamanı belirlendi

## Uyumluluk

- [ ] Extension listesi çıkarıldı
- [ ] Hedef extension paketleri doğrulandı
- [ ] Uygulama driver/ORM test edildi
- [ ] Deprecated parametreler kontrol edildi
- [ ] Collation/ICU etkisi kontrol edildi

## Veri Güvenliği

- [ ] Güncel backup var
- [ ] Restore testi başarılı
- [ ] WAL archive sağlıklı
- [ ] Disk kapasitesi yeterli
- [ ] Link mode kullanılıyorsa rollback riski kabul edildi

## Test

- [ ] `pg_upgrade --check` başarılı
- [ ] Staging provası tamamlandı
- [ ] Süre ölçüldü
- [ ] Kritik sorgular karşılaştırıldı
- [ ] Failover/switchover testi yapıldı

## Uygulama

- [ ] Yazma trafiği durduruldu
- [ ] Son checkpoint/backup tamamlandı
- [ ] Upgrade logları saklandı
- [ ] PostgreSQL açılışı doğrulandı
- [ ] Extension update yapıldı
- [ ] ANALYZE tamamlandı

## Sonrası

- [ ] Uygulama smoke test başarılı
- [ ] Replica streaming
- [ ] Monitoring normal
- [ ] Backup zinciri yeniden başlatıldı
- [ ] Performans baseline ile karşılaştırıldı
- [ ] Rollback penceresi iş onayıyla kapatıldı
