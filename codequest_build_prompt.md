# CodeQuest: Build Prompt (paste everything below into the larger model)

---

## 1. Your role

You are a senior Java/Spring Boot engineer and patient mentor. You are helping a team of **four university students who are complete beginners in Java, Spring Boot, Thymeleaf and MySQL**. They have a hard deadline: **a working demo on November 17**. Today is November 11, so there are about six days.

Your job has two parts:
1. Produce a **working, runnable codebase** (a backbone plus five features) that they can clone, run and demo.
2. **Teach them while you do it.** Every file you produce should be understandable to someone who has never seen Spring. Add short comments explaining *why* each class exists. Do not dump code without explaining where it goes and what it does.

Assume nothing is installed or known. Explain every command. When there is a choice, pick the simplest option that works, not the most "professional" one.

## 2. The project

**CodeQuest** is a gamified web app that teaches students software architecture and code flow instead of memorizing syntax.

**Core concept (this is the heart of the demo):**
- A **Level** is a **castle**. A castle has several **Floors** stacked vertically.
- Each floor has an **Obstacle**: a **monster** that blocks the way up. The student can only climb to the next floor by completing the monster's **assessment** correctly (for now a multiple-choice question; more types come later, see section 5A).
- At the top of the castle the student sees the **whole code** they have been learning line by line.
- A level begins with an **Initial NPC** who gives the story/context (the teacher can edit this dialogue).
- **Teachers** create everything: castles, floors, monsters, assessments, NPC dialogue, hints. **Students** play through published castles.

**Teacher flow:** log in (or create an account) -> Creator Studio -> Level Design -> add floors, pick a monster per floor, add assessments, add hints, publish.
**Constraints the teacher UI must enforce:** a level needs **at least one floor**, and every floor needs **at least one monster/obstacle** selected, before it can be **published**.

**Student flow:** register/login -> join a class with a class code -> see published levels -> enter a castle -> talk to the NPC -> face the monster on floor 1 -> answer (use hints if stuck) -> climb to the next floor -> reach the top and see the full code -> progress is saved and shown on the profile.

## 3. Hard scope decision (read carefully)

The original project spec lists 20 features. **We are only building the five below.** Do NOT build the 2D walking map, WASD movement, live coding battles, real-time chat, AI explainer, drag-and-drop sandbox, timers, or charts. If you feel tempted to add them, don't. A smooth teacher-creates -> student-plays loop beats many half-finished features.

The five features, with the owner (so file ownership stays separate and merge conflicts stay rare):

