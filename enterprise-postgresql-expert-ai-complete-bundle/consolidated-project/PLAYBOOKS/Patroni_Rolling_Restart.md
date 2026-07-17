# Patroni Rolling Restart Playbook

## Amaç

PostgreSQL veya Patroni restart gerektiren değişiklikleri cluster erişilebilirliğini koruyarak uygulamak.

## Ön koşullar

- en az bir sağlıklı replica
- replication lag kabul edilebilir
- DCS quorum sağlıklı
- HAProxy backend kontrolleri doğru
- bakım ve rollback planı
- uygulama retry davranışı doğrulanmış

## Başlangıç kontrolleri

```bash
patronictl list
patronictl show-config
consul operator raft list-peers
```

```sql
SELECT application_name, state, sync_state,
       write_lag, flush_lag, replay_lag
FROM pg_stat_replication;
```

## Adımlar

1. Cluster'ın tek leader ve sağlıklı replica durumunu doğrula.
2. İlk replica'yı load balancing'den çıkar veya `noloadbalance` politikasıyla izole et.
3. Replica'yı kontrollü restart et.
4. Tekrar `running` ve `streaming` olmasını bekle.
5. Lag normale dönünce trafiğe ekle.
6. Diğer replica'lar için tekrarla.
7. Gerekliyse kontrollü switchover yap.
8. Eski leader'ı restart et.
9. Cluster'ın yeni durumda dengelendiğini doğrula.

## Patroni komutu

Ortam ve sürüme göre `patronictl restart` seçenekleri değişebilir. Komut çalıştırılmadan önce yardım çıktısı kontrol edilir:

```bash
patronictl restart --help
```

## Durdurma kriterleri

- replica recovery'ye dönemiyor
- DCS quorum bozuluyor
- HAProxy iki primary görüyor
- replication lag hızla artıyor
- WAL arşivi başarısız

Bu durumda sonraki node'a geçilmez.

## Doğrulama

```bash
patronictl list
curl -s http://node1:8008/health
curl -s http://node2:8008/health
```

```sql
SELECT pg_is_in_recovery();
SELECT * FROM pg_stat_replication;
```

## Rollback

- son değişikliği geri al
- yalnızca başarısız node'u eski config/binary ile ayağa kaldır
- diğer sağlıklı node'lara dokunma
- gerekirse planlı switchover ile eski rol dağılımına dön
