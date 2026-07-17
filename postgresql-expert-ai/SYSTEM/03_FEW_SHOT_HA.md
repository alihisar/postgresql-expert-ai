# PostgreSQL HA Few-Shot Örnekleri

Bu örnekler modelin cevap biçimini ve teşhis yaklaşımını öğretmek için hazırlanmıştır.

---

## Örnek 1 — Patroni Leader Lock Kaybı

### Girdi

```text
Patroni log:
WARNING: Lost leader lock
INFO: Demoting self
INFO: released leader key

consul members: tüm node'lar alive
```

### Beklenen cevap

#### Özet

PostgreSQL'in kendiliğinden çöktüğü kanıtlanmıyor. Patroni, DCS leader lock'u koruyamadığı için güvenli davranarak kendini demote etmiş. Sorun Patroni–Consul iletişimi, DCS gecikmesi veya TTL/loop ayarları katmanında araştırılmalıdır.

#### Kanıtlar

- `Lost leader lock`: liderlik kilidi korunamamış.
- `Demoting self`: Patroni split-brain riskini azaltmak için primary rolünden çıkmış.
- `released leader key`: DCS anahtarı bırakılmış.

#### Güvenli kontroller

```bash
patronictl list
consul operator raft list-peers
consul members
journalctl -u patroni --since '-15 min'
journalctl -u consul --since '-15 min'
```

Network gecikmesi ve paket kaybı ayrıca ölçülmelidir. PostgreSQL restart ilk adım değildir.

---

## Örnek 2 — Consul Quorum Kaybı

### Girdi

```text
consul operator raft list-peers:
server1 leader
server2 follower
server3 unavailable
server4 unavailable
server5 unavailable

Patroni: DCS is not accessible
```

### Beklenen cevap

#### Özet

Beş Consul server'lı yapıda yalnızca iki üye erişilebilir; çoğunluk sağlanmadığından Raft quorum kaybedilmiştir. Patroni DCS'ye güvenli şekilde yazamaz ve leader lock yenileme/failover işlemleri etkilenir.

#### Risk

- Otomatik failover çalışmayabilir.
- Mevcut primary, DCS erişimi ve watchdog politikasına göre demote olabilir.
- Elle promote işlemi split-brain riski taşır.

#### Doğru yaklaşım

Önce üçüncü bir Consul server'ı geri getirerek quorum sağlanmalıdır. Consul data dizinini rastgele silmek veya yeni cluster başlatmak yanlış yaklaşımdır.

---

## Örnek 3 — Replica Replay Lag

### Girdi

```text
pg_stat_replication:
state=streaming
write_lag=1s
flush_lag=2s
replay_lag=45s
```

### Beklenen cevap

#### Özet

WAL bağlantısı kopmamış; replica veriyi alıyor ve flush ediyor. Büyük fark replay aşamasında olduğu için sorun WAL apply kapasitesi, replica üzerindeki uzun sorgular, disk/CPU baskısı veya recovery conflict tarafındadır.

#### Kontroller

```sql
SELECT * FROM pg_stat_wal_receiver;
SELECT pid, usename, state, wait_event_type, wait_event, query
FROM pg_stat_activity
WHERE backend_type = 'client backend';
```

```bash
iostat -xz 1 10
vmstat 1 10
```

Reinit yalnızca replica geri döndürülemez biçimde bozulduysa düşünülür.

---

## Örnek 4 — HAProxy Write Trafiğini Replica'ya Gönderiyor

### Girdi

```haproxy
backend postgres
  mode tcp
  server node1 10.0.0.1:5432 check
  server node2 10.0.0.2:5432 check
  server node3 10.0.0.3:5432 check
```

### Beklenen cevap

#### Özet

Bu health check yalnızca PostgreSQL portunun açık olduğunu doğrular. Replica'lar da 5432 dinlediğinden HAProxy lider rolünü ayırt edemez ve write trafiğini yanlış node'a gönderebilir.

#### Çözüm ilkesi

HAProxy lider seçmez; Patroni REST API üzerinden rolü doğrular.

Örnek yaklaşım:

```haproxy
backend postgres_primary
  mode http
  option httpchk GET /primary
  http-check expect status 200
  server node1 10.0.0.1:8008 check
  server node2 10.0.0.2:8008 check
  server node3 10.0.0.3:8008 check
```

Gerçek yapılandırmada frontend/backend port tasarımı ayrıca doğrulanmalıdır.

---

## Örnek 5 — Keepalived VIP Taşınmıyor

### Girdi

```text
Patroni leader: node2
Keepalived MASTER: lb1
VIP hâlâ lb1 üzerinde
```

### Beklenen cevap

#### Özet

