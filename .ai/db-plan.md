# Schemat bazy danych MentorConnect

## 1. Tabele

### 1.1. profiles

**Tabela zintegrowana z Supabase Auth** - automatycznie synchronizowana z `auth.users` (tworzenie/usuwanie przez triggers).

Tabela przechowująca podstawowe dane użytkownika. Mapowana 1:1 z `auth.users` poprzez `id`.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY`, `REFERENCES auth.users(id) ON DELETE CASCADE` | ID użytkownika (tożsame z auth.users.id) |
| `full_name` | `text` | `NULL` | Pełne imię i nazwisko (wymagane do uzupełnienia) |
| `avatar_path` | `text` | `NULL DEFAULT 'avatars/default-avatar.png'` | Ścieżka do avatara w storage (domyślny avatar) |
| `avatar_updated_at` | `timestamptz` | `NULL` | Data ostatniej aktualizacji avatara |
| `is_profile_complete` | `boolean` | `NOT NULL DEFAULT false` | Czy profil jest kompletny (full_name wypełnione) |
| `is_active` | `boolean` | `NOT NULL DEFAULT true` | Czy konto jest aktywne (soft delete) |
| `deactivated_at` | `timestamptz` | `NULL` | Data dezaktywacji konta |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia profilu |
| `updated_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data ostatniej aktualizacji |

**Uwagi:**
- Profil tworzony automatycznie przez trigger po rejestracji w Supabase Auth
- Dwuetapowa rejestracja: signup → uzupełnienie `full_name` (wymagane) + opcjonalnie avatar
- `is_profile_complete` automatycznie = true gdy `full_name` wypełnione (trigger)
- Soft delete zachowuje historię rezerwacji

### 1.2. expert_profiles

Tabela przechowująca dane specyficzne dla ekspertów. Relacja 0..1 do `profiles`.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY DEFAULT gen_random_uuid()` | ID profilu eksperta |
| `user_id` | `uuid` | `NOT NULL UNIQUE`, `REFERENCES profiles(id) ON DELETE CASCADE` | ID użytkownika (właściciel profilu) |
| `bio` | `text` | `NOT NULL` | Krótki opis doświadczenia |
| `experience` | `text` | `NOT NULL` | Doświadczenie/portfolio (dłuższy tekst) |
| `timezone` | `text` | `NOT NULL DEFAULT 'Europe/Warsaw'` | Strefa czasowa eksperta |
| `is_published` | `boolean` | `NOT NULL DEFAULT true` | Czy profil jest opublikowany i widoczny |
| `is_active` | `boolean` | `NOT NULL DEFAULT true` | Czy profil jest aktywny (soft delete) |
| `deactivated_at` | `timestamptz` | `NULL` | Data dezaktywacji profilu eksperta |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia profilu |
| `updated_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data ostatniej aktualizacji |

**Uwagi:**
- `user_id` ma constraint `UNIQUE` - jeden użytkownik może mieć tylko jeden profil eksperta
- Dezaktywacja eksperta (`is_active = false`) powoduje anulowanie wszystkich przyszłych rezerwacji
- `timezone` wspiera poprawne generowanie slotów czasowych (DST)
- `is_published` pozwala ekspertowi ukryć profil bez dezaktywacji

### 1.3. categories

Tabela słownikowa z predefiniowanymi kategoriami (seed data). Płaska struktura w MVP.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY DEFAULT gen_random_uuid()` | ID kategorii |
| `slug` | `text` | `NOT NULL UNIQUE` | Unikalny, stały identyfikator (np. 'law-and-finance') |
| `name` | `text` | `NOT NULL` | Nazwa kategorii wyświetlana użytkownikowi |
| `description` | `text` | `NULL` | Opcjonalny opis kategorii |
| `display_order` | `integer` | `NOT NULL DEFAULT 0` | Kolejność wyświetlania |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia |

**Uwagi:**
- `slug` jest stały i nie zmienia się (używany w URL)
- `name` może być edytowane bez wpływu na integracje
- Kategorie są tworzone jako seed data w migracji
- MVP obejmuje 6 predefiniowanych kategorii: Prawo i Finanse, Zdrowie i Medycyna, Technologia, Edukacja i Rozwój, Biznes, Inne

### 1.4. expert_categories

Tabela łącząca (junction table) dla relacji M:N między ekspertami a kategoriami.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY DEFAULT gen_random_uuid()` | ID powiązania |
| `expert_id` | `uuid` | `NOT NULL`, `REFERENCES expert_profiles(id) ON DELETE CASCADE` | ID eksperta |
| `category_id` | `uuid` | `NOT NULL`, `REFERENCES categories(id) ON DELETE CASCADE` | ID kategorii |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data przypisania |

