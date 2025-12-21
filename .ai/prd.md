# Dokument wymagań produktu (PRD) - MentorConnect

## 1. Przegląd produktu

MentorConnect to platforma łącząca użytkowników z ekspertami i mentorami z różnych dziedzin, umożliwiająca szybki dostęp do pomocy, weryfikację pomysłów i mentoring w rozwiązywaniu problemów. Platforma pozwala użytkownikom sprawdzić swoje koncepcje, zweryfikować wiedzę z różnych źródeł i otrzymać wsparcie od doświadczonych praktyków.

### Cel produktu

Stworzenie bezpiecznej, łatwej w użyciu platformy, która umożliwia:
- Szybkie znajdowanie odpowiednich ekspertów z różnych dziedzin
- Rezerwowanie sesji mentoringowych i konsultacyjnych
- Komunikację między użytkownikami a ekspertami
- Weryfikację pomysłów przed wdrożeniem
- Sprawdzenie wiedzy zdobytej z różnych źródeł (AI, internet, książki, kursy)
- Mentoring w rozwiązywaniu konkretnych problemów

### Wizja

Platforma ma być uniwersalnym miejscem, gdzie każdy może znaleźć eksperta w dowolnej dziedzinie - od porady prawnej, przez ocenę dokumentu, mentoring muzyczny, po zabezpieczenie aplikacji webowej. W przyszłości platforma będzie wspierać płatne sesje, ale w MVP wszystkie sesje są darmowe.

### Zasady projektowe
- **Uniwersalność i Prostota:** Interfejs użytkownika (UI) powinien być intuicyjny, prosty w obsłudze i dostępny dla szerokiego grona odbiorców, niezależnie od ich wieku czy biegłości technicznej.
- **Zachęta do Rozwoju:** Projekt wizualny i sposób komunikacji platformy mają aktywnie motywować użytkowników do nauki, weryfikowania pomysłów i korzystania z wiedzy ekspertów.

### Stack technologiczny

- Frontend: Astro 5, React 19, TypeScript, Tailwind CSS 4
- Backend: Astro SSR z Node.js adapter
- Baza danych: Supabase (PostgreSQL)
- Autentykacja: Supabase Auth
- Deployment: Vercel (platforma hostingowa)

## 2. Problem użytkownika

### Główny problem

Użytkownicy mają dostęp do ogromnej ilości informacji z różnych źródeł (AI, internet, książki, kursy), ale potrzebują szybkiego dostępu do ekspertów, którzy mogą:
- Zweryfikować ich pomysły przed wdrożeniem
- Pomóc w rozwiązaniu konkretnych problemów poprzez mentoring
- Potwierdzić lub skorygować wiedzę zdobytą z różnych źródeł
- Udzielić szybkiej porady w kluczowych momentach

Platforma ma umożliwić szybkie połączenie z odpowiednim ekspertem, aby użytkownicy mogli sprawdzić swoje pomysły, zweryfikować wiedzę i otrzymać wsparcie w rozwiązywaniu problemów.

### Konkretne problemy użytkowników

1. Trudność w znalezieniu odpowiedniego eksperta w danej dziedzinie
2. Brak prostego i szybkiego sposobu na umówienie sesji z ekspertem
3. Potrzeba weryfikacji pomysłów przed podjęciem ważnych decyzji lub rozpoczęciem projektu
4. Potrzeba szybkiej porady lub wsparcia w konkretnym problemie
5. Chęć sprawdzenia wiedzy zdobytej z różnych źródeł (AI, internet, książki, kursy) u doświadczonego praktyka
6. Potrzeba mentoringu w rozwiązywaniu problemów, a nie tylko dostępu do informacji
7. Brak pewności co do jakości, aktualności i odpowiedniości wiedzy dla konkretnej sytuacji
8. Chęć uczenia się od doświadczonych praktyków, którzy mogą podzielić się realnym doświadczeniem

### Konkretne problemy ekspertów

1. Trudność w dotarciu do osób potrzebujących ich wiedzy
2. Brak centralnego miejsca do oferowania swoich usług
3. Potrzeba prostego systemu zarządzania rezerwacjami
4. Chęć dzielenia się wiedzą i budowania reputacji

## 3. Wymagania funkcjonalne

### 3.1. Autentykacja i zarządzanie kontem