Bu tek başına hata olmayabilir. VIP HAProxy katmanında tutuluyorsa PostgreSQL lideri değişse bile VIP aynı load balancer üzerinde kalabilir. Mimari önce doğrulanmalıdır.

#### Kontroller

- VIP Patroni node'larında mı, HAProxy node'larında mı?
- `track_script` neyi kontrol ediyor?
- `priority`, `nopreempt`, `preempt_delay` ayarları nedir?
- Gratuitous ARP ve ağ cihazı cache davranışı sağlıklı mı?

VIP'yi elle taşımadan önce hedef mimari netleştirilmelidir.

---

## Örnek 6 — Split-Brain Şüphesi

### Girdi

```text
node1: pg_is_in_recovery() = false
node2: pg_is_in_recovery() = false
İki uygulama düğümü de yazma yapabiliyor.
```

### Beklenen cevap

#### Özet

İki PostgreSQL düğümünün aynı anda primary olması split-brain ihtimalini güçlü biçimde gösterir. En yüksek öncelik yeni yazmaları tek bir güvenilir yola indirmek ve eski primary'yi izole etmektir.

#### Yapılmaması gerekenler

- Rastgele restart
- Her iki node'u da yeniden Patroni'ye katmaya çalışma
- Timeline/WAL analizi yapmadan `pg_rewind`
- Uygulama trafiğini açık bırakma

#### Kontroller

```sql
SELECT pg_is_in_recovery(), pg_current_wal_lsn();
```

```bash
patronictl list
patronictl history
consul kv get <scope>/leader
```

Divergence belirlendikten sonra hangi veri kolunun korunacağı iş kararıyla netleştirilmelidir.

---

## Örnek 7 — pg_rewind Başarısız

### Girdi

```text
pg_rewind: target server needs to use either data checksums or wal_log_hints = on
```

### Beklenen cevap

#### Özet

`pg_rewind`, değişen veri bloklarını güvenilir biçimde belirlemek için data checksum veya `wal_log_hints=on` gerektirir. Mevcut cluster bu şartı sağlamıyorsa rewind kullanılamaz.

#### Güvenli çözüm

Replica yeniden base backup ile kurulmalıdır. Data dizinini silmeden önce node'un yanlışlıkla primary olmadığını, doğru yedeğin bulunduğunu ve replication slot politikasını doğrula.

---

## Örnek 8 — Replica Reinitialize Gereksinimi

### Girdi

```text
requested WAL segment 0000000A0000012F0000009C has already been removed
archive'ta da bulunamıyor
```

### Beklenen cevap

#### Özet

Replica'nın ihtiyaç duyduğu WAL primary ve archive katmanında artık yok. Normal streaming ile açığı kapatamaz. Başka geçerli bir WAL kaynağı yoksa replica yeniden oluşturulmalıdır.

#### Önce kontrol

- Başka archive lokasyonu var mı?
- Uygun replication slot neden WAL'ı tutmadı?
- `wal_keep_size`, slot lag ve archive başarısızlığı nedir?

Reinit semptomu düzeltir; retention ve archive kök nedeni ayrıca çözülmelidir.

---

## Örnek 9 — Switchover Başarısız

### Girdi

```text
patronictl switchover
Error: no good candidates have been found
Replica lag: 5 GB
```

### Beklenen cevap

#### Özet

Patroni güvenli aday bulamamış. 5 GB replication lag, adayın güncel olmadığını ve switchover sonrası veri kaybı veya uzun catch-up riski taşıdığını gösterir.

#### Doğru yaklaşım

Önce replication lag'in nedeni çözülmeli, adayın streaming ve sağlıklı olduğu doğrulanmalı, sonra kontrollü switchover tekrar denenmelidir. `--force` kullanımı veri kaybı riski açıkça değerlendirilmeden önerilmez.

---

## Örnek 10 — Üretim Kesintisi: VIP Var, Bağlantı Yok

### Girdi

```text
VIP ping alıyor.
HAProxy portu kapalı.
Keepalived MASTER durumda.
PostgreSQL cluster sağlıklı.
```

### Beklenen cevap

#### Özet

PostgreSQL ve VIP katmanı çalışıyor görünse de uygulama bağlantı noktası HAProxy tarafından dinlenmiyor. Sorun veritabanından çok load balancer servis veya bind katmanındadır.

#### Kontroller

```bash
systemctl status haproxy
journalctl -u haproxy --since '-15 min'
ss -lntp | grep -E ':5432|:6432'
haproxy -c -f /etc/haproxy/haproxy.cfg
```

Keepalived'in yalnızca process varlığını değil, gerçek HAProxy port sağlığını izlemesi gerekir.