**Constraints:**
- `UNIQUE(expert_id, category_id)` - ekspert może być przypisany do kategorii tylko raz

**Uwagi:**
- Umożliwia ekspertowi bycie przypisanym do wielu kategorii
- Usunięcie eksperta lub kategorii automatycznie usuwa powiązania (CASCADE)

### 1.5. expert_availability_slots

Tabela przechowująca tygodniowy szablon dostępności eksperta.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY DEFAULT gen_random_uuid()` | ID slotu |
| `expert_id` | `uuid` | `NOT NULL`, `REFERENCES expert_profiles(id) ON DELETE CASCADE` | ID eksperta |
| `weekday` | `integer` | `NOT NULL`, `CHECK (weekday >= 0 AND weekday <= 6)` | Dzień tygodnia (0=Niedziela, 6=Sobota) |
| `start_time` | `time` | `NOT NULL` | Godzina rozpoczęcia |
| `end_time` | `time` | `NOT NULL`, `CHECK (end_time > start_time)` | Godzina zakończenia |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia slotu |

**Constraints:**
- `CHECK (end_time > start_time)` - slot musi mieć pozytywny czas trwania

**Uwagi:**
- Tygodniowy szablon dostępności (np. "każdy wtorek 9:00-17:00")
- Edycja przez nadpisanie wszystkich slotów w transakcji
- `weekday`: 0=Niedziela (konwencja ISO/PostgreSQL)

### 1.6. bookings

Tabela przechowująca rezerwacje sesji między użytkownikami a ekspertami.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `id` | `uuid` | `PRIMARY KEY DEFAULT gen_random_uuid()` | ID rezerwacji |
| `user_id` | `uuid` | `NOT NULL`, `REFERENCES profiles(id) ON DELETE CASCADE` | ID użytkownika rezerwującego |
| `expert_id` | `uuid` | `NOT NULL`, `REFERENCES expert_profiles(id) ON DELETE CASCADE` | ID eksperta |
| `start_at` | `timestamptz` | `NOT NULL` | Data i godzina rozpoczęcia sesji |
| `end_at` | `timestamptz` | `NOT NULL` | Data i godzina zakończenia sesji |
| `status` | `text` | `NOT NULL DEFAULT 'pending'`, `CHECK (status IN ('pending', 'confirmed', 'cancelled', 'completed'))` | Status rezerwacji |
| `user_need_description` | `text` | `NOT NULL` | Opis potrzeby/pytania użytkownika |
| `cancelled_at` | `timestamptz` | `NULL` | Data anulowania rezerwacji |
| `cancelled_by_user_id` | `uuid` | `NULL`, `REFERENCES profiles(id) ON DELETE SET NULL` | ID użytkownika, który anulował |
| `cancellation_reason` | `text` | `NULL` | Powód anulowania (wymagany tylko dla eksperta) |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia rezerwacji |
| `updated_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data ostatniej aktualizacji |

**Constraints:**
- `CHECK (end_at > start_at)` - rezerwacja musi mieć pozytywny czas trwania
- Exclusion constraint dla unikania kolizji rezerwacji (patrz sekcja "Dodatkowe constraints")

**Uwagi:**
- Statusy: `pending`, `confirmed`, `cancelled`, `completed`
- `cancellation_reason` wymagany tylko dla eksperta (walidacja w RPC)
- `cancelled_by_user_id` może być NULL (soft delete)

### 1.7. profile_contact

Tabela przechowująca dane kontaktowe użytkowników i ekspertów z ostrzejszym kontrolą dostępu.

