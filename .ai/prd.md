# Dokument wymagań produktu (PRD) - ESMpress

## 1. Przegląd produktu

ESMpress to aplikacja internetowa typu MVP (Minimum Viable Product) służąca do przeprowadzania badań metodą Experience Sampling Method (ESM). Głównym celem systemu jest automatyzacja procesu zbierania danych o doświadczeniach uczestników w czasie rzeczywistym.

System umożliwia badaczom tworzenie kampanii badawczych, w ramach których do uczestników wysyłane są wiadomości e-mail z linkami do ankiet w losowych momentach dnia (w określonym oknie czasowym). Kluczową cechą rozwiązania jest mechanizm zarządzania ważnością linków – nowy link unieważnia poprzedni, co wymusza na uczestnikach reagowanie na bieżąco, zwiększając wiarygodność danych (ekologiczna trafność badania).

Interfejs administratora oparty jest o panel Django Admin, a interfejs uczestnika zaprojektowano w podejściu mobile-first, aby umożliwić szybkie wypełnianie ankiet na smartfonach.

## 2. Problem użytkownika

### Perspektywa Badacza (Administratora)
Badacze chcący stosować metodę ESM napotykają trudności w ręcznym koordynowaniu wysyłki ankiet o losowych porach do wielu uczestników jednocześnie. Badacz potrzebuje prostego narzędzia do konfiguracji harmonogramu, treści maili i pobrania surowych danych.

### Perspektywa Uczestnika (Badanego)
Uczestnicy badania potrzebują sposobu na szybkie i bezproblemowe raportowanie swojego stanu. Konieczność logowania się do portalu, pamiętania haseł lub wypełniania skomplikowanych formularzy na małym ekranie telefonu zniechęca do udziału. Uczestnik oczekuje prostego linku w e-mailu, który przenosi go bezpośrednio do krótkiej ankiety dostosowanej do urządzeń mobilnych.

## 3. Wymagania funkcjonalne

### 3.1. Panel Administratora (Django Admin)
* System musi poziom dostępu (superuser) on widzi wszytsko i periodic tasks, User ktory widzi swoje kampanie ankiety itd. tworzony poprzez seedowanie bazy danych (brak rejestracji w UI).
* Możliwość tworzenia i edycji Kampanii z polami: Nazwa, Data startu, Data końca, Okno godzinowe (od-do), Liczba ankiet dziennie, Minimalny bufor czasowy (w minutach) między ankietami, Temat maila, Treść maila.
* Implementacja walidatora matematycznego: system musi blokować zapisanie kampanii, jeśli iloczyn liczby ankiet i bufora oraz czasu potrzebnego na wysyłkę przekracza dostępne okno czasowe w ciągu dnia.
* Zarządzanie pytaniami w ramach kampanii. Obsługiwane typy:
    * Pole tekstowe (otwarte).
    * Jednokrotny wybór (radio buttons).
    * Wielokrotny wybór (checkboxes).
    * Suwak (slider) – z konfiguracją wartości min/max oraz etykietami tekstowymi dla skrajnych wartości.
* Zarządzanie listą uczestników: możliwość ręcznego dodawania lub importu listy adresów e-mail (np. z CSV lub wklejenie listy).
* Dashboard wyświetlający status kampanii oraz proste liczniki: Łączna liczba wysłanych powiadomień, Łączna liczba zebranych odpowiedzi.
* Przycisk do ręcznego zatrzymania aktywnej kampanii.

### 3.2. Logika Biznesowa i Backend
* Scheduler (Harmonogram): Algorytm losujący godziny wysyłki dla każdego uczestnika indywidualnie w zadanym oknie czasowym, z zachowaniem zdefiniowanego bufora czasowego.
* Strefa czasowa systemu ustawiona na sztywno na Polskę (Europe/Warsaw).
* Integracja z SendGrid API do wysyłki maili (klucze API w zmiennych środowiskowych).
* System tokenów: Każdy wysłany link zawiera unikalny token.
* Logika inwalidacji: W momencie wygenerowania i wysłania nowego linku do uczestnika, poprzedni niewypełniony link tego uczestnika musi zostać natychmiast dezaktywowany. Linki wygasają również po zakończeniu kampanii oraz przesłaniu odpowiedzi.
* Zapis danych: Odpowiedzi są zapisywane w bazie danych wraz ze znacznikiem czasu wysłania ankiety oraz znacznikiem czasu jej wypełnienia.

### 3.3. Eksport Danych
* Możliwość pobrania wyników dla wybranej kampanii w formacie CSV.
* Format "Wide" (Szeroki): Każdy wiersz to pojedyncze wypełnienie ankiety.
* Kolumny: ID uczestnika (email), Data i godzina wysłania, Data i godzina wypełnienia, Pytanie 1 (nagłówek), Pytanie 2 (nagłówek)...
* Pytania stanowią nagłówki kolumn.

