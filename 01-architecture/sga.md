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

| NAME | BYTES | RESIZEABLE |
|---|---|---|
| Fixed SGA Size | 9038960 | No |
| Redo Buffers | 7737344 | No |
| Buffer Cache Size | 4177526784 | Yes |
| Shared Pool Size | 905969664 | Yes |
| Large Pool Size | 16777216 | Yes |
| Java Pool Size | 0 | Yes |
| Streams Pool Size | 0 | Yes |
| Shared IO Pool Size | 134217728 | Yes |
| Maximum SGA Size | 5117049968 | No |

**Interpretácia:** V$SGAINFO potvrdzuje výpočet z V$SGA - Variable Size 
(922 746 880 B) sa presne rovná súčtu Shared Pool (905 969 664) + Large 
Pool (16 777 216) + Java Pool (0). Java Pool a Streams Pool sú nulové, 
lebo táto inštancia nevyužíva Java stored procedures ani replikáciu. 
Shared Pool (~864 MB) je najväčšia zložka Variable Size, čo zodpovedá 
tomu, že drží library cache aj data dictionary cache.

| Pool | Name | Bytes | Poznámka |
|---|---|---|---|
| (null) | buffer_cache | 4 043 309 056 | Skutočne využitý Buffer Cache |
| shared pool | free memory | 516 894 336 | Nevyužitá časť Shared Pool (~57%) |
| shared pool | SQLA | 38 909 264 | Library Cache – SQL Area (parsed príkazy) |
| shared pool | row cache mutex | 9 019 032 | Data Dictionary Cache |
| (null) | log_buffer | 7 737 344 | Redo Log Buffer |
| (null) | fixed_sga | 9 038 960 | Fixed SGA |

**Interpretácia:** V$SGASTAT rozbíja Shared Pool na desiatky vnútorných 
štruktúr (spolu 1400 riadkov). Najdôležitejšie pre pochopenie architektúry: 
"SQLA" a KGL* položky = Library Cache, "row cache" položky = Data Dictionary 
Cache. Voľná pamäť v shared pool (~517 MB z ~906 MB) naznačuje, že 
inštancia momentálne nie je pod väčšou záťažou.


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