| Kolumna | Typ | Ograniczenia | Opis |
|---------|-----|--------------|------|
| `user_id` | `uuid` | `PRIMARY KEY`, `REFERENCES profiles(id) ON DELETE CASCADE` | ID użytkownika |
| `email` | `text` | `NULL` | Adres email kontaktowy |
| `phone` | `text` | `NULL` | Numer telefonu |
| `created_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data utworzenia |
| `updated_at` | `timestamptz` | `NOT NULL DEFAULT now()` | Data ostatniej aktualizacji |

**Constraints:**
- `CHECK (email IS NOT NULL OR phone IS NOT NULL)` - przynajmniej jeden kanał kontaktu jest wymagany

**Uwagi:**
- Ujawniane tylko właścicielowi lub po istniejącej rezerwacji (RLS + view/RPC)
- Alternatywny kontakt (email z `auth.users` jako fallback przez view)
- Wymagany min. 1 kanał (email lub telefon)
- Dopuszczalna anonimizacja bez usuwania historii

---

## 2. Relacje między tabelami

### 2.1. profiles ↔ auth.users
- **Typ:** 1:1 (obowiązkowa)
- **Relacja:** `profiles.id` → `auth.users.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie użytkownika z auth usuwa profil

### 2.2. profiles ↔ expert_profiles
- **Typ:** 1:0..1 (opcjonalna)
- **Relacja:** `expert_profiles.user_id` → `profiles.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie profilu usuwa profil eksperta
- **Constraint:** `UNIQUE(user_id)` - jeden użytkownik = jeden profil eksperta

### 2.3. expert_profiles ↔ categories (przez expert_categories)
- **Typ:** M:N
- **Tabela łącząca:** `expert_categories`
- **Relacje:**
  - `expert_categories.expert_id` → `expert_profiles.id`
  - `expert_categories.category_id` → `categories.id`
- **Kaskada:** `ON DELETE CASCADE` na obu stronach

### 2.4. expert_profiles ↔ expert_availability_slots
- **Typ:** 1:N
- **Relacja:** `expert_availability_slots.expert_id` → `expert_profiles.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie eksperta usuwa jego sloty

### 2.5. profiles ↔ bookings (jako użytkownik)
- **Typ:** 1:N
- **Relacja:** `bookings.user_id` → `profiles.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie profilu usuwa rezerwacje

### 2.6. expert_profiles ↔ bookings (jako ekspert)
- **Typ:** 1:N
- **Relacja:** `bookings.expert_id` → `expert_profiles.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie eksperta usuwa rezerwacje

### 2.7. profiles ↔ bookings (anulowanie)
- **Typ:** 1:N
- **Relacja:** `bookings.cancelled_by_user_id` → `profiles.id`
- **Kaskada:** `ON DELETE SET NULL` - zachowanie historii anulowania

### 2.8. profiles ↔ profile_contact
- **Typ:** 1:1
- **Relacja:** `profile_contact.user_id` → `profiles.id`
- **Kaskada:** `ON DELETE CASCADE` - usunięcie profilu usuwa dane kontaktowe

---

## 3. Indeksy

Indeksy zoptymalizowane pod kątem najczęstszych zapytań w MVP.

### 3.1. profiles
```sql
CREATE INDEX idx_profiles_is_active ON profiles(is_active) WHERE is_active = true;
```
- **Uzasadnienie:** Filtrowanie aktywnych użytkowników

### 3.2. expert_profiles
```sql
CREATE INDEX idx_expert_profiles_user_id ON expert_profiles(user_id);
CREATE INDEX idx_expert_profiles_active_published ON expert_profiles(is_active, is_published) 
  WHERE is_active = true AND is_published = true;
```
- **Uzasadnienie:** 
  - Wyszukiwanie profilu eksperta po user_id
  - Listowanie aktywnych i opublikowanych ekspertów

### 3.3. expert_categories
```sql
CREATE INDEX idx_expert_categories_category_id ON expert_categories(category_id, expert_id);
CREATE INDEX idx_expert_categories_expert_id ON expert_categories(expert_id);
```
- **Uzasadnienie:** 
  - Filtrowanie ekspertów po kategoriach (główny use case)
  - Pobieranie kategorii dla eksperta

