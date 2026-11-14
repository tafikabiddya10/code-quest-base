# com.codequest — Package Map (Backbone Only)

Java entry point for the CodeQuest Spring Boot application.
Current state: **bare authentication + backbone foundation**. All feature
modules were removed and are pending implementation in upcoming sprints
(see the root `README.md` §2).

## Present packages

| Package | Contents | Role |
|---|---|---|
| *(root)* | `CodeQuestApplication` | `@SpringBootApplication` entry point |
| `common.config` | `SecurityConfig` | Form login, BCrypt, CSRF, role-based URL security |
| `common.security` | `CodeQuestUserDetailsService`, `LoginSuccessHandler` | User loading; post-login streak update + redirect to `/` |
| `common.web` | `HomeController`, `GlobalExceptionHandler`, `form/RegisterForm` | Landing/login/register (TEACH-2026 passcode) + placeholder `/dashboard` ("coming soon"), error pages |
| `common.seed` | `DemoDataSeeder` | Seeds only the two demo accounts (idempotent) |
| `domain` | 13 JPA entities (`User`, `Role` + 11 feature-domain entities) | Auth tables live; others kept as database foundation |
| `repository` | 12 Spring Data repositories (`UserRepository` + 11 feature-domain) | Auth queries live; others kept as database foundation |

## Removed / pending packages (do not exist yet)

`assessment/`, `studio/`, `classroom/`, `hints/`, `discussion/`, `profile/`
and their `form/` + `view/` sub-packages — these are the F1–F5 feature
verticals to be implemented in upcoming sprints.
