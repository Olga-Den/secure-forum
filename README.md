# SecureForum

SecureForum to webowa aplikacja forum zbudowana na Django 5.2, uruchamiana w kontenerach Docker z PostgreSQL jako bazą danych. Projekt kładzie nacisk na bezpieczeństwo - każda warstwa aplikacji, od przechowywania haseł po nagłówki HTTP, jest skonfigurowana zgodnie z aktualnymi standardami.

## Uruchomienie

```bash
git clone https://github.com/your-username/secureforum.git
cd secureforum
docker compose up --build
```

Aplikacja będzie dostępna pod adresem `http://localhost`. Przy starcie kontener automatycznie czeka na gotowość bazy danych, wykonuje migracje i tworzy konto administratora.

## Stack technologiczny

- **Python 3.11 / Django 5.2.7** - warstwa aplikacji
- **PostgreSQL** - baza danych przez AWS
- **Argon2** - algorytm haszowania haseł (z fallbackiem na PBKDF2 i bcrypt-SHA256)
- **django-axes** - ochrona przed atakami brute-force
- **Docker + Docker Compose** - konteneryzacja

## Struktura projektu

```
secureforum/
├── forum/                  # główna aplikacja (widoki, modele, formularze)
│   ├── management/commands/
│   ├── migrations/
│   └── templates/
├── secureforum/            # konfiguracja projektu Django
├── Dockerfile
├── docker-compose.yml
├── manage.py
└── requirements.txt
```

## Endpointy

| Ścieżka | Metody | Dostęp |
|---|---|---|
| `/` | GET | zalogowany użytkownik |
| `/login/` | GET, POST | publiczny |
| `/register/` | GET, POST | publiczny |
| `/logout/` | POST | zalogowany użytkownik |
| `/create/` | GET, POST | zalogowany użytkownik |
| `/health/` | GET | publiczny |

## Modele danych

Aplikacja korzysta ze standardowego modelu `User` dostarczanego przez Django (`id`, `username`, `email`, `password`, `date_joined`, `last_login`).

Model `Post` przechowuje: `id`, `title`, `content`, `created_at` oraz relację `author` (klucz obcy do `User`).

## Bezpieczeństwo

### Hasła

Hasło musi mieć co najmniej 12 znaków i zawierać małą literę, wielką literę, cyfrę oraz znak specjalny. Walidacja odbywa się przez customowy `StrongPasswordValidator` uzupełniony o wbudowane walidatory Django (`CommonPasswordValidator`, `UserAttributeSimilarityValidator` i inne). Hasła są przechowywane jako hashe Argon2.

### Ochrona przed brute-force

Za pomocą biblioteki django-axes konto jest blokowane po 5 błędnych próbach logowania. Blokada trwa godzinę i jest identyfikowana po adresie IP. Licznik resetuje się po poprawnym zalogowaniu.

### Sesje

Sesja wygasa po 2 godzinach aktywności lub po zamknięciu przeglądarki. Ciasteczka sesji i CSRF mają ustawione flagi `HttpOnly` oraz `SameSite=Lax`.

### Nagłówki HTTP

Ustawione są nagłówki: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` oraz `Referrer-Policy: strict-origin-when-cross-origin`.

### HTTPS (środowisko produkcyjne)

W środowisku produkcyjnym włączone są: secure cookies, przekierowanie HTTP→HTTPS, HSTS z czasem 1 roku (31 536 000 sekund) oraz obsługa nagłówka `X-Forwarded-Proto`.

### Logi bezpieczeństwa

Każde istotne zdarzenie jest zapisywane wraz z nazwą użytkownika, adresem IP i user-agentem. Rejestrowane są: `USER_REGISTER`, `LOGIN_SUCCESS`, `LOGIN_FAIL`, `LOGOUT` i `POST_CREATE`. Wpisy są sanityzowane przez `sanitize_for_log()`, co zapobiega atakom Log Injection i CRLF Injection.

## Komendy zarządzające

- `wait_for_db` - czeka na dostępność bazy danych przed startem aplikacji
- `ensure_admin` - tworzy konto administratora, jeśli jeszcze nie istnieje
- `createsu` - tworzy superużytkownika Django

## Licencja

MIT