### 3.4. Frontend Ankiety (Interfejs Uczestnika)
* Brak konieczności logowania – dostęp przez unikalny token w URL.
* Responsywność (Mobile-first): duże przyciski, czytelne czcionki, obsługa dotykowa suwaków.
* Walidacja formularza: wszystkie pytania są wymagalne.
* Strona błędu informująca o wygaśnięciu linku (jeśli token jest nieaktywny).
* Statyczna strona z podziękowaniem po poprawnym przesłaniu formularza.

## 4. Granice produktu

### Wchodzi w zakres (In-Scope)
* Tworzenie i edycja ankiet oraz kampanii przez panel Django Admin.
* Algorytm losowania czasu wysyłki z uwzględnieniem buforów.
* Wysyłka e-maili transakcyjnych przez SendGrid.
* Mechanizm unieważniania starych linków ("tylko jedna aktywna ankieta na raz").
* Eksport danych do CSV.
* Frontend do wypełniania ankiet zoptymalizowany pod urządzenia mobilne.

### Nie wchodzi w zakres (Out-of-Scope)
* Panel logowania dla uczestników (uczestnicy nie mają kont, są tylko odbiorcami maili).
* Zaawansowane statystyki i wykresy wewnątrz aplikacji (analiza odbywa się w zewnętrznych narzędziach po eksporcie CSV).
* Integracja z AI lub skomplikowane systemy analizy tekstu.
* Możliwość edycji pytań w trakcie trwania aktywnej kampanii (blokada edycji po starcie).
* Przypomnienia o niewypełnieniu ankiety (tylko jeden mail per okno losowania).

## 5. Historyjki użytkowników

### Uwierzytelnianie i Bezpieczeństwo

* ID: US-001
    * Tytuł: Logowanie administratora
    * Opis: Jako Administrator chcę zalogować się do panelu administracyjnego przy użyciu bezpiecznych poświadczeń, aby zarządzać systemem.
    * Kryteria akceptacji:
        1. Dostęp do panelu admina `/admin` wymaga podania loginu i hasła.
        2. Konto administratora jest tworzone przy wdrażaniu aplikacji (seed/createsuperuser).
        3. Nieudana próba logowania wyświetla komunikat błędu.

### Zarządzanie Kampaniami

* ID: US-002
    * Tytuł: Konfiguracja parametrów kampanii
    * Opis: Jako Administrator chcę zdefiniować czas trwania badania, dzienne okno godzinowe, liczbę ankiet oraz treść maila, aby dostosować badanie do moich potrzeb.
    * Kryteria akceptacji:
        1. Administrator może ustawić datę początkową i końcową.
        2. Administrator może ustawić ramy godzinowe (np. 8:00 - 20:00).
        3. Administrator może wpisać temat i treść wiadomości email.
        4. Administrator może zdefiniować liczbę ankiet dziennie.

* ID: US-003
    * Tytuł: Walidacja harmonogramu (Anti-Crash)
    * Opis: Jako Administrator chcę otrzymać ostrzeżenie, jeśli moje ustawienia (liczba ankiet + bufor) nie mieszczą się w oknie czasowym, aby uniknąć błędów w losowaniu.
    * Kryteria akceptacji:
        1. System przelicza: (liczba ankiet * bufor min) w stosunku do długości okna godzinowego.
        2. Próba zapisu niemożliwej konfiguracji (np. 10 ankiet z buforem 60 min w oknie 8h) kończy się błędem walidacji.
        3. Kampania nie zostaje zapisana/aktywowana do momentu poprawy parametrów.

* ID: US-004
    * Tytuł: Tworzenie pytań do ankiety
    * Opis: Jako Administrator chcę dodać różne typy pytań do kampanii, w tym suwaki i pola wyboru, aby zebrać zróżnicowane dane.
    * Kryteria akceptacji:
        1. Możliwość dodania pytania typu: Tekst, Jednokrotny wybór, Wielokrotny wybór.
        2. Możliwość dodania pytania typu Suwak (Slider) z definicją min, max, label min, label max.
        3. Możliwość ustalenia kolejności pytań.

* ID: US-005
    * Tytuł: Import bazy uczestników
    * Opis: Jako Administrator chcę wgrać listę adresów email uczestników, aby nie musieć wprowadzać ich ręcznie pojedynczo.
    * Kryteria akceptacji:
        1. Możliwość wklejenia listy maili lub prosty import.
        2. System waliduje format adresów email.
        3. Duplikaty maili w ramach jednej kampanii są ignorowane lub scalane.

### Przebieg Badania i Logika

