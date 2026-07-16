# Oracle 19c – DBA Notes

Osobný študijný repozitár zameraný na administráciu Oracle Database 19c. 
Cieľom je prepojiť teóriu (architektúra, správa, výkon) s praktickým 
overovaním na bežiacej inštancii 

## O tomto projekte

- Poznámky vznikajú postupne, podľa toho, ako prechádzam jednotlivé témy
- Ku každej téme sa snažím pridať aj praktické overenie (SQL queries spustené 
  na reálnej inštancii, nie len teória)
- Cieľ: pripraviť sa na prácu v oblasti DB administrácie / IT a mať 
  hmatateľný dôkaz vedomostí

## Štruktúra repozitára

| Priečinok | Obsah |
|---|---|
| `01-architecture/` | SGA, PGA, background procesy, inštancia vs databáza |
| `02-multitenant/` | CDB/PDB architektúra a správa |
| `03-storage/` | Tablespaces, datafiles, control files, undo/redo |
| `04-security/` | Používatelia, role, privilégiá, profily, audit |
| `05-backup-recovery/` | RMAN, flashback |
| `06-performance/` | AWR, tuning buffer cache, SQL tuning |
| `queries/` | Samostatné .sql skripty použité pri overovaní |
| `screenshots/` | Výstupy z reálnej inštancie ako dôkaz praktického overenia |

## Postup (checklist)

- [x] SGA – architektúra, komponenty, teória
- [ ] SGA – praktické overenie (V$SGA, V$SGAINFO, V$SGASTAT, hit ratios)
- [ ] PGA
- [ ] Background procesy
- [ ] CDB/PDB
- [ ] Tablespaces a storage
- [ ] Users, roles, privileges
- [ ] RMAN základy
- [ ] AWR reporty

## Prostredie

- Oracle Database 19c
- Pripojenie cez SQL*Plus / SQL Developer
- OS: Windows 11

## Poznámka

Toto repo je súčasť môjho prechodu z manuálnej/technickej práce smerom 
k IT a administratívnym rolám. Praktické projekty a poznámky slúžia ako 
doplnok k CV a portfóliu.
