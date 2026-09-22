---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 54
---
# 54. Protokół Blockchain – zasada działania, przeznaczenie
---
> **Blockchain** (łańcuch bloków) to **rozproszony, replikowany rejestr** (_distributed ledger_) transakcji, utrzymywany przez sieć P2P węzłów bez zaufanej centralnej instytucji. Dane grupowane są w **bloki** połączone **wskaźnikami kryptograficznymi** (skrótami poprzednich bloków), więc zmiana historii jest **wykrywalna i praktycznie niemożliwa**. Kolejność bloków węzły uzgadniają **mechanizmem konsensusu** (np. Proof of Work), tolerującym nieuczciwych uczestników.

Pierwsza implementacja: **Bitcoin** – Satoshi Nakamoto, „Bitcoin: A Peer-to-Peer Electronic Cash System” (2008), sieć od stycznia 2009. Rozwiązał problem **podwójnego wydania** (_double spending_) cyfrowego pieniądza bez zaufanej strony trzeciej.

## Budowa
### Blok
```
┌──────────────────── Nagłówek bloku (80 bajtów w Bitcoinie) ────────────────────┐
│ wersja │ skrót poprzedniego bloku │ korzeń drzewa Merkle │ znacznik czasu │     │
│        │ (SHA-256²)                │ transakcji           │                │     │
│ trudność (nBits – cel)  │  nonce                                                 │
└───────────────────────────────────────────────────────────────────────────────┘
│ Lista transakcji (pierwsza: coinbase – nagroda dla górnika)                     │
└───────────────────────────────────────────────────────────────────────────────┘
```
### Łańcuch – wskaźniki skrótów
```
[Blok genesis] ◀── hash ── [Blok 1] ◀── hash ── [Blok 2] ◀── hash ── [Blok 3] ...
```
- Każdy nagłówek zawiera **skrót nagłówka poprzedniego bloku** → **wskaźnik z dowodem integralności** (_hash pointer_).
- Zmiana transakcji w bloku $k$ zmienia jego korzeń Merkle → skrót bloku $k$ → pole w bloku $k+1$ → skrót $k+1$ → … → **cały dalszy łańcuch**. Podmiana wymagałaby ponownego „wykopania” wszystkich kolejnych bloków szybciej niż reszta sieci (**niezmienność** w sensie probabilistycznym).

### Drzewo Merkle
- Liście = skróty transakcji; węzeł wewnętrzny = skrót konkatenacji skrótów dzieci; **korzeń** w nagłówku.
- **Dowód przynależności** transakcji w $O(\log n)$ skrótów → **lekkie klienty SPV** (_Simplified Payment Verification_) weryfikują płatności, pobierając tylko nagłówki (80 B na blok), bez całego łańcucha.

### Kryptografia
- **funkcje skrótu** – SHA-256 (Bitcoin), Keccak-256 (Ethereum): odporność na kolizje i przeciwobrazy,
- **podpisy cyfrowe** – ECDSA / Schnorr na krzywej **secp256k1**: **adres** (tożsamość) = skrót klucza publicznego; tylko posiadacz **klucza prywatnego** może wydać środki (utrata klucza = utrata środków),
- **pseudonimowość** – adresy nie są powiązane z tożsamością, ale wszystkie transakcje są **publiczne** (analiza grafu transakcji).

## Transakcje
### Model UTXO (Bitcoin)
- **UTXO** (_Unspent Transaction Output_) – niewydane wyjścia wcześniejszych transakcji; „saldo” adresu to suma jego UTXO.
- **Transakcja**:
  - **wejścia** – odwołania do UTXO (id transakcji + indeks wyjścia) + **podpis** (skrypt odblokowujący),
  - **wyjścia** – kwoty + **skrypty blokujące** (np. P2PKH: „wydać może posiadacz klucza o skrócie X”; multisig, timelocki),
  - suma wejść ≥ suma wyjść; **różnica = opłata** dla górnika; reszta wraca na własny adres jako nowe wyjście.
- **Walidacja**: poprawność podpisów i skryptów, wejścia istnieją i **nie zostały wydane** (zbiór UTXO), brak nadwyżki wyjść, poprawny format.
- **Double spending**: dwie transakcje wydające ten sam UTXO – do łańcucha może trafić tylko jedna.

