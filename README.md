# CodexBot L2 — etap 1: obserwator mózgu
Osobny projekt dla własnej postaci na autoryzowanym serwerze L2Solo C4.
Wersja 0.1.0 odczytuje świat i proponuje decyzje. **Nie wykonuje ataków,
ruchu, skilli, użycia itemów ani zmian w bazie gry.** Panel nie jest jeszcze pełnym botem.

## Uruchom demonstrację
Wymagany Node.js 22.5 lub nowszy. Brak dodatkowych zależności npm.

    npm start

Otwórz http://127.0.0.1:18766.
Panel jest wyraźnie oznaczony DEMO. Co 10 sekund zmieniają się scenariusze:
jeden przeciwnik, trzech przeciwników i utrata HP, śmierć, odrodzenie.
Przyciski zatrzymują/wznawiają obserwację, nie sterują postacią.

## Podłącz prawdziwe L2Solo
Najpierw zatrzymaj serwer w dotychczasowym launcherze. Nie uruchamiaj dwóch
instancji na tej samej bazie. W osobnym terminalu, w katalogu tego projektu:

    npm run start:l2solo -- --root "C:\Users\user\Desktop\L2Solo-main" --character 123

Zamień 123 na numeryczne ID własnej postaci widoczne w szczegółach postaci
WorldObserver (końcowy numer adresu strony /observer/actors/player/ID).
Zaloguj tę postać w kliencie C4 i otwórz http://127.0.0.1:18766.
Połączenie ma wtedy etykietę L2SOLO, bez danych demonstracyjnych.
Gdy postać jest wylogowana, panel pokazuje brak połączenia z postacią.

Skrypt uruchamia istniejący scripts/run-server.js z preloadem obserwatora.
Nie kopiuje plików do serwera i nie zmienia jego źródeł ani konfiguracji.
L2Solo normalnie zarządza własną bazą podczas pracy.
Po Ctrl+C serwer dostaje standardową prośbę o zamknięcie. Następnym razem
można uruchomić zwykły launcher, bez obserwatora.
Alternatywny port panelu: zmienna CODEXBOT_PORT.

## Co działa
- Odczyt postaci ~100 ms; znanych klientowi NPC w promieniu 2000 i party ~500 ms.
- Skille/buffy ~1 s, inventory ~5 s; znaczniki czasu każdego czujnika.
- Ponowne wykorzystanie rzeczywistego BotTargetScorer z L2Solo, bez kopiowania jego kodu.
- Priorytety śmierć → przetrwanie → obrona → regeneracja → cel → poszukiwanie.
- Ranking z uzasadnieniami, ograniczona pamięć i historia 50 decyzji.
- Czyszczenie pamięci po utracie sesji/zmianie postaci; odrzucanie starych migawek.
- Odczyt efektów nie uruchamia EffectStore.list(), który przy okazji usuwa wygasłe efekty.
- Kontroler pojedynczej akcji z blokadą, anulowaniem i timeoutem, sprawdzony na adapterach testowych.
  Adaptery wersji 0.1.0 zgłaszają pustą listę obsługiwanych akcji i odrzucają wykonanie.

## Ograniczenia etapu
Nie ma jeszcze autonomicznej walki, poruszania, zakupów, leczenia, lootu,
stref farmienia ani trwałego uczenia. SURVIVE to wskazanie nadrzędnego celu,
a nie potwierdzenie dostępności konkretnej mikstury/skilla.
NPC są oceniane punktowo; nie potwierdzamy jeszcze przechodniej trasy.
Party to projekcja lidera i jego towarzyszy, nie pełna obsługa wszystkich wariantów party.
Gracze/PvP nie są celami tego etapu. Puste pole skilli/buffów oznacza pusty odczyt.
Tempo utraty HP jest przybliżeniem z kolejnych próbek, nie dziennikiem trafień.
Leczenie między próbkami może zaniżyć wynik. Interwały są orientacyjne — zależą
od obciążenia głównej pętli serwera.
Testy offline nie zastępują potwierdzenia integracji na zalogowanej postaci.

## Weryfikacja i paczka
    npm test
    npm run build

Build sprawdza składnię i przygotowuje wykonywalną paczkę CommonJS w dist/.
Nie jest to instalator EXE. npm start w dist uruchamia demonstrację.
Podczas rozwoju testuj źródła, a nie folder dist.

## Architektura
src/adapters — granica serwera; migawki JSON i jawne możliwości adaptera.
src/core — scheduler, pamięć, aplikacja, kontroler akcji.
src/brain — niezależna ocena propozycji.
src/ui — lokalny panel i API obserwacji.
integrations/l2solo — preload uruchamiany tylko w procesie NodeL2.
tests — scenariusze, adapter i HTTP.

## Następny etap
Dodać jawnie włączany adapter akcji dla jednej sesji, przejęcie ręczne,
potwierdzenia rezultatów, ruch z geodatą i testy w grze. Następnie połączyć
kontroler z walką i SurvivalBrain. Nie podszywać gracza pod BotSession.

## Źródła audytu
Audytowano lokalną kopię L2Solo 0.0.7, 17.10.2026, a nie rozpakowane RAR.
978 plików JS źródeł przeszło parsowanie. Testy test_bot_target_scorer
i test_tcp_packet_framing zakończyły się powodzeniem. To nie jest pełna
certyfikacja L2Solo ani gwarancja zgodności z każdą wersją.
Repozytorium nie zawiera bazy, zapisów, geodaty, kluczy ani kodu L2Solo.
