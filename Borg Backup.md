#SRC #Sem1 #ZSK 

- Deduplikacja na poziomie fragmentów plików (_chunks_)
- Kompresja danych (różne algorytmy: zlib, lz4, lzma, zstd)
- Możliwość szyfrowania repozytorium
- Wbudowany mechanizm przedawniania ekspotencjalnego
		`brog prune ... --keep-daily=7 --keep-weekly=8 --keep-monthly=12`
- Rozmiary _chunks_: 512 KiB : 8 MiB
- Interfejs konsolowy i brak klienta dla Windows