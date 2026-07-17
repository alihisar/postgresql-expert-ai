# PostgreSQL Mimarisi — Senior DBA Temeli

## Amaç

Bu belge PostgreSQL'in process, bellek, disk ve WAL mimarisini operasyonel bakışla açıklar. Amaç yalnızca bileşenleri ezberlemek değil, bir arıza veya performans sorununun hangi katmanda oluştuğunu ayırabilmektir.

## Process modeli

PostgreSQL geleneksel olarak process tabanlı çalışır. Ana `postmaster`/`postgres` süreci bağlantıları kabul eder ve her istemci için ayrı backend process oluşturur. Bunun yanında checkpointer, background writer, WAL writer, autovacuum launcher ve worker processleri bulunur.

### DBA için anlamı

- Çok yüksek bağlantı sayısı process, bellek ve context-switch yükünü artırır.
- Connection pool olmadan binlerce doğrudan bağlantı genellikle kötü bir tasarımdır.
- Tek bir backend'in yüksek CPU tüketmesi yalnızca SQL problemine değil, fonksiyon, JIT, sort veya hash işlemine de işaret edebilir.

## Bellek katmanları

### Shared memory

- `shared_buffers`
- WAL buffers
- lock tabloları
- processler arası paylaşılan kontrol yapıları

### Process başına bellek

- `work_mem`
- sort ve hash alanları
- execution context
- prepared statement ve plan yapıları

`work_mem` bağlantı başına sabit ayrılan bir havuz değildir. Bir sorgu içindeki birden fazla sort/hash düğümü ve eş zamanlı sorgular belleği katlayabilir.

## Veri sayfaları

PostgreSQL tabloları ve indeksleri bloklar hâlinde okur. Varsayılan blok boyutu çoğu kurulumda 8 KB'dır. `EXPLAIN (ANALYZE, BUFFERS)` çıktısındaki blok sayıları yaklaşık veri hacmine çevrilebilir.

```text
50.000 blok × 8 KB ≈ 390 MB
```

Bu hesap dosya düzeyinde yaklaşık I/O miktarını verir; kullanıcı satır boyutu ile birebir aynı değildir.

## WAL

Write-Ahead Logging kuralı, veri sayfası kalıcı yazılmadan önce ilgili değişikliğin WAL kaydının güvenli konuma yazılmasını gerektirir.

WAL şu alanlarda merkezîdir:

- crash recovery
- streaming replication
- PITR
- logical decoding
- timeline ve failover

### Operasyonel sonuç

- WAL arşivi bozuksa PITR zinciri kırılır.
- Replication slot tüketilmeyen WAL'ı tutarak diski doldurabilir.
- Çok yoğun checkpoint WAL ve veri dosyası I/O'sunu artırabilir.

## Checkpointer ve background writer

### Checkpointer

Checkpoint sırasında dirty page'lerin kalıcı yazımı organize edilir ve recovery başlangıç noktası ilerletilir.

### Background writer

Backend'lerin doğrudan dirty buffer yazma ihtiyacını azaltmaya çalışır.

### Teşhis sinyalleri

- sık checkpoint
- `checkpoint_warning`
- yüksek write latency
- `buffers_backend` artışı
- yüksek WAL üretimi

## MVCC

PostgreSQL satırları yerinde ezmek yerine yeni tuple sürümleri üretir. Eski sürümler görünürlük kurallarına göre tutulur ve vacuum tarafından temizlenebilir.

Sonuçları:

- okuyucu ve yazıcılar çoğu durumda birbirini engellemez,
- uzun transaction'lar dead tuple temizliğini geciktirebilir,
- autovacuum hayati önemdedir,
- transaction ID wraparound riski vardır.

## Visibility Map ve Index Only Scan

Index Only Scan'in heap erişimini tamamen atlayabilmesi için ilgili sayfaların all-visible olması gerekir. Yüksek `Heap Fetches` değeri vacuum veya yoğun değişiklikler nedeniyle visibility map avantajının kaybolduğunu gösterebilir.

## FSM ve TOAST

### Free Space Map

Sayfalardaki boş alanı takip eder ve yeni tuple yerleşimini kolaylaştırır.

### TOAST

Büyük text, JSONB veya bytea değerlerini sıkıştırır ya da tablo dışına taşır. `SELECT *` gereksiz TOAST erişimi ve ağ trafiği yaratabilir.

## DBA teşhis matrisi

| Belirti | İlk katman |
|---|---|
| Çok bağlantı ve yüksek context switch | Process modeli / pool |
| Temp file artışı | work_mem / sort / hash |
| WAL dizini büyüyor | slot / archive / write yükü |
| Heap Fetches yüksek | visibility map / vacuum |
| Checkpoint spike | WAL ve checkpoint ayarları |
| Disk doluyor ama tablo büyümüyor | WAL, temp, log, slot |

## Doğrulama komutları

```sql
SELECT version();
SHOW shared_buffers;
SHOW work_mem;
SHOW wal_level;
SHOW checkpoint_timeout;
SHOW max_wal_size;
```

```sql
SELECT * FROM pg_stat_bgwriter;
SELECT * FROM pg_stat_wal;
SELECT * FROM pg_stat_activity;
```

## Sık yapılan hata

Bir PostgreSQL problemini yalnızca SQL, yalnızca Linux veya yalnızca disk sorunu olarak ele almak. Senior DBA önce katmanları ayırır, sonra kanıt toplar.
