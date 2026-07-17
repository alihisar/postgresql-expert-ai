# Consul Quorum Kaybı Playbook

## Amaç

Patroni'nin DCS olarak kullandığı Consul cluster'ında quorum kaybını split-brain oluşturmadan yönetmek.

## Belirtiler

- `DCS is not accessible`
- `no cluster leader`
- Patroni leader lock yenilenemiyor
- failover gerçekleşmiyor
- Consul write işlemleri başarısız

## İlk risk değerlendirmesi

- Aynı anda kaç PostgreSQL primary var?
- Uygulama yazma trafiği hangi node'a gidiyor?
- Watchdog/fencing var mı?
- Consul server çoğunluğu gerçekten kayıp mı?

## Kontroller

```bash
consul members
consul operator raft list-peers
consul info
journalctl -u consul --since '-30 min'
```

```bash
patronictl list
journalctl -u patroni --since '-30 min'
```

```sql
SELECT pg_is_in_recovery();
```

## Doğru müdahale

1. Yeni write'ları kontrol altına al.
2. Eski ve yeni primary ihtimalini dışla.
3. Mevcut Consul cluster'ın çoğunluğunu geri getir.
4. Ağ, disk ve saat senkronizasyonu sorunlarını düzelt.
5. Quorum geri geldikten sonra Patroni state'ini doğrula.
6. Trafiği ancak tek güvenilir leader doğrulandıktan sonra aç.

## Yapılmaması gerekenler

- Consul data dizinlerini rastgele silmek
- aynı isimle yeni, ayrı bir Consul cluster başlatmak
- Patroni node'larını elle promote etmek
- yalnızca tüm servisleri restart ederek çözüm beklemek

## Recovery notu

Quorum geri getirilemiyorsa Consul disaster recovery prosedürü uygulanır. Bu işlem mevcut Raft state'i, snapshot ve surviving server bilgisiyle yapılmalıdır; doğaçlama edilmemelidir.

## Doğrulama

```bash
consul operator raft list-peers
consul members
patronictl list
```

- tek Consul leader
- beklenen voter sayısı
- tek Patroni leader
- HAProxy write backend'inde tek UP primary
