# CodeQuest — Phase 1 Backbone (Clean Base)

CodeQuest is a gamified web application that teaches students software architecture
and system workflows through interactive "castle climbs".

> ⚠️ **CURRENT STATE: BARE BACKBONE ONLY.**
> This repository has been stripped down to the **authentication & backbone
> foundation**. All five feature modules (Creator Studio, Classrooms, Hints,
> Discussion, Profile/Play) and the assessment plug-in engine were **removed**
> and are **pending implementation in upcoming sprints** — see §2.
> The only working functionality right now is: landing page, registration,
> login/logout, role-based security, and the shared UI skeleton.

---

## 1. What This Repository Contains (Backbone Only)

| Layer | Location | Purpose |
|---|---|---|
| Entry point | `src/main/java/com/codequest/CodeQuestApplication.java` | `@SpringBootApplication` main class |
| Security | `common/config/SecurityConfig.java` | Form login, BCrypt hashing, CSRF/session protection, role-based URL security, logout |
| Auth support | `common/security/CodeQuestUserDetailsService.java`, `common/security/LoginSuccessHandler.java` | Loads users for login; updates daily login streak on success |
| Public web | `common/web/HomeController.java` | Landing (`/`), login (`/login`), registration (`/register` with **TEACH-2026** teacher passcode), placeholder dashboard (`/dashboard`) |
| Error handling | `common/web/GlobalExceptionHandler.java` + `templates/error/` | Friendly 403/404/500 pages |
| Seeder | `common/seed/DemoDataSeeder.java` | Seeds **only** the two demo accounts (idempotent — runs when the users table is empty) |
| Entities | `domain/` | `User`, `Role` (auth) + the 11 feature-domain entities kept as the database foundation for upcoming sprints |
| Repositories | `repository/` | `UserRepository` (auth) + the 11 feature-domain repositories kept alongside the entities |
| UI skeleton | `templates/fragments/layout.html` | Shared navbar (role badges), flash messages, footer, `<head>` |
| Pages | `templates/home/index.html`, `templates/auth/login.html`, `templates/auth/register.html`, `templates/dashboard.html` | Landing, login, registration, post-login placeholder dashboard |
| Styles | `static/css/app.css` | Dark retro-RPG theme (backbone components only) |
| Config | `application.properties` (MySQL), `application-test.properties` (H2), `pom.xml`, `docker-compose.yml` | Database/server configuration |

---

## 2. Feature Modules — NOT BUILT (Pending Implementation)

All feature-specific controllers, services, forms, view templates, DTOs and
tests have been **removed**. They are to be rebuilt in upcoming sprints on top
of the kept entity/repository foundation:

| Feature | Owner | Java package (to create) | Templates (to create) | Status |
|---|---|---|---|---|
| **F1** Course Content Creator Studio | Hasin | `com.codequest.studio` | `templates/studio/` | ⬜ NOT BUILT |
| **F2** Classroom & Role Management | Hasin | `com.codequest.classroom` | `templates/classroom/` | ⬜ NOT BUILT |
| **F3** Three-Level Smart Hints | Adnan | `com.codequest.hints` | `templates/hints/` | ⬜ NOT BUILT |
| **F4** Level Discussion Board | Tafika | `com.codequest.discussion` | `templates/discussion/` | ⬜ NOT BUILT |
| **F5** Profile, Progress & Castle Play | Tanvir | `com.codequest.profile` | `templates/profile/` | ⬜ NOT BUILT |
| Assessment plug-in engine (MCQ + future types) | Shared | `com.codequest.assessment` | `templates/assessment/` | ⬜ NOT BUILT |

> The `domain/` entities and `repository/` interfaces these modules need are
> already present and compile — do **not** recreate them. Refer to
> `PROJECT_MAP.md` for what each historical package contained.

---

## 3. Quick Start & Prerequisites

### Required Tools
- **Java 17+** (OpenJDK 17 recommended; JDK 21/25 also works — the build targets release 17)
- **Apache Maven 3.9+**
- MySQL 8.x *or nothing at all* (H2 in-memory fallback)

```bash
java -version
mvn -version
```

### Database Options

**Option A — In-memory H2 (zero setup, recommended for a quick look):**
```bash
mvn spring-boot:run "-Dspring-boot.run.profiles=test"
```

**Option B — Local MySQL (default profile):**
1. Start your local MySQL 8 server.
2. Create the database (auto-created on boot if the user has rights):
   ```sql
   CREATE DATABASE IF NOT EXISTS codequest;
   ```
3. Credentials default to `root` / `root`; override via
   `SPRING_DATASOURCE_USERNAME` / `SPRING_DATASOURCE_PASSWORD` or
   `application.properties`.

**Option C — Docker Compose (requires Docker):**
```bash
docker compose up -d
```

### Run
```bash
mvn spring-boot:run
```
Then open **http://localhost:8080**.

> **PowerShell note:** always quote the `-D` flag —
> `mvn spring-boot:run "-Dspring-boot.run.profiles=test"` — otherwise the
> argument is split at the dot and Maven fails with
> *"Unknown lifecycle phase .run.profiles=test"*.

---

## 4. Seeded Demo Accounts & Teacher Registration

On first boot `DemoDataSeeder` creates (only if the users table is empty):

| Account | Email | Password | Role |
|---|---|---|---|
| Teacher | `teacher@codequest.com` | `password123` | `ROLE_TEACHER` |
| Student | `student@codequest.com` | `password123` | `ROLE_STUDENT` |

Registering a **Teacher** account through `/register` requires the passcode
`TEACH-2026` (`app.teacher-code` in `application.properties`). Students
register freely.

After login you are taken to the **placeholder dashboard** (`/dashboard`), which
shows your name, role, login streak, and a "Feature Modules Coming Soon" panel
listing all five pending features. The navbar shows your role badge, a
Dashboard link and a logout button. No feature UI exists yet — it arrives with
the feature sprints.

---

## 5. Common Setup Errors

1. **`Communications link failure`** — MySQL is not running on
   `localhost:3306`. Start MySQL, or use the H2 profile (Option A above).
2. **`Access denied for user 'root'@'localhost'`** — wrong password; fix
   `spring.datasource.password` in `application.properties`.
3. **`Port 8080 was already in use`** — another instance is running:
   ```powershell
   Get-NetTCPConnection -LocalPort 8080 -State Listen |
     ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
   ```
   or start on another port with `-Dspring-boot.run.arguments="--server.port=8081"`.
4. **Java version mismatch** — the build targets Java 17 (`java.version` in
   `pom.xml`); run Maven with a JDK 17+.
5. **`Invalid teacher registration code`** — use `TEACH-2026`.

---

## 6. Verifying the Build

```bash
mvn test
```
Runs the single backbone test class `CodeQuestApplicationTests`
(context boots on H2 + seeder accounts exist + BCrypt hash verifies).

---

## 7. About the Other Documentation Files

`PROJECT_MAP.md`, `PROJECT_STATUS.md`, `HANDOVER.md`, `spec.md`,
`team/SPRINTS.md` and `team/*` describe the **full pre-cleanup build** and are
kept for historical reference and sprint planning only. The current source
tree is intentionally smaller than what those documents describe.