### Model kont (Ethereum)
Globalny **stan** = konta z saldami, licznikiem transakcji (_nonce_ – ochrona przed powtórzeniem), kodem i pamięcią (kontrakty); transakcja zmienia stan (przelew lub wywołanie kontraktu). Stan uwierzytelniony drzewem **Merkle Patricia Trie**.

## Sieć P2P
- **Pełne węzły** – przechowują i **niezależnie weryfikują** cały łańcuch i zbiór UTXO/stan; **górnicy/walidatorzy** – dodatkowo tworzą bloki; **lekkie klienty** (SPV) – tylko nagłówki.
- **Rozgłaszanie plotkowaniem** (_gossip_ – [[Algorytmy Rozproszone/Gossiping]]): nowa transakcja → walidacja → przekazanie sąsiadom → **mempool** (pula oczekujących); nowy blok → walidacja → przekazanie dalej.
- Odkrywanie węzłów: DNS seeds, wymiana adresów; Ethereum – Kademlia DHT ([[49 Ustrukturyzowane systemy P2P i DHT]]).
- Każdy węzeł stosuje te same **reguły konsensusu** i sam decyduje, który łańcuch jest poprawny.

## Konsensus Nakamoto – Proof of Work (PoW)
### Kopanie (_mining_)
Górnik:
1. wybiera transakcje z mempoolu (zwykle najwyższe opłaty), dodaje **coinbase** (nagroda),
2. buduje nagłówek i szuka **nonce** takiego, że:
$$SHA\text{-}256(SHA\text{-}256(\text{nagłówek})) < \text{cel (target)}$$
   czyli skrót zaczyna się od odpowiednio wielu zer. Jedyna metoda to **zgadywanie** (miliardy prób na sekundę na układach ASIC) – **łamigłówka kryptograficzna**, trudna do rozwiązania i trywialna do sprawdzenia,
3. po znalezieniu rozgłasza blok; inni sprawdzają skrót i wszystkie transakcje, a następnie **budują na nim** kolejny blok.

- **Dostosowanie trudności**: co **2016 bloków** cel korygowany tak, by średni czas bloku wynosił **~10 minut**, niezależnie od mocy obliczeniowej sieci.
- **Zachęty**: **nagroda za blok** (_block subsidy_ – nowe monety; 50 BTC na początku, **halving** co 210 000 bloków ≈ 4 lata; łączna podaż ograniczona do 21 mln BTC) + **opłaty transakcyjne**. Opłaca się grać uczciwie – wkład energii zwraca się tylko w łańcuchu akceptowanym przez sieć.

### Reguła najdłuższego łańcucha
- Węzły uznają za obowiązujący łańcuch o **największej łącznej pracy** (w uproszczeniu – najdłuższy).
- **Rozwidlenia tymczasowe** (_forks_): dwóch górników znajduje bloki prawie jednocześnie → węzły budują na pierwszym otrzymanym; gdy jedna gałąź wydłuży się, druga zostaje porzucona (**bloki osierocone** / _stale_), a jej transakcje wracają do mempoolu.
- **Potwierdzenia**: im więcej bloków nad blokiem z transakcją, tym mniejsze prawdopodobieństwo jej wycofania – maleje **wykładniczo** z liczbą potwierdzeń (analiza Nakamoto – spacer losowy). Zwyczajowo **6 potwierdzeń** (~1 h) → **finalność probabilistyczna**.
- **Sybil resistance**: „głos” = moc obliczeniowa (a nie liczba tożsamości) → tanie tworzenie węzłów nic nie daje.

### Bezpieczeństwo PoW
- **Atak 51%**: podmiot z większością mocy obliczeniowej może budować alternatywny łańcuch szybciej niż reszta → **wycofanie** własnych transakcji (double spend), cenzura transakcji; **nie może** ukraść cudzych środków (podpisy) ani stworzyć monet ponad reguły.
- **Selfish mining** (Eyal, Sirer 2014) – ukrywanie znalezionych bloków daje przewagę już przy ~25–33% mocy.
- Koszt ataku = koszt sprzętu i energii → bezpieczeństwo ekonomiczne.
- Tolerancja: działa w **otwartej** sieci z nieznaną liczbą uczestników (w odróżnieniu od klasycznych algorytmów bizantyjskich, które wymagają znanego zbioru $N$ – [[23 Rozproszone uzgadnianie w środowisku zawodnym]]).