| # | Feature | Owner | Notes |
|---|---|---|---|
| F1 | **Teacher Content Creator Studio** (Simple #19) | Hasin | Teacher creates levels, floors, monsters, assessments, NPC dialogue, publishes. Most important feature. |
| F2 | **Classroom & Role Management** (Simple #20) | Hasin (shared with backbone) | Roles TEACHER/STUDENT, teacher creates class, unique class code, student joins by code. |
| F3 | **Three-Level Hint System** (Simple #9) | Adnan | Per-obstacle hint 1 (Nudge), 2 (Concept Reminder), 3 (Near-Solution), each with a point cost deducted from the student's score. |
| F4 | **Level Discussion Board** (Simple #15) | Tafika | Students and teachers post comments on a level; teacher can delete. |
| F5 | **Student Profile & Learning Progress** (Simple #4) | Tanvir | Completion percentage, daily login streak, per-level progress. Also owns the "play a castle" student page. |

If the team later wants to swap a feature, the rest of the design should still hold.

## 4. Technology (fixed, do not substitute)

- **Java 17** and **Maven**
- **Spring Boot 3.x** with **Spring MVC** (server-rendered, no React/Angular)
- **Thymeleaf** templates with **HTML, CSS and a little vanilla JavaScript**
- **Spring Data JPA (Hibernate)** with **MySQL 8**
- **Spring Security** (form login, BCrypt password hashing, role-based access)
- Dependencies via start.spring.io style: spring-boot-starter-web, thymeleaf, data-jpa, security, validation, mysql-connector-j, thymeleaf-extras-springsecurity6, lombok (optional; if you use Lombok, say so and explain it in one sentence)
- Use `spring.jpa.hibernate.ddl-auto=update` so tables are generated from entities. Each developer runs MySQL locally.
- Include a **seed** that creates: one teacher account, one student account, one class, and one complete sample castle (3 floors, each with a monster, an MCQ assessment and 3 hints), so the app is demo-ready immediately after startup. Implement it as a Spring `CommandLineRunner` (NOT `data.sql`, which runs before Hibernate creates the tables and cannot BCrypt passwords). It must be **idempotent**: only insert when the tables are empty, so restarting never duplicates data.

## 5. Output plan: work in phases, and stop after each

Do **not** output everything at once. Work in the phases below. At the end of each phase, give a short "how to run and verify this" checklist and **wait for me to say "continue"**.

### 5A. Extensible assessment design (IMPORTANT, a core requirement)

The teachers must be able to assign **many different kinds of assessments** through monsters, and the team wants to experiment with new kinds later (scrambled code lines, fill-in-the-blank, flow-connecting snippets, falling-snippet timed puzzles, architecture drag-and-drop, and so on) **without changing the database tables or rewriting existing code**. So design it like this:

**Data model (one generic table, not one table per type):**
- `Assessment.type`: a string/enum such as `MCQ`, `FILL_BLANK`, `SCRAMBLE`, `FLOW_CONNECT`. Stored as a string so new types need no schema change.
- `Assessment.prompt`: the question text shown to the student.
- `Assessment.content`: **JSON text** (a `TEXT`/`LONGTEXT` column, parsed with Jackson) holding the type-specific material shown to the student. Examples: MCQ -> `{"options":["A","B","C","D"]}`; SCRAMBLE -> `{"lines":["x = 1","print(x)"]}` (stored in any order); FILL_BLANK -> `{"template":"for ___ in range(20):"}`.
- `Assessment.answerKey`: **JSON text** holding the correct answer, e.g. `{"correct":2}` or `{"order":[1,0]}` or `{"blanks":["i"]}`. **Never send the answer key to the browser**; checking happens on the server only.
- `Assessment.points` and `Assessment.orderIndex` stay as normal columns.
- Hints attach to the **Obstacle** (the monster), not to the question, so every assessment type automatically works with the hint system.

**Code structure (a simple "plug-in" per type, no fancy patterns):**
- Create an interface `AssessmentHandler` with: `String getType()`, `void validateDefinition(String contentJson, String answerKeyJson)` (so teachers cannot save a broken assessment), and `CheckResult check(Assessment assessment, String submittedAnswerJson)` returning `correct`, `scoreAwarded` and a friendly `feedback` message.
- Each type is one small Spring `@Component` implementing that interface. Spring injects all handlers as a `List<AssessmentHandler>`; an `AssessmentService` keeps them in a map by type and calls the right one. Adding a type must never require editing an `if/else` or `switch` over types.
- Each type has two small Thymeleaf fragments: a **teacher editor** (`templates/assessment/{type}-editor.html`) and a **student player** (`templates/assessment/{type}-play.html`). The Creator Studio and the student play page choose the fragment by `type`.
- Every submission is saved as an `Attempt` row (JSON submitted answer, correct or not, score, hints used, time taken). This is what later powers diagnostics, the teacher monitor and badges.

**What to implement now:** only the **MCQ** handler fully, plus a tiny **FILL_BLANK** handler *only if* all five features are already finished and working (treat it as optional bonus; say clearly when you skip it). The goal now is that the plug-in structure exists and MCQ proves it works.

**Also deliver a short guide, "How to add a new assessment type in 4 steps"** (add the type name, write a handler class, write the editor fragment, write the player fragment) with a worked example using SCRAMBLE, so the team can experiment on their own after the deadline.

### Phase 0: Design (no code yet)
1. Give a **plain-English summary** of the architecture in under 200 words, explaining the flow *Browser -> Controller -> Service -> Repository -> Entity -> MySQL -> Thymeleaf template* using one concrete example from this project (e.g. "teacher saves a floor").
2. Produce an **ER diagram** in **Mermaid** (`erDiagram`) covering all entities, with primary keys, foreign keys and cardinalities. Then a table explaining each entity in one sentence.
3. Produce a **UML class diagram** in Mermaid (`classDiagram`) showing entities plus the main controllers/services/repositories and how they depend on each other.
4. Produce a **package-by-feature folder structure** (see section 6) and a **URL/route table** (method, path, who can access, what it does).
5. Produce a **simple wireframe description** (text) for each page.
6. **Extensibility self-review (mandatory):** after the diagrams, answer this question yourself in a short table: *"If we later add (a) badges, (b) a teacher class monitor, (c) a new puzzle type such as scrambled code lines, (d) diagnostic charts, and (e) a feedback/star-rating popup, which existing tables and classes would have to change?"* For each, say either "only new tables/classes are added" or name exactly what would change and why. If any answer is not "only new tables are added", **revise the design now** until it is, then show the corrected diagrams. I will ask you the same question myself after reading your answer, so be honest and specific.
7. List any assumptions you made and ask me up to 3 clarifying questions if truly necessary.

#### Suggested starting entities for Phase 0
Adjust if you see a better design, but justify it:
`User` (id, name, email, passwordHash, role, lastLoginDate, loginStreak), `Classroom` (id, name, classCode, teacher), `ClassroomMember`/many-to-many between Classroom and student Users, `Level` (id, title, description, npcDialogue, published, teacher), `Floor` (id, level, floorNumber, title), `Obstacle` (id, floor, monsterType, monsterName, introText), `Assessment` (the challenge a monster poses; see section 5A for its extensible design: id, obstacle, type, prompt, content, answerKey, points, orderIndex, createdAt), `Attempt` (id, student, assessment, submittedAnswer, correct, scoreAwarded, hintsUsed, timeTakenSeconds, createdAt; one row per try, never overwritten), `Hint` (id, obstacle, hintNumber 1..3, text, pointCost), `HintUsage` (id, student, hint, createdAt; records each hint a student revealed so its cost is deducted exactly once and hints can only be revealed in order), `Comment` (id, level, author, text, createdAt), `LevelProgress` (id, student, level, currentFloor, completed, score, hintsUsed, completedAt). Give every main entity a `createdAt` timestamp.

### Phase 1: The backbone (this is what unblocks the whole team)
Deliver a project that **runs with `mvn spring-boot:run`** and contains ONLY:
- `pom.xml`, `application.properties` (with placeholders and comments for DB user/password), and `.gitignore`
- All entity classes and Spring Data repositories from the ER diagram
- The **shared assessment plug-in infrastructure** from section 5A in its own package `com.codequest.assessment`: the `AssessmentHandler` interface, `CheckResult`, `AssessmentService` (handler registry), and the fully working **MCQ handler** with its two fragments. This is built now, in the backbone, because both the Creator Studio (F1) and the Student Play Page (F5) depend on it and two people must not both write it.
- Spring Security: registration page, login page, logout, BCrypt, roles `ROLE_TEACHER` and `ROLE_STUDENT`, redirect to a teacher home or student home after login, and access rules (teacher pages blocked for students and vice versa)
- A shared Thymeleaf **layout fragment** (navbar showing links by role, footer) and one CSS file, so every feature page looks consistent
- Teacher home and student home pages with **placeholder cards/links** for each of the five features (links can point to "coming soon" stubs)
- The seed data from section 4
- A `README.md` with exact setup steps: install Java 17, Maven, MySQL, create the database, set credentials, run, default seed logins, and how to open the app in a browser

Do **not** build any feature logic in this phase. This phase is complete when a teammate can clone, run, log in as teacher and as student, and see the home pages.

### Phase 2: Each feature as an independent vertical slice
For each feature, in this order F1 -> F2 -> F3 -> F4 -> F5, deliver the controller, service, DTO/form classes if needed, Thymeleaf templates and any small JS, and state **exactly which files are new and which existing files (if any) need a one-line change**. Features must use the Phase 1 entities/repositories and should not rewrite each other's files. **Treat each feature as its own checkpoint: stop after each one (F1, then F2, and so on) with its run-and-verify checklist, and wait for me to say "continue".**

- **F1 Creator Studio:** pages to list my levels, create/edit a level (title, description, NPC dialogue), add/reorder/delete floors, add an obstacle per floor (choose monster from a fixed list such as Goblin, Dragon, Skeleton, Slime with emoji or simple CSS art), add one or more **assessments** to each obstacle by choosing a type from a dropdown (only MCQ is available now, but the dropdown and the form must be driven by the registered handlers from section 5A, so new types appear automatically), and a **Publish** button that validates the constraints from section 2 and shows friendly error messages.
- **F2 Classroom & Roles:** teacher creates a class and gets an auto-generated unique class code; student enters the code to join; teacher sees the class roster; students only see levels that teachers of their class published.
- **F3 Hints:** teacher can enter 3 hints plus point cost per obstacle (inside the Creator Studio floor page: coordinate by adding a clearly marked include/fragment); student sees a "Hint" button that reveals hints **in order only** (1 then 2 then 3), deducts the cost from the score, and records the hint use.
- **F4 Discussion Board:** a comment section on the level page: list, add, and (teacher only) delete comments. Show author name and time.
- **F5 Profile & Progress + Student Play Page:** the student play page for a castle (NPC dialogue, then one floor at a time: monster, assessment rendered by its type, answer submitted to the server and checked by the matching handler, correct -> unlock next floor, wrong -> try again, every try saved as an `Attempt`; final floor shows the full code). The profile page shows completion percentage, current streak (update on login), and a table of levels with status and score.

### Phase 3: Integration and demo readiness
- A **manual test checklist** (step-by-step clicks) covering the full demo flow
- A **demo script** (3 to 5 minutes, who says what, which account to use)
- A list of **known limitations** and the **next features to add after the deadline**
- A **Git workflow** guide (below)

## 6. Code organization rules (to prevent team merge conflicts)

- **Package by feature**, not by layer. Example: `com.codequest.studio`, `com.codequest.classroom`, `com.codequest.hints`, `com.codequest.discussion`, `com.codequest.profile`, `com.codequest.assessment` (shared plug-in infrastructure, built in Phase 1), plus `com.codequest.common` (security, entities, layout) owned by the backbone.
- Put shared entities in `com.codequest.domain` and avoid editing them after Phase 1 unless necessary (if a change is needed, call it out loudly).
- Each feature owns its own Thymeleaf folder: `templates/studio/`, `templates/classroom/`, `templates/hints/`, `templates/discussion/`, `templates/profile/`.
- Keep controllers thin, put logic in services, put database access in repositories.
- Use clear names, small methods and short comments. Beginner readability beats cleverness. No advanced patterns (no hexagonal architecture, no microservices, no custom annotations).
- Never put passwords or secrets in committed files; use placeholders.

## 7. Git workflow to teach them

Give a short guide covering: create GitHub repo, `main` branch protected by convention, one branch per person (`feature/studio`, `feature/hints` ...), commit small and often, pull `main` daily, open a pull request, resolve a merge conflict step by step (with a tiny example), and what to do if they break something. Use plain commands and explain each.

## 8. Teaching requirements

- After each phase, add a **"What you just learned"** section of at most 5 bullets in plain language.
- Add a **glossary** of the Spring terms you used (Controller, Service, Repository, Entity, Bean, Dependency Injection, Model, Thymeleaf fragment, ddl-auto, BCrypt).
- When something might fail (MySQL not running, wrong password, port 8080 in use, Java version mismatch), list the **top 5 likely errors and their fixes** in the README.
- If I paste an error message, diagnose it step by step and give the smallest fix.

## 9. Quality bar

- The app must start cleanly and the seeded demo flow must work end to end.
- Validate form input (no empty titles, no negative costs) and show readable error messages rather than stack traces.
- Basic security: passwords hashed, CSRF left enabled with forms using Thymeleaf's `th:action`, teachers cannot access other teachers' levels, students cannot access teacher pages.
- Simple, clean, consistent styling. A "game" feel is welcome through colors, emoji or CSS monsters, but do not spend time on art.
- Do not invent libraries or APIs. If you are unsure about a version or method, say so and give the safest alternative.

## 10. Start now

Begin with **Phase 0 only**. Produce the architecture explanation, the Mermaid ER diagram, the Mermaid class diagram, the folder structure, the route table, the page wireframes and the mandatory extensibility self-review table. Then stop and wait for me to reply "continue" to begin Phase 1.
