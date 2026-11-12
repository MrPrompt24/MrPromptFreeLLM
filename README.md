# MrPrompt FreeLLM – aplikacja na Windows

Czat z wieloma darmowymi API LLM. Dodajesz klucze dostawców, a gdy jednemu skończy się limit, aplikacja sama przechodzi do następnego.

## Budowanie i aktualizacje

```powershell
.\build.ps1                  # podnosi wersję (0.1.N -> 0.1.N+1), uruchamia testy, buduje instalator
.\build.ps1 -Version 0.2.0   # ustawia wersję ręcznie
.\build.ps1 -NoBump          # ta sama wersja
```

Wynik: `dist\MrPromptFreeLLM-Setup-<wersja>.exe`.

**Aktualizacja u użytkowników = wysłanie im nowego instalatora.** Instalator ma stały `AppId`, więc nadpisuje zainstalowaną wersję w tym samym folderze (bez uprawnień administratora), samodzielnie zamyka działającą aplikację i nie rusza danych użytkownika. Starszej wersji na nowszą nie zainstaluje bez pytania.

> Nie zmieniaj `AppId` w `installer\setup.iss`. Po zmianie instalator uznałby aplikację za inną i zainstalował ją obok.

## Gdzie są dane użytkownika

`%APPDATA%\MrPromptFreeLLM`: `settings.json`, `keys.dat` (klucze szyfrowane DPAPI, tylko dla tego konta Windows), `conversations\`, `state.json` (przerwy dostawców).
Zmienna `MRPROMPT_DATA` pozwala uruchomić aplikację na innym folderze danych (do testów).

## Pomoc i „O aplikacji”

Na górze okna są dwa przyciski: **Pomoc** (opis działania i wszystkich funkcji prostym językiem, 20 tematów z wyszukiwarką, a klawisz **F1** otwiera temat do aktualnej zakładki) oraz **O aplikacji** (nazwa, wersja, autor MrPrompt, odnośniki, folder danych i przycisk kopiujący informacje o wersji bez kluczy). Treść pomocy leży w `src\MrPromptChat\HelpContent.cs`.

## Obraz w pytaniu (dla każdego)

W czacie i w arenie przycisk 📎 dołącza obraz do pytania (albo wklej zrzut ekranu przez Ctrl+V). Działa z modelami, które rozumieją obrazy (np. z rodziny Gemini). Duże zdjęcia są zmniejszane (dłuższy bok do 1568 px), a obraz idzie tylko z ostatnią wiadomością. Obrazy nie są zapisywane w plikach rozmów.

## Tryb zaawansowany (platforma do eksperymentów)

Domyślnie aplikacja jest prostym czatem. Przycisk „Włącz tryb zaawansowany” jest w oknie głównym, w panelu bocznym (można go też przełączyć w ustawieniach, zakładka Ogólne). Po włączeniu pojawiają się:

- **Prompty**: biblioteka promptów z folderami, zmiennymi `{{nazwa}}` i historią wersji (stare wersje zostają, można pokazać zmiany między wersjami). Prompt można jednym kliknięciem użyć w czacie albo w arenie (aplikacja zapyta o wartości zmiennych).
- **Studio**: trzy funkcje poza czatem. **Obrazy**: generowanie obrazu u kilku dostawców naraz, z porównaniem i zapisem do pliku. **Dźwięk na tekst**: transkrypcja nagrania (do 25 MB) u jednego lub kilku dostawców. **Podobieństwo tekstów**: embeddingi, mapa podobieństwa i ranking tekstów względem zapytania. Przycisk „Znajdź model” pobiera listę modeli dostawcy i proponuje właściwy. Nie każdy dostawca obsługuje każdą funkcję, a brak obsługi jest zgłaszany czytelnym komunikatem i nie blokuje jego zwykłego czatu.
- **Wiedza (RAG)**: baza własnych dokumentów (tekst, Markdown, CSV, HTML, Word .docx i podobne, bez PDF). Aplikacja dzieli pliki na fragmenty, liczy embeddingi u wybranego dostawcy (po zgodzie, z szacunkiem zapytań) i zapisuje indeks lokalnie, a przy pytaniu znajduje pasujące fragmenty i prosi model o odpowiedź wyłącznie na ich podstawie, ze wskazaniem źródeł.
- **Agent**: zadanie wykonuje model, który sam wybiera narzędzia: kalkulator, data i godzina, moje dokumenty (z bazy wiedzy), pliki z jednego folderu i pobieranie stron https (te dwa ostatnie włączasz świadomie). Każdy krok widać, jest limit kroków. Wymaga modelu z obsługą narzędzi (function calling).
- **Serwer**: lokalny serwer zgodny z API OpenAI (`/v1/models`, `/v1/chat/completions`, także strumieniowo). Edytory kodu, skrypty i n8n łączą się z nim jak z OpenAI (adres `http://127.0.0.1:port/v1`, klucz API = token z zakładki), a aplikacja przełącza dostawców, gdy któremuś skończy się limit. Nasłuch tylko na tym komputerze, wymagany token, model „auto” albo „dostawca/model”. Narzędzia (tools) klienta nie są obsługiwane.
- **Testy**: zestawy przypadków testowych z oceną odpowiedzi regułami (zawiera tekst, dokładnie, regex, poprawny JSON, długość) albo przez model-sędziego. Uruchamiasz je na wybranych modelach i wariantach promptu, a wynik to tabela model × przypadek z punktacją (eksport CSV i JSON). Test pokazuje przed startem, ile zapytań wyśle, wysyła do jednego dostawcy po jednym zapytaniu naraz i respektuje przerwy po limitach.

- **Arena**: jedno pytanie do 2–4 modeli naraz, odpowiedzi obok siebie z pomiarem czasu i szybkości, głosowanie „która lepsza”.
- **Parametry** (⚙): temperatura, top P, maks. długość, seed, własne polecenie, presety i podgląd żądania. Działają w czacie i w arenie.
- **Dziennik**: ranking modeli (skuteczność, czasy, wygrane z areny) i lista ostatnich wywołań, eksport do CSV. Zapisuje tylko liczby i wynik, nigdy treści rozmów (`%APPDATA%\MrPromptFreeLLM\calls.jsonl`, `votes.jsonl`).

## Struktura

- `src\MrPromptChat.Core` – logika bez interfejsu: klient API, router z przełączaniem, katalog, magazyn kluczy.
- `src\MrPromptChat` – interfejs WPF (.NET 9).
- `tests\MrPromptChat.Tests` – testy routera na atrapie serwera (`dotnet run --project tests\MrPromptChat.Tests`).
- `installer\setup.iss` – skrypt Inno Setup.

Ten folder leży w katalogu WWW, ale `.htaccess` blokuje go w Apache. Nie wgrywaj go na serwer publiczny.