- Rejestracja użytkownika (email + hasło)
- Logowanie użytkownika
- Wylogowanie użytkownika
- Odzyskiwanie hasła
- Jeden profil użytkownika z możliwością bycia zarówno użytkownikiem jak i ekspertem
- Podstawowy profil użytkownika (imię, email, avatar opcjonalnie)

### 3.2. Profile ekspertów

- Tworzenie profilu eksperta (użytkownik może zostać ekspertem)
- Edycja profilu eksperta
- Wyświetlanie profilu eksperta z podstawowymi informacjami:
  - Imię i nazwisko
  - Kategorie/usługi (przypisanie do kategorii z drzewa kategorii)
  - Krótki opis doświadczenia
  - Doświadczenie/portfolio (tekst)
  - Kontakt (email/telefon)
- Możliwość przypisania eksperta do wielu kategorii

### 3.3. System kategorii

- Płaskie kategorie (bez hierarchii) - uproszczenie dla MVP
- Predefiniowane kategorie:
  - Prawo i Finanse
  - Zdrowie i Medycyna
  - Technologia
  - Edukacja i Rozwój
  - Biznes
  - Inne
- Filtrowanie ekspertów po kategoriach (jedna kategoria na raz w MVP)
- Przypisywanie ekspertów do kategorii (możliwość wyboru wielu kategorii)
- Hierarchiczne drzewo kategorii będzie dodane w późniejszych iteracjach

### 3.4. Wyszukiwanie i przeglądanie ekspertów

- Lista wszystkich ekspertów
- Wyszukiwarka ekspertów (po imieniu, kategorii, opisie)
- Filtrowanie ekspertów po kategoriach
- Wyświetlanie szczegółów eksperta
- Sortowanie ekspertów (domyślnie: alfabetycznie)

### 3.5. Rezerwacja sesji

- Wybór eksperta
- Wybór terminu sesji (data i godzina)
- Podanie podstawowych informacji o potrzebie (tekst)
- Potwierdzenie rezerwacji
- Wyświetlanie moich rezerwacji (dla użytkownika)
- Wyświetlanie rezerwacji do mnie (dla eksperta)
- Wyświetlanie danych kontaktowych po rezerwacji:
  - Dla użytkownika: email i telefon eksperta
  - Dla eksperta: email i telefon użytkownika
- Anulowanie rezerwacji

### 3.6. Dashboard

- Dla użytkownika:
  - Moje rezerwacje (nadchodzące, przeszłe)
  - Przełącznik widoku (szukam eksperta / jestem ekspertem)
- Dla eksperta:
  - Moje rezerwacje (nadchodzące, przeszłe)
  - Mój profil eksperta
  - Statystyki (liczba rezerwacji, opcjonalnie w MVP)

## 4. Granice produktu

### Co NIE będzie w MVP

1. System płatności
   - Wszystkie sesje w MVP są darmowe
   - Płatności będą dodane w późniejszych iteracjach

2. System ocen i recenzji
   - Rating ekspertów nie będzie dostępny w MVP
   - Recenzje będą dodane w późniejszych iteracjach

3. Zintegrowane video call
   - W MVP sesje odbywają się przez chat + kontakt (email/telefon)
   - Video call w aplikacji będzie dodany w późniejszych iteracjach

4. Zaawansowane rekomendacje
   - Brak algorytmu rekomendacji ekspertów w MVP
   - Proste filtrowanie i wyszukiwanie

5. System komunikacji (Chat)
   - Chat w aplikacji nie będzie dostępny w MVP
   - Komunikacja odbywa się przez email/telefon (dane kontaktowe dostępne po rezerwacji)
   - Chat będzie dodany w późniejszych iteracjach

6. Zaawansowane powiadomienia
   - Rozbudowane powiadomienia w aplikacji i e-mail (np. przypomnienia o sesjach, newslettery) są poza zakresem MVP.
   - Podstawowe e-maile transakcyjne (potwierdzenie rezerwacji, informacja o anulowaniu) są w zakresie MVP, aby umożliwić kluczową komunikację.

7. Panel administracyjny
   - Zarządzanie kategoriami przez seed data w bazie
   - Panel admina będzie dodany w późniejszych iteracjach

8. Zaawansowane statystyki
   - Podstawowe statystyki dla ekspertów (opcjonalnie)
   - Szczegółowe analytics będą dodane później

9. Hierarchiczne drzewo kategorii
   - W MVP kategorie są płaskie (bez podkategorii)
   - Hierarchiczne drzewo kategorii będzie dodane w późniejszych iteracjach

