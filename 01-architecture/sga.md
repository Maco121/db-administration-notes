# SGA (System Global Area)

## Teória

SGA je zdieľaná pamäťová oblasť alokovaná inštanciou pri štarte 
(STARTUP). Je spoločná pre všetky procesy pripojené k danej inštancii – 
na rozdiel od PGA, ktorá je privátna pre každý proces.

Veľkosť SGA riadi parameter `SGA_TARGET` (Automatic Shared Memory 
Management), prípadne `MEMORY_TARGET` ak sa automaticky prerozdeľuje 
pamäť medzi SGA a PGA dohromady.

## Komponenty

| Komponent | Funkcia |
|---|---|
| Shared Pool | Library Cache (parsed SQL, execution plány) + Data Dictionary Cache (metadáta) |
| Database Buffer Cache | Kópie dátových blokov z datafiles, LRU algoritmus |
| Redo Log Buffer | Redo záznamy pred zápisom do online redo log súborov |
| Large Pool | RMAN buffery, shared server session memory, paralelné dopyty |
| Java Pool | Java stored procedures |
| Streams Pool | Replikácia (Streams/GoldenGate), Advanced Queuing |

## Kľúčové súvislosti

- **Soft parse vs hard parse**: ak je SQL text už v Library Cache, 
  Oracle znovu použije execution plan (soft parse) namiesto nového 
  spracovania (hard parse) – dôvod, prečo sú bind variables dôležité
- **Write-ahead logging**: pri COMMIT zapíše LGWR redo log buffer na 
  disk okamžite, kým DBWn zapisuje zmenené (dirty) dátové bloky na 
  disk asynchrónne, neskôr – zaručuje durabilitu bez straty výkonu

## Praktické overenie

### Základný prehľad SGA

```sql
SELECT * FROM V$SGA;
SELECT * FROM V$SGAINFO;
SELECT pool, name, bytes FROM V$SGASTAT ORDER BY bytes DESC;
```

**Výstup:** 
| NAME | VALUE | CON_ID |
|---|---|---|
| Fixed Size | 9038960 | 0 |
| Variable Size | 922746880 | 0 |
| Database Buffers | 4177526784 | 0 |
| Redo Buffers | 7737344 | 0 |

**Interpretácia:** Variable Size (~880 MB) zahŕňa Shared Pool + Large Pool 
+ Java Pool dohromady. Database Buffers (~3,98 GB) je Buffer Cache – 
najväčšia časť SGA v tomto prípade. Pre presnejší rozpad Variable Size 
je potrebné V$SGAINFO alebo V$SGASTAT.


### Aktuálne parametre

```sql
SHOW PARAMETER sga;
SHOW PARAMETER db_cache_size;
SHOW PARAMETER shared_pool_size;
```

**Výstup:** [doplniť]

### Buffer Cache Hit Ratio

```sql
SELECT 
  1 - (phy.value / (cur.value + con.value)) AS hit_ratio
FROM V$SYSSTAT phy, V$SYSSTAT cur, V$SYSSTAT con
WHERE phy.name = 'physical reads'
  AND cur.name = 'db block gets'
  AND con.name = 'consistent gets';
```

**Výstup:** [doplniť]
**Interpretácia:** [čo hodnota znamená - nízky ratio = potreba viac RAM / zlé dopyty]

### Library Cache Hit Ratio

```sql
SELECT namespace, gethitratio FROM V$LIBRARYCACHE;
```

**Výstup:** [doplniť]

### Soft parse vs Hard parse demo

```sql
-- Bez bind variables - každé volanie = hard parse
SELECT * FROM student WHERE id_student = 1;
SELECT * FROM student WHERE id_student = 2;

-- Kontrola v library cache
SELECT sql_text, executions, parse_calls 
FROM V$SQL 
WHERE sql_text LIKE 'SELECT * FROM student WHERE id_student%';
```

**Výstup:** [doplniť]
**Záver:** [koľko samostatných záznamov vzniklo a prečo]

## Čo som sa naučil

[doplniť vlastnými slovami po dokončení praktickej časti]