### 3.4. expert_availability_slots
```sql
CREATE INDEX idx_availability_expert_weekday ON expert_availability_slots(expert_id, weekday);
```
- **Uzasadnienie:** Pobieranie slotów dostępności dla danego dnia tygodnia

### 3.5. bookings
```sql
CREATE INDEX idx_bookings_user_start ON bookings(user_id, start_at DESC);
CREATE INDEX idx_bookings_expert_start ON bookings(expert_id, start_at DESC);
CREATE INDEX idx_bookings_status_start ON bookings(status, start_at) 
  WHERE status IN ('pending', 'confirmed');
```
- **Uzasadnienie:** 
  - Dashboard użytkownika (moje rezerwacje, sortowane chronologicznie)
  - Dashboard eksperta (rezerwacje do mnie, sortowane chronologicznie)
  - Filtrowanie aktywnych rezerwacji po statusie

### 3.6. profile_contact
```sql
-- Brak dodatkowych indeksów - PRIMARY KEY (user_id) jest wystarczający
```

---

## 4. Dodatkowe constraints

### 4.1. Exclusion constraint dla bookings (unikanie kolizji)

PostgreSQL wspiera exclusion constraints z użyciem rozszerzeń GiST i `tstzrange` dla zapewnienia, że żadne dwie rezerwacje tego samego eksperta nie nakładają się w czasie.

```sql
-- Wymagane rozszerzenie
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Exclusion constraint
ALTER TABLE bookings
ADD CONSTRAINT bookings_expert_time_overlap_excl 
EXCLUDE USING gist (
  expert_id WITH =,
  tstzrange(start_at, end_at, '[)') WITH &&
)
WHERE (status IN ('pending', 'confirmed'));
```

**Uzasadnienie:**
- Gwarantuje, że ekspert nie ma nakładających się rezerwacji
- Działa tylko dla aktywnych rezerwacji (`pending`, `confirmed`)
- Anulowane i zakończone rezerwacje nie blokują slotów

---

## 5. Zasady PostgreSQL (RLS Policies)

Row Level Security (RLS) będzie włączone na wszystkich tabelach. Poniżej przedstawiono główne zasady dostępu.

### 5.1. profiles

```sql
-- Włączenie RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Użytkownik może odczytać własny profil
CREATE POLICY profiles_select_own ON profiles
  FOR SELECT
  USING (auth.uid() = id);

-- Użytkownik może aktualizować własny profil
CREATE POLICY profiles_update_own ON profiles
  FOR UPDATE
  USING (auth.uid() = id)
  WITH CHECK (auth.uid() = id);

-- Każdy może odczytać podstawowe dane aktywnych użytkowników (dla listy ekspertów)
CREATE POLICY profiles_select_public ON profiles
  FOR SELECT
  USING (is_active = true);
```

**Uwagi:**
- Użytkownik może edytować tylko swój profil (własne `id`)
- Automatyczne tworzenie profilu odbywa się przez trigger (nie przez RLS INSERT policy)
- `WITH CHECK` zapewnia, że użytkownik nie może zmienić swojego `id` podczas aktualizacji

### 5.2. expert_profiles

```sql
-- Włączenie RLS
ALTER TABLE expert_profiles ENABLE ROW LEVEL SECURITY;

-- Każdy może odczytać aktywne i opublikowane profile ekspertów
CREATE POLICY expert_profiles_select_public ON expert_profiles
  FOR SELECT
  USING (is_active = true AND is_published = true);

-- Ekspert może odczytać własny profil (nawet jeśli nieopublikowany)
CREATE POLICY expert_profiles_select_own ON expert_profiles
  FOR SELECT
  USING (auth.uid() = user_id);

-- Ekspert może tworzyć własny profil
CREATE POLICY expert_profiles_insert_own ON expert_profiles
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Ekspert może aktualizować własny profil
CREATE POLICY expert_profiles_update_own ON expert_profiles
  FOR UPDATE
  USING (auth.uid() = user_id);

-- Ekspert może usuwać własny profil
CREATE POLICY expert_profiles_delete_own ON expert_profiles
  FOR DELETE
  USING (auth.uid() = user_id);
```