10. Mobilna aplikacja
   - Tylko aplikacja webowa (responsive design)
   - Aplikacja mobilna będzie rozważana w przyszłości

11. Deployment i infrastruktura
   - Platforma hostingowa: Vercel
   - CI/CD: GitHub Actions (będzie zaimplementowany później w ramach MVP)
   - Automatyczny pipeline CI/CD (planowany):
     - Lint i format check
     - Build aplikacji
     - Testy e2e (Playwright)
     - TypeScript type check
     - Deployment na Vercel (tylko jeśli wszystkie testy przejdą)
   - Deployment początkowo ręczny/CLI, później automatyczny przez CI/CD
   - Zmienne środowiskowe konfigurowane w GitHub Secrets i Vercel dashboard
   - SSL/HTTPS automatycznie zapewnione przez Vercel
   - Domena własna (opcjonalnie w MVP, można dodać później)
   - Monitoring i logi dostępne w Vercel dashboard
   - Deployment będzie zaimplementowany w iteracji 6 (finalizacja MVP)

12. Testy automatyczne
   - Framework: Playwright (testy e2e)
   - Kluczowe testy e2e dla głównych scenariuszy:
     - Rejestracja i logowanie użytkownika
     - Tworzenie profilu eksperta
     - Wyszukiwanie i filtrowanie ekspertów
     - Rezerwacja sesji (najważniejszy flow)
   - Testy będą zaimplementowane w iteracji 6 (finalizacja MVP)
   - Testy manualne jako uzupełnienie testów automatycznych

## 5. Historyjki użytkowników

### Priorytetyzacja historyjek (MoSCoW)

Poniższa lista określa priorytety dla poszczególnych historyjek w ramach MVP.

**MUST-HAVE (Krytyczne dla MVP):**
*   `US-001`: Rejestracja
*   `US-002`: Logowanie
*   `US-003`: Wylogowanie
*   `US-005`: Tworzenie profilu eksperta
*   `US-006`: Edycja profilu eksperta
*   `US-010`: Wyświetlanie szczegółów profilu eksperta
*   `US-011`: Rezerwacja sesji
*   `US-012`: Wyświetlanie moich rezerwacji (użytkownik)
*   `US-013`: Wyświetlanie rezerwacji do mnie (ekspert)
*   `US-016`: Walidacja dostępności terminu
*   `US-017`: Wyświetlanie dostępnych slotów
*   `US-018`: Definiowanie dostępności przez eksperta

**SHOULD-HAVE (Ważne, ale nie krytyczne):**
*   `US-004`: Odzyskiwanie hasła
*   `US-007`: Przeglądanie listy ekspertów
*   `US-008`: Wyszukiwanie ekspertów
*   `US-009`: Filtrowanie po kategoriach
*   `US-014`: Anulowanie rezerwacji

**COULD-HAVE ("Miło mieć"):**
*   `US-015`: Przełączanie między widokiem użytkownika a eksperta

### US-001: Rejestracja nowego użytkownika

Jako nowy użytkownik chcę zarejestrować się w systemie, aby móc korzystać z platformy.

Opis: Użytkownik wchodzi na stronę główną i widzi opcję rejestracji. Wypełnia formularz z emailem i hasłem, następnie otrzymuje potwierdzenie rejestracji i jest automatycznie zalogowany.

Kryteria akceptacji:
- Formularz rejestracji zawiera pola: email, hasło, potwierdzenie hasła
- Walidacja emaila (format email)
- Walidacja hasła (minimum 8 znaków)
- Sprawdzenie czy email nie jest już zarejestrowany
- Po udanej rejestracji użytkownik jest automatycznie zalogowany
- Użytkownik jest przekierowany do dashboardu

### US-002: Logowanie użytkownika

Jako zarejestrowany użytkownik chcę zalogować się do systemu, aby uzyskać dostęp do mojego konta.

Opis: Użytkownik wchodzi na stronę logowania, wprowadza email i hasło, następnie jest logowany do systemu.

Kryteria akceptacji:
- Formularz logowania zawiera pola: email, hasło
- Walidacja czy użytkownik istnieje
- Walidacja hasła
- Po udanym logowaniu użytkownik jest przekierowany do dashboardu
- W przypadku błędu wyświetlany jest komunikat o błędzie

### US-003: Wylogowanie użytkownika

