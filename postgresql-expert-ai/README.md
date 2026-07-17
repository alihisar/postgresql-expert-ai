# PostgreSQL Expert AI Knowledge Base

Türkçe çalışan yerel bir PostgreSQL DBA/HA uzmanı oluşturmak için hazırlanmış modüler bilgi tabanı.

## Kapsam

- PostgreSQL mimarisi, performans, indeksler, MVCC, WAL, vacuum ve kilitler
- Fiziksel/logical replication, backup, PITR ve disaster recovery
- Patroni, Consul, HAProxy, Keepalived ve PgBouncer
- Minor/major upgrade, `pg_upgrade`, blue/green ve logical replication ile geçiş
- İzleme, güvenlik, Linux, kapasite planlama ve olay müdahalesi

## Önerilen kullanım

1. `SYSTEM/00_SYSTEM_PROMPT.md` dosyasını modelin system prompt alanına koy.
2. `SYSTEM/01_SHORT_RULES.md` dosyasını aynı promptun sonuna ekle.
3. `SYSTEM/02_OUTPUT_FORMAT.md` dosyasını ekle.
4. `SYSTEM/03_FEW_SHOT_HA.md` içinden 5-10 örneği bağlama dahil et.
5. `KNOWLEDGE/` ve `PLAYBOOKS/` klasörlerini RAG bilgi tabanı olarak indeksle.

## Ollama örneği

```dockerfile
FROM qwen3.5:14b

PARAMETER temperature 0.15
PARAMETER top_p 0.9
PARAMETER repeat_penalty 1.1
PARAMETER num_ctx 4096

SYSTEM """
00_SYSTEM_PROMPT.md + 01_SHORT_RULES.md + 02_OUTPUT_FORMAT.md içeriği
"""
```

16 GB RAM'de 14B model sıkışırsa 7B-8B Q4 model ve 4096 context daha dengeli olur.

## Dizin yapısı

- `SYSTEM/`: model davranışı ve örnek cevaplar
- `KNOWLEDGE/`: konu anlatımları
- `PLAYBOOKS/`: üretim olaylarında uygulanabilir adımlar
- `CHECKLISTS/`: değişiklik ve bakım kontrol listeleri
- `LABS/`: laboratuvar senaryoları
- `EXAMPLES/`: örnek çıktı ve analizler

## Güvenlik

Bu depo üretim ortamında otomatik komut çalıştırmak için değil, karar desteği ve eğitim için tasarlanmıştır. Veri kaybı riski taşıyan işlemlerde insan onayı, güncel yedek ve geri dönüş planı zorunludur.