### 5.3. categories

```sql
-- Włączenie RLS
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;

-- Każdy może odczytać kategorie
CREATE POLICY categories_select_all ON categories
  FOR SELECT
  USING (true);

-- Tylko admin może modyfikować kategorie (w przyszłości)
-- W MVP kategorie są seed data i nie są edytowane przez UI
```

### 5.4. expert_categories

```sql
-- Włączenie RLS
ALTER TABLE expert_categories ENABLE ROW LEVEL SECURITY;

-- Każdy może odczytać powiązania ekspert-kategoria
CREATE POLICY expert_categories_select_all ON expert_categories
  FOR SELECT
  USING (true);

-- Ekspert może zarządzać własnymi kategoriami
CREATE POLICY expert_categories_manage_own ON expert_categories
  FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM expert_profiles
      WHERE id = expert_categories.expert_id
      AND user_id = auth.uid()
    )
  );
```

### 5.5. expert_availability_slots

```sql
-- Włączenie RLS
ALTER TABLE expert_availability_slots ENABLE ROW LEVEL SECURITY;

-- Każdy może odczytać dostępność (do wyświetlania w kalendarzu)
CREATE POLICY availability_select_all ON expert_availability_slots
  FOR SELECT
  USING (true);

-- Ekspert może zarządzać własnymi slotami
CREATE POLICY availability_manage_own ON expert_availability_slots
  FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM expert_profiles
      WHERE id = expert_availability_slots.expert_id
      AND user_id = auth.uid()
    )
  );
```

### 5.6. bookings

```sql
-- Włączenie RLS
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;

-- Użytkownik może odczytać własne rezerwacje
CREATE POLICY bookings_select_as_user ON bookings
  FOR SELECT
  USING (auth.uid() = user_id);

-- Ekspert może odczytać rezerwacje do siebie
CREATE POLICY bookings_select_as_expert ON bookings
  FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM expert_profiles
      WHERE id = bookings.expert_id
      AND user_id = auth.uid()
    )
  );

-- Użytkownik może tworzyć rezerwacje (jako user_id)
CREATE POLICY bookings_insert_as_user ON bookings
  FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Anulowanie przez RPC (cancel_booking) z walidacją roli i powodu
-- RLS pozwala na update tylko dla właściciela lub eksperta
CREATE POLICY bookings_update_own ON bookings
  FOR UPDATE
  USING (
    auth.uid() = user_id OR 
    EXISTS (
      SELECT 1 FROM expert_profiles
      WHERE id = bookings.expert_id
      AND user_id = auth.uid()
    )
  );
```

### 5.7. profile_contact

```sql
-- Włączenie RLS
ALTER TABLE profile_contact ENABLE ROW LEVEL SECURITY;

-- Użytkownik może odczytać własne dane kontaktowe
CREATE POLICY contact_select_own ON profile_contact
  FOR SELECT
  USING (auth.uid() = user_id);

-- Użytkownik może odczytać dane kontaktowe drugiej strony po istniejącej rezerwacji
CREATE POLICY contact_select_after_booking ON profile_contact
  FOR SELECT
  USING (
    -- Jako użytkownik: mogę zobaczyć kontakt eksperta
    EXISTS (
      SELECT 1 FROM bookings b
      JOIN expert_profiles ep ON b.expert_id = ep.id
      WHERE b.user_id = auth.uid()
      AND ep.user_id = profile_contact.user_id
      AND b.status IN ('pending', 'confirmed', 'completed')
    )
    OR
    -- Jako ekspert: mogę zobaczyć kontakt użytkownika
    EXISTS (
      SELECT 1 FROM bookings b
      JOIN expert_profiles ep ON b.expert_id = ep.id
      WHERE ep.user_id = auth.uid()
      AND b.user_id = profile_contact.user_id
      AND b.status IN ('pending', 'confirmed', 'completed')
    )
  );

-- Użytkownik może zarządzać własnymi danymi kontaktowymi
CREATE POLICY contact_manage_own ON profile_contact
  FOR ALL
  USING (auth.uid() = user_id);
```

---

## 6. Funkcje i wyzwalacze

