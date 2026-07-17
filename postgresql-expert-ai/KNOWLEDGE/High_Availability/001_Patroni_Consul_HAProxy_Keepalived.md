# Patroni + Consul + HAProxy + Keepalived Mimarisi

## Bileşenler

- Patroni: PostgreSQL rol yönetimi ve otomatik failover
- Consul: DCS, leader lock ve cluster koordinasyonu
- HAProxy: primary/replica endpoint yönlendirmesi
- Keepalived: HAProxy katmanı için VIP yüksek erişilebilirliği

## Normal Trafik Akışı

Uygulama → VIP → aktif HAProxy → Patroni REST health check sonucu → PostgreSQL primary

## Kritik Tasarım İlkeleri

- Consul server sayısı tek ve en az 3 olmalı.
- HAProxy write backend'i rol farkındalığı olan Patroni endpointini kullanmalı.
- Keepalived health script'i yalnızca process değil gerçek servis portunu doğrulamalı.
- Patroni node'ları arasında düşük gecikmeli ağ olmalı.
- Watchdog/fencing split-brain riskini azaltır.
- Asenkron replication sıfır veri kaybı garantisi vermez.

## Temel Kontroller

```bash
patronictl list
curl -s http://127.0.0.1:8008/patroni
consul members
consul operator raft list-peers
systemctl status haproxy keepalived
ip addr show
```

```sql
SELECT pg_is_in_recovery();
SELECT * FROM pg_stat_replication;
```
