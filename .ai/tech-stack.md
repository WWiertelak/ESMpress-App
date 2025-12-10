# Tech Stack – ESMpress (Headless Architecture)

## 1. Frontend (Client Layer)
*Architektura Headless – aplikacja kliencka komunikująca się z backendem wyłącznie przez API.*

- **Astro 5:** Główny framework frontendowy. Odpowiada za routing, Server-Side Rendering (SSR) oraz serwowanie statycznych treści. Zapewnia wysoką wydajność (Core Web Vitals) kluczową dla użytkowników mobilnych.
- **React 19:** Warstwa interaktywna osadzona w Astro (architektura "Islands"). Odpowiada za dynamiczne elementy UI, stan formularza ankiety, walidację po stronie klienta oraz komunikację z API.
- **TypeScript 5:** Zapewnia statyczne typowanie kodu, zwiększając bezpieczeństwo i ułatwiając utrzymanie spójności danych między API a frontendem.
- **Tailwind CSS 4:** Silnik stylów CSS pozwalający na szybkie tworzenie responsywnego interfejsu w podejściu *Mobile-First*.
- **Shadcn/ui:** Biblioteka dostępnych i konfigurowalnych komponentów (bazująca na Radix UI), wykorzystywana do budowy spójnego interfejsu (formularze, suwaki, modale).

## 2. Backend (API & Business Logic)
*Serwer aplikacji odpowiedzialny za logikę biznesową, bezpieczeństwo i zarządzanie danymi.*

- **Django 5:** Framework bazowy. Dostarcza panel administracyjny (Django Admin) do zarządzania kampaniami, system ORM oraz mechanizmy bezpieczeństwa.
- **Django DRF:** Warstwa API. Służy do wystawienia endpointów REST (JSON), które są konsumowane przez frontend. Wybrany ze względu na szybkość działania i natywną integrację z typami Pydantic (łatwiejsza współpraca z TypeScript).
- **PostgreSQL:** Relacyjna baza danych. Wymagana w środowisku produkcyjnym do obsługi współbieżnych zapisów (użytkownicy + workery).

## 3. Asynchronous Task Queue (Scheduler & Workers)
*Serce systemu ESM – obsługa zadań w tle i harmonogramu.*

- **Celery:** Rozproszony system kolejkowy.
  - **Celery Beat:** Harmonogram (Scheduler). Cyklicznie sprawdza bazę danych, losuje godziny wysyłki zgodnie z logiką kampanii i zleca zadania.
  - **Celery Workers:** Procesy wykonawcze. Obsługują zadania zlecone przez Beat (np. "Wyślij maila", "Unieważnij token").
- **Redis:** Broker wiadomości (Message Broker) oraz Cache. Pośredniczy w wymianie komunikatów między Django a Celery.

## 4. Usługi Zewnętrzne (External Services)
- **SendGrid API:** Serwis do wysyłki e-maili transakcyjnych. Wywoływany przez Celery Workers.

## 5. DevOps & Infrastructure
- **Docker & Docker Compose:** Konteneryzacja środowiska. Obsługa serwisów: `backend` (Django), `frontend` (Node/Astro), `worker` (Celery), `beat` (Scheduler), `db` (Postgres), `redis`.
- **GitHub Actions:** Pipeline CI/CD do automatyzacji testów i budowania obrazów.
- **DigitalOcean:** Hosting infrastruktury (Droplet) uruchamiający kontenery Docker.