### 6.0. Automatyczne tworzenie profilu po rejestracji (Supabase Auth)

Funkcja i trigger do automatycznego tworzenia rekordu w `profiles` po utworzeniu użytkownika w `auth.users`.

```sql
-- Funkcja do automatycznego tworzenia profilu
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, avatar_path, is_profile_complete)
  VALUES (
    NEW.id,
    'avatars/default-avatar.png',
    false
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger na auth.users
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION handle_new_user();
```

**Uwagi:**
- Automatyczne po rejestracji, tworzy profil z domyślnym avatarem
- `SECURITY DEFINER` wymagane dla uprawnień do `public.profiles`
- Aplikacja przekierowuje do uzupełnienia `full_name`

### 6.0b. Automatyczna aktualizacja `is_profile_complete`

Funkcja i trigger do automatycznej aktualizacji flagi `is_profile_complete`, gdy użytkownik uzupełni imię.

```sql
-- Funkcja do sprawdzania kompletności profilu
CREATE OR REPLACE FUNCTION check_profile_completeness()
RETURNS TRIGGER AS $$
BEGIN
  -- Sprawdź czy full_name jest wypełnione
  IF NEW.full_name IS NOT NULL AND trim(NEW.full_name) != '' THEN
    NEW.is_profile_complete = true;
  ELSE
    NEW.is_profile_complete = false;
  END IF;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger na profiles (przed INSERT i UPDATE)
CREATE TRIGGER check_profile_complete_on_change
  BEFORE INSERT OR UPDATE OF full_name ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION check_profile_completeness();
```

**Uwagi:**
- Automatycznie ustawia `is_profile_complete = true` gdy `full_name` wypełnione
- Aplikacja sprawdza flagę przed dostępem do funkcji

### 6.1. Automatyczna aktualizacja `updated_at`

```sql
-- Funkcja do aktualizacji updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Wyzwalacze dla tabel z updated_at
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_expert_profiles_updated_at
  BEFORE UPDATE ON expert_profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_bookings_updated_at
  BEFORE UPDATE ON bookings
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_profile_contact_updated_at
  BEFORE UPDATE ON profile_contact
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

### 6.2. RPC: cancel_booking

Funkcja do anulowania rezerwacji z walidacją roli i wymogu podania powodu dla eksperta.

```sql
CREATE OR REPLACE FUNCTION cancel_booking(
  booking_id uuid,
  reason text DEFAULT NULL
)
RETURNS void AS $$
DECLARE
  v_booking bookings;
  v_is_expert boolean;
BEGIN
  -- Pobierz rezerwację
  SELECT * INTO v_booking
  FROM bookings
  WHERE id = booking_id;

  IF NOT FOUND THEN
    RAISE EXCEPTION 'Booking not found';
  END IF;

  -- Sprawdź czy rezerwacja nie jest już anulowana
  IF v_booking.status = 'cancelled' THEN
    RAISE EXCEPTION 'Booking is already cancelled';
  END IF;

  -- Sprawdź czy użytkownik ma prawo anulować
  IF v_booking.user_id != auth.uid() THEN
    -- Sprawdź czy jest ekspertem
    SELECT EXISTS (
      SELECT 1 FROM expert_profiles
      WHERE id = v_booking.expert_id
      AND user_id = auth.uid()
    ) INTO v_is_expert;

    IF NOT v_is_expert THEN
      RAISE EXCEPTION 'Unauthorized';
    END IF;

    -- Jeśli anuluje ekspert, powód jest wymagany
    IF reason IS NULL OR trim(reason) = '' THEN
      RAISE EXCEPTION 'Cancellation reason is required for experts';
    END IF;
  END IF;

  -- Anuluj rezerwację
  UPDATE bookings
  SET
    status = 'cancelled',
    cancelled_at = now(),
    cancelled_by_user_id = auth.uid(),
    cancellation_reason = reason
  WHERE id = booking_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 6.3. RPC: deactivate_expert_profile

Funkcja do dezaktywacji profilu eksperta i anulowania wszystkich przyszłych rezerwacji.