## Proof of Stake (PoS) i inne mechanizmy konsensusu
### Proof of Stake
- Prawo do tworzenia bloku proporcjonalne do **zdeponowanych środków** (_stake_), a nie mocy obliczeniowej.
- **Ethereum** (od **The Merge**, wrzesień 2022): walidator deponuje **32 ETH**; czas podzielony na **sloty** (12 s) i **epoki** (32 sloty); losowo wybierany proponent bloku, komitety **attestują** (głosują); reguła wyboru gałęzi LMD-GHOST + **Casper FFG** – bloki **ostatecznie sfinalizowane** po 2 epokach (~13 min) przy głosach ≥ 2/3 stake’u.
- **Slashing** – kara (utrata części depozytu) za nieuczciwe zachowanie (podwójne głosowanie, sprzeczne propozycje) → atak kosztuje utratę własnego kapitału.
- ✔ ~99,9% mniej energii niż PoW, szybsza finalność; ✘ „bogaci się bogacą”, problem _nothing at stake_ (rozwiązany slashingiem), ataki dalekiego zasięgu (_long-range_ – punkty kontrolne), koncentracja w pulach stakingowych.
### Inne
| Mechanizm | Idea | Przykłady |
|---|---|---|
| **DPoS** (_Delegated PoS_) | posiadacze tokenów wybierają ograniczoną liczbę delegatów produkujących bloki | EOS, Tron |
| **PBFT i warianty BFT** | głosowanie znanego zbioru walidatorów, $N \geq 3f+1$, **natychmiastowa finalność** | Tendermint/CometBFT (Cosmos), HotStuff (Diem) |
| **Raft / Kafka** (crash fault tolerance) | uporządkowanie transakcji w sieci zaufanych organizacji | Hyperledger Fabric (ordering service) |
| **Proof of Authority** | zaufane, zidentyfikowane podmioty-walidatory | sieci testowe, konsorcja |
| **Proof of Space / Time** | dowód posiadania miejsca na dysku | Chia, Filecoin |

## Rodzaje blockchainów
| Rodzaj | Kto może uczestniczyć / walidować | Konsensus | Przykłady |
|---|---|---|---|
| **publiczny, bez uprawnień** (_permissionless_) | każdy czyta, wysyła transakcje i może walidować | PoW, PoS | Bitcoin, Ethereum |
| **prywatny, z uprawnieniami** (_permissioned_) | jedna organizacja kontroluje uczestników | BFT/Raft | wewnętrzne rejestry |
| **konsorcjalny** | grupa organizacji (np. banki, firmy logistyczne) | BFT | Hyperledger Fabric, R3 Corda, Quorum |

**Hyperledger Fabric** – model **execute-order-validate**:
1. transakcja **symulowana** przez węzły zatwierdzające (_endorsers_) wg polityki,
2. **porządkowanie** przez usługę ordering (Raft) w bloki,
3. **walidacja** i zatwierdzenie w każdym węźle (sprawdzenie polityki i konfliktów wersji);
kanały prywatne, tożsamości z certyfikatów (MSP), inteligentne kontrakty (_chaincode_) w Go/Java/JS.

## Inteligentne kontrakty (_smart contracts_)
- **Programy** przechowywane w łańcuchu i wykonywane **deterministycznie** przez wszystkie węzły przy transakcjach – „kod jest prawem”.
- **Ethereum** (V. Buterin, 2015): maszyna wirtualna **EVM** (stos, Turing-zupełna), języki **Solidity**, Vyper; **gaz** – opłata za każdą operację (ogranicza obliczenia, chroni przed nieskończonymi pętlami i DoS).
- **Zastosowania**: tokeny (ERC-20), **NFT** (ERC-721), **DeFi** (giełdy zdecentralizowane – AMM, pożyczki, stablecoiny), DAO (organizacje zarządzane kodem), wyrocznie (_oracles_ – dostarczanie danych z zewnątrz, np. Chainlink).
- **Ryzyka**: błędy w kodzie są nieodwracalne (**The DAO** 2016 – reentrancy, kradzież ~60 mln USD → twardy fork Ethereum), ataki na mosty między łańcuchami, manipulacja wyroczniami.

