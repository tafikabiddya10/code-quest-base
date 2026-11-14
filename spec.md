# CodeQuest: Project Spec (text version of the PDFs)

This file is a plain-text copy of the project brief and the handwritten mockup notes, so AI agents can read it without opening PDFs.

## About the project

CodeQuest is a modern approach to learning coding. With AI, developers spend less time typing code and more time planning architecture. CodeQuest teaches students system workflows and component integration so they can build real applications without getting stuck on repetitive syntax.

### Project goals
- **Architecture-first learning:** teach high-level software design, system workflows and component integration rather than manual syntax.
- **Interactive 2D learning loop:** gamified 2D world map with interactive nodes, code parsing, data flow simulations and drag-and-drop integration sandboxes.
- **AI-assisted guidance:** beginner-friendly code breakdowns and progressive 3-tier hints through a dedicated AI microservice.
- **Track and analyze progress:** diagnostic insights for students and real-time class monitoring for instructors.

### Preferred stack
- Backend: Java with Spring Boot, Spring MVC
- View: HTML, CSS, JavaScript, Thymeleaf templates
- Database: MySQL
- Security: Spring Security

## Full feature list (20 features, the long-term vision)

1. Student Profile & Progress Tracker: completion percentage, daily login streak, progress across projects.
2. Admin & Teacher Permissions System: what students see vs what instructors can do.
3. Interactive Game Map: simple 2D map with NPCs and dialogues.
4. Walking Character & Item Pickup: arrow/WASD movement, collisions, pick up floating items.
5. Choice-Based Story Sequences: different stories and characters per project.
6. Clickable Code Inspector: break syntax into atomic parts.
7. Simple Visualizer: shows how components work together.
8. Timed Code Puzzles & Mini-Games: countdown timers, drag and drop scrambled code lines into order.
9. Drag-and-Drop Architecture Sandbox: drag items and connect them into a working project.
10. AI Code Assistant: plain-English breakdown of a confusing line of code.
11. Course Content Creator Studio: admin page where teachers type lessons, arrange map levels, paste code examples.
12. Automatic Code Checker Engine: checks puzzle solutions against answer keys and unit tests.
13. Student Diagnostic Charts: learning time, accuracy rates, weak topics.
14. Automatic Achievement Badges.
15. 3-Step Smart Hints: nudge, then concept reminder, then near-solution.
16. Level Discussion & Question Board: comments attached to map nodes or code snippets.
17. System Architecture Guide Reader: shows architecture guides, diagrams and requirements before a level.
18. User Activity & Audit Logger: logins, node unlocks, puzzle attempts, item pickups with timestamps.
19. Level Feedback & Star Rating Popup: 1 to 5 stars, difficulty ranking, comments after a module.
20. Teacher Class Progress Dashboard: class completion rates, levels students struggle with most.

## Team task distribution (from the original brief)

**Tanvir Muhtady (Game World)**
- Complex: Interactive 2D World Map & Character Movement; Story-Based Missions & Interactive Choices; Real-Time Live Coding Battles.
- Simple: Student Profile & Learning Progress (completion %, streaks, per-project progress); Achievement Badges & Rewards (teachers define badges with earning conditions, awarded automatically).

**Adnan Sakeeb (Puzzles & Checking)**
- Complex: Timed Code Arrangement Puzzles; Drag-and-Drop Software Architecture Sandbox (databases, APIs, controllers, views); Automatic Answer Checking & Test Case Evaluation.
- Simple: Three-Level Hint System (teachers write 3 hints per level: Nudge, Concept Reminder, Near-Solution; each hint has a point cost deducted when used); Level Feedback & Star Rating (1 to 5 stars, perceived difficulty, comments).

**Tafika Biddya (Understanding Code)**
- Complex: AI-Powered Code Explainer; Interactive Clickable Code Inspector (click keywords, variables, method calls; matching elements highlight); Software Component Flow Visualizer (animated View -> Controller -> Model -> Database and back).
- Simple: Architecture Guide Reader (materials shown before each level); Level Discussion Board (per level and code example).

**Hasin Jahan Ananya (Teachers & Classroom)**
- Complex: Real-Time Classroom Activity Monitor; Real-Time Class Group Chat (announcements, code snippet sharing); Student Learning Diagnostic Charts.
- Simple: Teacher Content Creator Studio (create lessons, organize into levels, arrange on map, add code examples, materials and hints); Classroom & Role Management (create classrooms, invite students with unique class codes, role-based access for teachers and students, admins control permissions).

## Handwritten mockup notes (the core idea, from the team lead)

### The castle
- A level is shown as a **castle** made of **floors**. The player starts at the bottom and **moves up** floor by floor.
- An **initial NPC** at the base **gives context** (story/background for the lesson).
- **Floors of the castle have different obstacles / monsters.**
- To get through them, students must **answer coding questions properly**.
- **This is where teachers come in.** They add:
  1. Floors
  2. Obstacles
  3. Questions to overcome the obstacles. This is where **logic puzzles or mini-games** come in.
- **At the top of the castle the student gets the whole code.** The hope is that by then the student has learned each line of the code from a **logical / code-flow standpoint**. This is where things like the **AI code explainer** come in.
- Example of how the AI should break syntax down and give context: `for i in range(20)` -> `for` = Python syntax word; `i` = variable; `range(20)` = a function that counts 20 numbers.

### A single floor
- A **monster asks you questions; without answering you cannot pass.**
- If the monster is **defeated**, the player can go to the next level (stairs/ladder up to the level above).
- Challenges can be an **MCQ** or a **puzzle**, **set by the teacher**.
- The player character stands on the floor near the stairs.

### Puzzle ideas
- A **code snippet falls** down the screen; the student **must guess the right answer before it lands**.
- Other puzzle styles: **flow connecting snippets**, **filling in the blanks**, **scrambles** (reorder lines).

### Priorities (what to build first)
- Focus first on **level design and the teacher's point of view**.
- **Analytics, class code and the student's point of view come later.**
- At the core:
  1. Teacher logs in and goes to the Creator Studio.
  2. Teacher sets up levels and adds obstacles per floor.
  3. Teacher can edit the initial NPC's dialogue if they wish.

### Teacher flow
- Teacher login -> **create account** or **login** -> **Creator Studio** -> **Level Design**.
- Level design lets the teacher: add **floors**, select **monsters**, add a **template of a puzzle**, add **questions**, and **publish the level**.
- **Constraints:** (1) a level must have at least one floor; (2) the teacher must select at least one monster.

### Extra idea
- Students could **gain snippets / items as they go along**.