```sql
CREATE OR REPLACE FUNCTION deactivate_expert_profile(
  expert_profile_id uuid,
  reason text
)
RETURNS void AS $$
DECLARE
  v_expert expert_profiles;
BEGIN
  -- Pobierz profil eksperta
  SELECT * INTO v_expert
  FROM expert_profiles
  WHERE id = expert_profile_id
  AND user_id = auth.uid();

  IF NOT FOUND THEN
    RAISE EXCEPTION 'Expert profile not found or unauthorized';
  END IF;

  -- Dezaktywuj profil
  UPDATE expert_profiles
  SET
    is_active = false,
    deactivated_at = now()
  WHERE id = expert_profile_id;

  -- Anuluj wszystkie przyszłe rezerwacje
  UPDATE bookings
  SET
    status = 'cancelled',
    cancelled_at = now(),
    cancelled_by_user_id = auth.uid(),
    cancellation_reason = reason
  WHERE expert_id = expert_profile_id
  AND start_at > now()
  AND status IN ('pending', 'confirmed');
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### 6.4. RPC/View: get_user_contact_info

Funkcja do pobierania pełnych danych kontaktowych użytkownika (z auth.users jako fallback dla email).

```sql
-- View do pobierania danych kontaktowych z fallback na auth.users
CREATE OR REPLACE VIEW user_contact_view AS
SELECT 
  p.id as user_id,
  p.full_name,
  COALESCE(pc.email, au.email) as email,
  pc.phone,
  au.email as auth_email
FROM profiles p
LEFT JOIN profile_contact pc ON p.id = pc.user_id
LEFT JOIN auth.users au ON p.id = au.id
WHERE p.is_active = true;

-- RPC do pobierania kontaktu użytkownika (tylko po rezerwacji)
CREATE OR REPLACE FUNCTION get_user_contact(target_user_id uuid)
RETURNS TABLE (
  user_id uuid,
  full_name text,
  email text,
  phone text
) AS $$
BEGIN
  -- Sprawdź czy użytkownik ma prawo zobaczyć dane kontaktowe
  IF NOT EXISTS (
    -- Jako użytkownik: mam rezerwację z tym ekspertem
    SELECT 1 FROM bookings b
    JOIN expert_profiles ep ON b.expert_id = ep.id
    WHERE b.user_id = auth.uid()
    AND ep.user_id = target_user_id
    AND b.status IN ('pending', 'confirmed', 'completed')
    
    UNION
    
    -- Jako ekspert: ten użytkownik ma rezerwację ze mną
    SELECT 1 FROM bookings b
    JOIN expert_profiles ep ON b.expert_id = ep.id
    WHERE ep.user_id = auth.uid()
    AND b.user_id = target_user_id
    AND b.status IN ('pending', 'confirmed', 'completed')
    
    UNION
    
    -- Sprawdzam własne dane
    SELECT 1 WHERE auth.uid() = target_user_id
  ) THEN
    RAISE EXCEPTION 'Unauthorized to view contact information';
  END IF;
  
  -- Zwróć dane kontaktowe z fallback na auth.users
  RETURN QUERY
  SELECT 
    ucv.user_id,
    ucv.full_name,
    ucv.email,
    ucv.phone
  FROM user_contact_view ucv
  WHERE ucv.user_id = target_user_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

**Uwagi:**
- View z fallbackiem email na `auth.users`
- RPC sprawdza uprawnienia (rezerwacja lub własne dane)
- `SECURITY DEFINER` dla dostępu do `auth.users`

---

## 7. Seed Data

### 7.1. Kategorie

Predefiniowane kategorie dla MVP (płaska struktura, bez hierarchii).

```sql
INSERT INTO categories (slug, name, description, display_order) VALUES
  ('law-and-finance', 'Prawo i Finanse', 'Porady prawne, finansowe, podatkowe', 1),
  ('health-and-medicine', 'Zdrowie i Medycyna', 'Konsultacje zdrowotne, medyczne, wellness', 2),
  ('technology', 'Technologia', 'Programowanie, IT, cyberbezpieczeństwo, DevOps', 3),
  ('education-and-development', 'Edukacja i Rozwój', 'Coaching, mentoring, rozwój osobisty', 4),
  ('business', 'Biznes', 'Strategie biznesowe, marketing, zarządzanie', 5),
  ('other', 'Inne', 'Pozostałe kategorie', 6);
```