## Przeznaczenie i zastosowania
- **kryptowaluty i płatności** bez pośredników, przelewy transgraniczne,
- **zdecentralizowane finanse** (DeFi), tokenizacja aktywów, stablecoiny, CBDC (cyfrowe waluty banków centralnych – często nie blockchain publiczny),
- **łańcuchy dostaw** – śledzenie pochodzenia (żywność, leki – IBM Food Trust, TradeLens – zamknięte),
- **notaryzacja i znaczniki czasu** – dowód istnienia dokumentu w danym czasie (zapis skrótu),
- **tożsamość cyfrowa** (DID, self-sovereign identity), certyfikaty i dyplomy,
- **rejestry** (grunty, własność), głosowanie (projekty pilotażowe), gry i przedmioty cyfrowe,
- **rozproszone składowanie** (Filecoin, Arweave).

**Kiedy blockchain ma sens**: wiele **wzajemnie nieufających** stron musi współdzielić i modyfikować wspólny rejestr, a brak jest (lub nie jest pożądana) zaufana strona trzecia; potrzebna jest audytowalność i niezmienność historii. **Gdy istnieje zaufany operator – zwykła replikowana baza danych jest prostsza, szybsza i tańsza.**

## Własności, ograniczenia i rozwiązania
### Własności
**decentralizacja**, **niezmienność** (append-only, wykrywalność zmian), **przejrzystość i audytowalność**, **odporność na cenzurę i awarie** (tysiące replik), **pseudonimowość**, **programowalność** (kontrakty).

### Ograniczenia
- **Skalowalność**: Bitcoin ~**7 transakcji/s** (blok ~1–4 MB co 10 min), Ethereum ~15–30 tx/s (L1) vs Visa ~tysiące tx/s; wszystkie węzły weryfikują wszystko,
- **opóźnienie i finalność probabilistyczna** (PoW),
- **zużycie energii PoW** (Bitcoin – porównywalne z zużyciem średniego kraju),
- **trylemat blockchain** (Buterin): trudno jednocześnie osiągnąć **decentralizację**, **bezpieczeństwo** i **skalowalność**,
- **prywatność** – wszystkie transakcje jawne,
- **nieodwracalność** – błędy i kradzieże nie do cofnięcia, utrata kluczy,
- rosnący rozmiar łańcucha (Bitcoin > 500 GB), **regulacje prawne**, zmienność cen.

### Rozwiązania skalowalności i prywatności
- **warstwa 2** (_Layer 2_):
  - kanały płatności – **Lightning Network** (transakcje poza łańcuchem, rozliczenie w łańcuchu),
  - **rollupy** (optimistic – z okresem na dowody oszustwa, **ZK-rollupy** – z dowodami poprawności) – wiele transakcji jako jedna,
- **sharding** łańcucha, większe bloki (spory – rozwidlenie Bitcoin Cash), **SegWit**,
- **dowody z wiedzą zerową** (zk-SNARK – Zcash), mieszanie (CoinJoin), podpisy pierścieniowe (Monero).

## Blockchain w perspektywie systemów rozproszonych
| Aspekt | Klasyczny system rozproszony | Blockchain publiczny |
|---|---|---|
| Uczestnicy | znani, ograniczona liczba | nieznani, otwarta sieć, zmienni |
| Model awarii | crash / bizantyjskie przy znanym $N$ | bizantyjskie + racjonalni (zachęty ekonomiczne) |
| Konsensus | Paxos/Raft/PBFT – deterministyczny | Nakamoto PoW/PoS – probabilistyczny/ekonomiczny |
| Ochrona przed Sybil | uwierzytelnianie | koszt zasobu (praca, depozyt) |
| Spójność (CAP) | wybór C lub A | PoW preferuje **dostępność** (każdy dopisuje bloki), spójność **ostateczna** – rozwidlenia rozwiązywane z czasem |
| Replikacja | kilka replik | tysiące pełnych replik |

## Zobacz też
- [[47 Architektury systemów rozproszonych dużej skali]]
- [[24 Niezawodne zatwierdzanie transakcji rozproszonych]] – transakcje z koordynacją vs rejestr z konsensusem
