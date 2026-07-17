# Minor Upgrade Checklist

## Planlama

- [ ] Hedef minor sürüm belirlendi
- [ ] Release notes incelendi
- [ ] Güvenlik ve bug fix etkileri değerlendirildi
- [ ] Bakım penceresi onaylandı
- [ ] Değişiklik kaydı açıldı
- [ ] Rollback adımları yazıldı

## Yedek

- [ ] Son full backup başarılı
- [ ] WAL arşivi sağlıklı
- [ ] Restore testi güncel
- [ ] Backup repository erişilebilir

## Envanter

- [ ] Mevcut PostgreSQL versiyonu kaydedildi
- [ ] Extension listesi kaydedildi
- [ ] Paket repository doğrulandı
- [ ] Patroni/HA topolojisi kaydedildi
- [ ] PgBouncer/HAProxy/Keepalived etkisi değerlendirildi

## Uygulama

- [ ] Replica'lar tek tek güncellendi
- [ ] Her node sonrası cluster sağlık kontrolü yapıldı
- [ ] Switchover planı uygulandı
- [ ] Eski leader güncellendi
- [ ] Beklenmeyen major paket kurulmadı

## Doğrulama

- [ ] `SELECT version()` beklenen sonucu veriyor
- [ ] Patroni cluster sağlıklı
- [ ] Replication lag kabul edilebilir
- [ ] Uygulama smoke test geçti
- [ ] Backup ve monitoring agent çalışıyor
- [ ] Loglarda yeni hata yok

## Kapanış

- [ ] Değişiklik kaydı güncellendi
- [ ] Ölçümler ve loglar eklendi
- [ ] Rollback penceresi kapatıldı