### 7.2. Komentarze do tabel (dokumentacja w bazie danych)

```sql
-- Dokumentacja dla przyszłych programistów
COMMENT ON TABLE profiles IS 
'Tabela zintegrowana z Supabase Auth. Rekordy są automatycznie tworzone przez trigger on_auth_user_created po rejestracji użytkownika w auth.users i usuwane przez CASCADE. Użytkownicy mogą edytować własne profile przez RLS policies.';

COMMENT ON COLUMN profiles.is_profile_complete IS 
'Flaga automatycznie ustawiana na true przez trigger check_profile_completeness() gdy full_name jest wypełnione. Używana do wymuszenia uzupełnienia profilu przed dostępem do funkcji aplikacji.';

COMMENT ON TABLE expert_profiles IS 
'Dane eksperckie dla użytkowników, którzy chcą oferować swoje usługi. Relacja 0..1 do profiles - nie każdy użytkownik musi być ekspertem.';

COMMENT ON TABLE bookings IS 
'Rezerwacje sesji między użytkownikami a ekspertami. Exclusion constraint zapobiega kolizjom terminów per ekspert.';

COMMENT ON TABLE profile_contact IS 
'Dane kontaktowe z ostrzejszym dostępem RLS. Email z fallbackiem na auth.users.email (przez view user_contact_view).';
```

---

## 8. Dodatkowe uwagi i decyzje projektowe

### 8.0. Integracja z Supabase Auth

**Tabela `profiles` jest zintegrowana (nie "zarządzana") z Auth:**
- Automatyczne tworzenie przez trigger `on_auth_user_created`
- Automatyczne usuwanie przez CASCADE
- Edytowalna przez użytkownika (RLS) - różnica od `auth.users`

**Flow rejestracji:**
1. Signup w Auth → trigger tworzy profil z domyślnym avatarem
2. Użytkownik uzupełnia `full_name` → `is_profile_complete = true` (trigger)
3. Flaga `is_profile_complete` blokuje dostęp do funkcji dopóki profil niepełny

**Dane kontaktowe:**
- Email: `auth.users.email` (główny) + `profile_contact.email` (alternatywny)
- View `user_contact_view` używa fallback na `auth.users.email`

### 8.1. Kluczowe decyzje projektowe

**Normalizacja:** 3NF, brak denormalizacji w MVP

**Strefy czasowe:** `timestamptz` wszędzie, `expert_profiles.timezone` dla DST

**Soft delete:** `is_active` + `deactivated_at` w `profiles` i `expert_profiles` zachowuje historię

**Bezpieczeństwo:** RLS na wszystkich tabelach, RPC dla złożonych reguł biznesowych

**Audyt:** `created_at`/`updated_at` + pola anulowania (`cancelled_at`, `cancelled_by_user_id`, `cancellation_reason`)

**Wydajność:** Indeksy pod dashboardy/listy, exclusion constraint dla kolizji

**Walidacja dostępności:** API + constraint na kolizje (brak pełnego wymuszania w DB)

**Długość sesji:** Stała 60 min w MVP

**Avatar:** Tylko `avatar_path` (ścieżka do Supabase Storage), bez metadanych

**Admin:** Brak w MVP (możliwość przez JWT custom claims w przyszłości)

---

## 9. Podsumowanie

Schemat zaprojektowany z myślą o:

✅ **Integracji z Supabase Auth** - automatyczne tworzenie/usuwanie profili, dwuetapowa rejestracja  
✅ **Zgodności z PRD** - wszystkie wymagania funkcjonalne wspierane  
✅ **Bezpieczeństwie** - RLS policies + RPC dla złożonych reguł  
✅ **Wydajności** - indeksy pod kluczowe zapytania, exclusion constraint  
✅ **Integralności** - foreign keys, constraints, soft delete  
✅ **Skalowalności** - 3NF, możliwość rozszerzenia  
✅ **Prostoty MVP** - minimalistyczne podejście  

**Gotowy do implementacji jako migracje Supabase.**