Jako zalogowany użytkownik chcę wylogować się z systemu, aby zabezpieczyć moje konto.

Opis: Użytkownik klika przycisk wylogowania w menu, następnie jest wylogowany i przekierowany na stronę główną.

Kryteria akceptacji:
- Przycisk wylogowania jest dostępny w menu dla zalogowanych użytkowników
- Po kliknięciu użytkownik jest wylogowany
- Sesja jest zakończona
- Użytkownik jest przekierowany na stronę główną

### US-004: Odzyskiwanie hasła

Jako użytkownik chcę odzyskać dostęp do mojego konta, jeśli zapomniałem hasła.

Opis: Użytkownik klika "Zapomniałem hasła" na stronie logowania, wprowadza email, następnie otrzymuje email z linkiem do resetowania hasła.

Kryteria akceptacji:
- Opcja "Zapomniałem hasła" jest dostępna na stronie logowania
- Formularz resetowania wymaga podania emaila
- Email z linkiem resetującym jest wysyłany (jeśli email istnieje w systemie)
- Link prowadzi do formularza zmiany hasła
- Nowe hasło jest zapisywane po potwierdzeniu

### US-005: Tworzenie profilu eksperta

Jako użytkownik chcę utworzyć profil eksperta, aby oferować swoje usługi mentoringowe.

Opis: Użytkownik wchodzi w tryb eksperta (przełącznik w dashboardzie), następnie wypełnia formularz tworzenia profilu eksperta z podstawowymi informacjami.

Kryteria akceptacji:
- Przełącznik "Jestem ekspertem" jest dostępny w dashboardzie
- Formularz tworzenia profilu zawiera pola:
  - Krótki opis doświadczenia (textarea)
  - Doświadczenie/portfolio (textarea)
  - Kontakt email/telefon
  - Wybór kategorii (możliwość wyboru wielu)
- Walidacja wymaganych pól
- Po zapisaniu profil jest aktywny i ekspert pojawia się w liście ekspertów
- Ekspert może edytować swój profil później

### US-006: Edycja profilu eksperta

Jako ekspert chcę edytować mój profil, aby zaktualizować informacje o moich usługach.

Opis: Ekspert wchodzi do sekcji "Mój profil eksperta" i może edytować wszystkie pola profilu.

Kryteria akceptacji:
- Przycisk "Edytuj profil" jest dostępny w dashboardzie eksperta
- Wszystkie pola profilu są edytowalne
- Zmiany są zapisywane po kliknięciu "Zapisz"
- Zaktualizowany profil jest widoczny dla innych użytkowników

### US-007: Przeglądanie listy ekspertów

Jako użytkownik chcę przeglądać listę dostępnych ekspertów, aby znaleźć odpowiedniego dla moich potrzeb.

Opis: Użytkownik wchodzi na stronę z listą ekspertów i widzi wszystkich dostępnych ekspertów z podstawowymi informacjami.

Kryteria akceptacji:
- Lista ekspertów wyświetla wszystkich aktywnych ekspertów
- Dla każdego eksperta wyświetlane są: imię, kategorie, krótki opis
- Lista jest paginowana (jeśli jest więcej niż 20 ekspertów)
- Kliknięcie w eksperta prowadzi do szczegółów profilu

### US-008: Wyszukiwanie ekspertów

Jako użytkownik chcę wyszukiwać ekspertów po nazwie, kategorii lub opisie, aby szybko znaleźć odpowiedniego eksperta.

Opis: Użytkownik wprowadza frazę wyszukiwania w pole wyszukiwarki, system wyświetla pasujących ekspertów.

Kryteria akceptacji:
- Pole wyszukiwarki jest dostępne na stronie z listą ekspertów
- Wyszukiwanie działa w czasie rzeczywistym (lub po kliknięciu przycisku)
- Wyszukiwanie obejmuje: imię eksperta, kategorie, opis
- Wyniki są wyświetlane natychmiast
- Jeśli brak wyników, wyświetlany jest komunikat "Brak wyników"

### US-009: Filtrowanie ekspertów po kategoriach

Jako użytkownik chcę filtrować ekspertów po kategoriach, aby znaleźć ekspertów w konkretnej dziedzinie.

Opis: Użytkownik wybiera kategorię z listy, a lista ekspertów jest filtrowana, aby pokazać tylko tych z wybranej kategorii.