* ID: US-006
    * Tytuł: Automatyczna wysyłka zaproszeń (Scheduler)
    * Opis: Jako Administrator chcę, aby system automatycznie wysyłał maile w wylosowanych godzinach, abym nie musiał tego robić ręcznie.
    * Kryteria akceptacji:
        1. System losuje godziny wysyłki dla każdego użytkownika osobno w zadanym oknie.
        2. Pomiędzy wysyłkami zachowany jest zdefiniowany minimalny bufor czasowy.
        3. Maile są wysyłane zgodnie z harmonogramem przez SendGrid.

* ID: US-007
    * Tytuł: Mechanizm rotacji tokenów (Unieważnianie linków)
    * Opis: Jako Badacz chcę, aby wysłanie nowej ankiety dezaktywowało poprzedni link tego samego uczestnika, aby uniknąć wypełniania zaległych ankiet ("backfilling").
    * Kryteria akceptacji:
        1. Użytkownik otrzymuje Link A. Po nadejściu czasu na kolejną ankietę, system generuje Link B i wysyła go.
        2. Kliknięcie w Link A po wygenerowaniu Linku B skutkuje wyświetleniem komunikatu "Ankieta nieaktywna".
        3. Kliknięcie w Link B otwiera formularz.

### Interfejs Uczestnika

* ID: US-008
    * Tytuł: Wypełnianie ankiety na telefonie
    * Opis: Jako Uczestnik chcę wygodnie wypełnić ankietę na smartfonie, używając interfejsu dotykowego, aby proces był szybki.
    * Kryteria akceptacji:
        1. Interfejs skaluje się do ekranu mobilnego.
        2. Suwaki są łatwe w obsłudze dotykiem.
        3. Przycisk "Wyślij" jest wyraźny i łatwo dostępny.
        4. Formularz nie pozwala na wysłanie pustych odpowiedzi (walidacja front-end).

* ID: US-009
    * Tytuł: Potwierdzenie wysłania
    * Opis: Jako Uczestnik po wysłaniu ankiety chcę zobaczyć ekran z podziękowaniem, aby mieć pewność, że dane dotarły.
    * Kryteria akceptacji:
        1. Po kliknięciu "Wyślij" następuje przekierowanie na stronę "Dziękujemy".
        2. Ponowne wejście w ten sam link po wypełnieniu wyświetla komunikat, że ankieta została już wypełniona.

### Dane i Raportowanie

* ID: US-010
    * Tytuł: Eksport danych do CSV
    * Opis: Jako Badacz chcę pobrać plik CSV ze wszystkimi odpowiedziami, sformatowany tak, aby każde wypełnienie było osobnym wierszem, co ułatwi mi analizę w Excelu/SPSS/R.
    * Kryteria akceptacji:
        1. Przycisk "Eksportuj CSV" dostępny w panelu kampanii.
        2. Plik zawiera kolumny: Email, Czas Wysłania, Czas Wypełnienia, Pytanie 1, Pytanie 2...
        3. Znaki specjalne w odpowiedziach tekstowych są poprawnie kodowane (UTF-8).

* ID: US-011
    * Tytuł: Podgląd postępów (Dashboard)
    * Opis: Jako Administrator chcę widzieć na dashboardzie proste liczniki wysłanych i wypełnionych ankiet, aby monitorować czy system działa.
    * Kryteria akceptacji:
        1. Wyświetlanie liczby wysłanych maili w danej kampanii.
        2. Wyświetlanie liczby otrzymanych odpowiedzi w danej kampanii.

## 6. Metryki sukcesu

Powodzenie wdrożenia ESMpress (MVP) będzie mierzone następującymi wskaźnikami:

1.  Niezawodność Harmonogramu (Scheduler Reliability)
    * Cel: 100% zaplanowanych ankiet zostaje przetworzonych przez system wysyłkowy w wyznaczonym oknie czasowym (brak pominiętych "slotów").

2.  Integralność Danych (Data Integrity)
    * Cel: 0 zgłoszeń dotyczących błędnego mapowania odpowiedzi do pytań w pliku CSV. Liczba rekordów w CSV musi być równa liczbie rekordów o statusie "Completed" w bazie danych.

3.  Skuteczność Mechanizmu Wygasania (Logic Accuracy)
    * Cel: 0 przypadków, w których użytkownik mógł wypełnić starą ankietę po otrzymaniu nowej. (Weryfikowane testami manualnymi przed wdrożeniem oraz analizą logów).

4.  Dostępność UX (User Accessibility)
    * Cel: Brak zgłoszeń krytycznych od uczestników (blockerów) uniemożliwiających wypełnienie ankiety na standardowych przeglądarkach mobilnych (Chrome/Safari na iOS/Android).