Kryteria akceptacji:
- Lista kategorii jest dostępna na stronie z listą ekspertów
- Wybór kategorii filtruje listę ekspertów
- Możliwość wyczyszczenia filtrów

### US-010: Wyświetlanie szczegółów profilu eksperta

Jako użytkownik chcę zobaczyć szczegóły profilu eksperta, aby zdecydować czy chcę zarezerwować sesję.

Opis: Użytkownik klika na eksperta z listy, następnie widzi pełny profil eksperta ze wszystkimi informacjami.

Kryteria akceptacji:
- Strona szczegółów eksperta wyświetla:
  - Imię i nazwisko
  - Wszystkie kategorie
  - Pełny opis doświadczenia
  - Portfolio/doświadczenie
  - Kontakt (email/telefon)
- Przycisk "Zarezerwuj sesję" jest dostępny
- Możliwość powrotu do listy ekspertów

### US-011: Rezerwacja sesji z ekspertem

Jako użytkownik chcę zarezerwować sesję z ekspertem, aby uzyskać poradę lub mentoring.

Opis: Użytkownik wybiera eksperta, klika "Zarezerwuj sesję", wybiera datę i godzinę, podaje podstawowe informacje o potrzebie, następnie potwierdza rezerwację. Po potwierdzeniu otrzymuje dane kontaktowe eksperta (email i telefon).

Kryteria akceptacji:
- Formularz rezerwacji zawiera:
  - Wybór daty (kalendarz)
  - Wybór godziny (dostępne sloty czasowe)
  - Pole tekstowe z opisem potrzeby/pytania
- Walidacja czy termin nie jest w przeszłości
- Walidacja czy termin jest dostępny (nie koliduje z innymi rezerwacjami)
- Po potwierdzeniu rezerwacja jest zapisana
- System wysyła transakcyjny e-mail z potwierdzeniem rezerwacji i danymi kontaktowymi do użytkownika.
- System wysyła transakcyjny e-mail z informacją o rezerwacji i danymi kontaktowymi do eksperta.
- Rezerwacja pojawia się w dashboardzie obu stron

### US-012: Wyświetlanie moich rezerwacji (użytkownik)

Jako użytkownik chcę zobaczyć moje rezerwacje, aby wiedzieć kiedy mam zaplanowane sesje i mieć dostęp do danych kontaktowych eksperta.

Opis: Użytkownik wchodzi do dashboardu i widzi sekcję "Moje rezerwacje" z listą nadchodzących i przeszłych rezerwacji. Dla każdej rezerwacji widzi dane kontaktowe eksperta.

Kryteria akceptacji:
- Lista rezerwacji wyświetla:
  - Eksperta (imię, kategorie)
  - Datę i godzinę
  - Status (nadchodząca, przeszła, anulowana)
  - Opis potrzeby
  - Dane kontaktowe eksperta (email i telefon) - widoczne po potwierdzeniu rezerwacji
- Rezerwacje są sortowane chronologicznie (najbliższe na górze)
- Możliwość filtrowania (nadchodzące/przeszłe)
- Możliwość anulowania nadchodzących rezerwacji

### US-013: Wyświetlanie rezerwacji do mnie (ekspert)

Jako ekspert chcę zobaczyć rezerwacje od użytkowników, aby wiedzieć kiedy mam zaplanowane sesje i mieć dostęp do danych kontaktowych użytkownika.

Opis: Ekspert wchodzi do dashboardu i widzi sekcję "Rezerwacje do mnie" z listą rezerwacji od użytkowników. Dla każdej rezerwacji widzi dane kontaktowe użytkownika.

Kryteria akceptacji:
- Lista rezerwacji wyświetla:
  - Użytkownika (imię)
  - Datę i godzinę
  - Status (nadchodząca, przeszła, anulowana)
  - Opis potrzeby użytkownika
  - Dane kontaktowe użytkownika (email i telefon) - widoczne po potwierdzeniu rezerwacji
- Rezerwacje są sortowane chronologicznie
- Możliwość filtrowania (nadchodzące/przeszłe)
- Możliwość anulowania nadchodzących rezerwacji

### US-014: Anulowanie rezerwacji

Jako użytkownik/ekspert chcę anulować rezerwację, jeśli nie mogę uczestniczyć w sesji.

Opis: Użytkownik lub ekspert klika "Anuluj" przy rezerwacji, potwierdza anulowanie, rezerwacja jest anulowana.

Kryteria akceptacji:
- Przycisk "Anuluj" jest dostępny dla nadchodzących rezerwacji
- Po kliknięciu wyświetlany jest dialog potwierdzenia
- Po potwierdzeniu rezerwacja jest oznaczona jako anulowana
- Druga strona (użytkownik/ekspert) otrzymuje informację o anulowaniu
- Anulowana rezerwacja nie jest już widoczna w aktywnych rezerwacjach

### US-015: Przełączanie między widokiem użytkownika a eksperta

Jako użytkownik, który jest również ekspertem, chcę przełączać się między widokiem szukania ekspertów a widokiem zarządzania moim profilem eksperta.

Opis: Użytkownik ma przełącznik w dashboardzie, który pozwala zmienić widok między "Szukam eksperta" a "Jestem ekspertem".

Kryteria akceptacji:
- Przełącznik jest widoczny w dashboardzie dla użytkowników, którzy są ekspertami
- Przełączenie zmienia widok dashboardu
- Widok "Szukam eksperta" pokazuje listę ekspertów i moje rezerwacje jako użytkownik
- Widok "Jestem ekspertem" pokazuje mój profil eksperta i rezerwacje do mnie

### US-016: Walidacja dostępności terminu przy rezerwacji

Jako system chcę sprawdzać dostępność terminu przed zapisaniem rezerwacji, aby uniknąć konfliktów czasowych.

Opis: Gdy użytkownik próbuje zarezerwować sesję, system sprawdza, czy wybrany termin jest zgodny ze zdefiniowanym przez eksperta szablonem dostępności i czy nie jest już zajęty przez inną rezerwację.

Kryteria akceptacji:
- System sprawdza dostępność terminu przed zapisaniem rezerwacji (czy jest w szablonie eksperta i czy nie jest już zajęty).
- Jeśli termin jest zajęty lub niedostępny, wyświetlany jest komunikat błędu.
- Użytkownik może wybrać inny termin.
- System nie sugeruje alternatywnych terminów w MVP (użytkownik wybiera inny termin ręcznie).

### US-017: Wyświetlanie dostępnych slotów czasowych

Jako użytkownik chcę widzieć dostępne sloty czasowe eksperta, aby wybrać odpowiedni termin.

Opis: Gdy użytkownik wybiera datę w kalendarzu rezerwacji, system pokazuje dostępne godziny dla tego dnia, bazując na tygodniowym szablonie dostępności zdefiniowanym przez eksperta oraz na już istniejących rezerwacjach.

Kryteria akceptacji:
- Po wyborze daty, system sprawdza, jaki to dzień tygodnia (np. wtorek).
- System pobiera zdefiniowane przez eksperta dostępne sloty dla tego dnia tygodnia.
- System sprawdza, które z tych slotów na wybraną, konkretną datę są już zajęte przez inne rezerwacje.
- Użytkownikowi wyświetlana jest finalna lista dostępnych slotów (dostępne w szablonie MINUS już zarezerwowane).
- Zajęte sloty są niewidoczne lub nieaktywne.
- Użytkownik może wybrać tylko dostępny slot.

### US-018: Definiowanie tygodniowego szablonu dostępności przez eksperta

Jako ekspert, chcę zdefiniować mój standardowy tygodniowy harmonogram dostępności, aby użytkownicy mogli rezerwować sesje tylko w godzinach, w których jestem dostępny.

Opis: W moim profilu eksperta mam dostęp do sekcji "Zarządzanie dostępnością". Widzę tam szablon tygodnia (poniedziałek-niedziela) z podziałem na godzinowe sloty (np. od 8:00 do 20:00). Mogę zaznaczyć (np. checkboxami), w których godzinach jestem typowo dostępny w każdym dniu tygodnia. System użyje tego szablonu do pokazywania mojej dostępności w kalendarzu rezerwacji.

Kryteria akceptacji:
- W profilu eksperta istnieje sekcja do zarządzania dostępnością.
- Interfejs wyświetla 7 dni tygodnia.
- Dla każdego dnia dostępne są do zaznaczenia godzinowe sloty (np. 9:00, 10:00, 11:00...).
- Ekspert może zaznaczyć i odznaczyć dowolny slot.
- Zmiany są zapisywane po kliknięciu przycisku "Zapisz".
- System przechowuje szablon dostępności dla każdego eksperta.
- Ten szablon jest używany jako podstawa do wyświetlania dostępnych terminów w formularzu rezerwacji.

## 6. Metryki sukcesu

Poniższe metryki służą do oceny powodzenia MVP. Dzielą się na dwie główne kategorie:

*   **Metryki Jakości Produktu (np. techniczne, funkcjonalne):** Mierzą, czy produkt został zbudowany poprawnie, zgodnie z wymaganiami i standardami jakości. Są one w dużej mierze zależne od pracy zespołu deweloperskiego.
*   **Metryki Przyjęcia Rynkowego (np. użytkowników, ekspertów, biznesowe):** Mierzą, czy produkt zdobywa zainteresowanie i jest używany. Ich osiągnięcie zależy nie tylko od jakości produktu, ale również od działań marketingowych, strategii wprowadzenia na rynek i ogólnego zapotrzebowania.

Taki podział pomaga oddzielić ocenę jakości wykonania produktu od oceny jego sukcesu rynkowego.

### 6.1. Metryki użytkowników

- Liczba zarejestrowanych użytkowników
- Liczba aktywnych użytkowników (logowanie w ciągu ostatnich 7 dni)
- Liczba użytkowników, którzy zarezerwowali przynajmniej jedną sesję
- Współczynnik konwersji: zarejestrowani użytkownicy -> rezerwacja sesji

### 6.2. Metryki ekspertów

- Liczba aktywnych ekspertów (eksperci z utworzonym profilem)
- Liczba ekspertów z przynajmniej jedną rezerwacją
- Średnia liczba rezerwacji na eksperta
- Liczba ekspertów w każdej kategorii

### 6.3. Metryki sesji

- Liczba zarezerwowanych sesji
- Liczba zrealizowanych sesji (sesje z przeszłości)
- Liczba anulowanych sesji
- Współczynnik realizacji: zarezerwowane -> zrealizowane sesje

### 6.4. Metryki techniczne

- Czas ładowania strony (cel: < 2 sekundy)
- Dostępność systemu (cel: > 99%)
- Liczba błędów w aplikacji

### 6.5. Metryki biznesowe (dla przyszłych iteracji)

- Liczba płatnych sesji (po wprowadzeniu płatności)
- Przychód z platformy (po wprowadzeniu prowizji)
- Współczynnik retencji użytkowników

### Definicja sukcesu MVP

MVP będzie uznane za sukces, jeśli w ciągu 4 tygodni po uruchomieniu zostaną spełnione następujące kryteria:

#### Kryteria ilościowe (użytkownicy i aktywność):
- Minimum 10 ekspertów utworzy profile w różnych kategoriach
- Minimum 3 ekspertów w każdej z 5 głównych kategorii (dla zapewnienia różnorodności)
- Minimum 20 użytkowników zarejestruje się
- Minimum 10 sesji zostanie zarezerwowanych
- Minimum 5 sesji zostanie zrealizowanych (sesje z przeszłymi datami)
- Współczynnik konwersji: minimum 30% zarejestrowanych użytkowników zarezerwuje przynajmniej jedną sesję

#### Kryteria jakościowe (funkcjonalność):
- Platforma będzie działać stabilnie bez krytycznych błędów (zero krytycznych błędów w produkcji)
- Wszystkie główne funkcje działają zgodnie z PRD:
  - Rejestracja i logowanie
  - Tworzenie i edycja profili ekspertów
  - Wyszukiwanie i filtrowanie ekspertów
  - Rezerwacja sesji z wyświetlaniem danych kontaktowych
  - Wyświetlanie rezerwacji w dashboardach
  - Anulowanie rezerwacji
- Użytkownicy będą mogli znaleźć ekspertów, zarezerwować sesje i skontaktować się przez email/telefon (dane kontaktowe dostępne po rezerwacji)

#### Kryteria techniczne:
- Czas ładowania strony < 2 sekundy (dla 90% requestów)
- Dostępność systemu > 99% (w czasie działania MVP)
- Zero krytycznych błędów w produkcji
- Minimum 2 kluczowe testy e2e przechodzą (rejestracja/logowanie, rezerwacja sesji) - pozostałe testy można dodać po MVP
- Wszystkie testy manualne głównych scenariuszy zakończone sukcesem
- Aplikacja jest wdrożona i dostępna publicznie na Vercel
- Wszystkie zmienne środowiskowe są poprawnie skonfigurowane
- SSL/HTTPS działa poprawnie

