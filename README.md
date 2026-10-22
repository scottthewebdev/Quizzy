# Quizzy

Quizzy is an ASP.NET Core Blazor web application for creating, managing, taking, and reviewing quizzes.

## Tech Stack

- ASP.NET Core 10 (Blazor Web App, Interactive Server)
- Entity Framework Core
- ASP.NET Core Identity (cookie authentication)
- SQL database via EF Core provider configuration
- Tailwind-based UI styling

## Dependencies

### NuGet packages

- `BCrypt.Net-Next` (4.0.3)
  - Password hashing support utilities.
- `Microsoft.AspNetCore.Identity.EntityFrameworkCore` (10.0.0)
  - ASP.NET Core Identity integration with EF Core stores.
- `Microsoft.EntityFrameworkCore.Design` (10.0.0)
  - EF Core design-time tooling (migrations/scaffolding).
- `Microsoft.EntityFrameworkCore.SqlServer` (10.0.0)
  - SQL Server provider for EF Core.

### Build/tooling dependencies

- **.NET SDK** for `net10.0` target framework.
- **Node.js + npx** to run Tailwind CLI during build.
- **Tailwind CSS CLI** (invoked via `npx tailwindcss`) to generate `wwwroot/css/tailwind.css` from `Styles/input.css`.

## Features

### User Features

- **Authentication**
  - Register and sign in
  - Persistent cookie-based login
  - Sign out endpoint
- **Quiz selection and participation**
  - Browse available quizzes
  - Start a quiz from the selector page
  - Dark-themed quiz-taking UI support in `TakeQuiz`
- **Attempts history** (`/quiz/attempts`, authenticated)
  - Non-admin users see only their own completed attempts
  - Admin users can view all completed attempts
  - Attempts list includes quiz, owner, timestamps, and score
- **Quiz review**
  - Review submitted answers and quiz context
- **Account management**
  - Update username/email
  - Change password
  - Session refresh flow supported through a dedicated HTTP endpoint

### Admin Features

- **Admin dashboard** (`/admin`, Admin role required)
  - View quizzes and question counts
  - Delete quizzes
  - Import quizzes from JSON
  - Generate/export a template JSON payload
  - View recent attempt reporting with owner details (username/email)
- **Create quiz** (`/quiz/create`, Admin role required)
  - Build quizzes with dynamic questions and answer options
  - Mark one or more correct answers when creating
  - Optional metadata: alternative info, reference links
  - Optional screenshot file upload
- **Edit quiz** (`/quiz/edit/{quizId}`, Admin role required)
  - Edit quiz name and metadata
  - Add/remove questions
  - Add/remove answer options
  - Mark correct answer per question

## Security and Authorization

- Identity is configured with role support.
- Admin pages are protected with `[Authorize(Roles = "Admin")]`.
- User account pages and quiz-taking flows require authentication.

## Data Model Overview

Core entities:

- `Quiz`
- `Question`
- `AnswerOption`
- `Attempt`
- `AttemptAnswer`
- `ApplicationUser` (Identity user)

Attempt tracking details:

- `Attempt` now stores `UserId` and links to `ApplicationUser`
- Attempt creation in quiz-taking flow records the authenticated user
- EF Core migration `AddUserIdToAttempts` adds:
  - nullable `Attempts.UserId`
  - index on `Attempts.UserId`
  - foreign key to `AspNetUsers(Id)`

A baseline sample quiz is seeded in `QuizDbContext`.

## Startup and Seeding

On startup the app:

1. Applies EF Core migrations (`Database.MigrateAsync()`)
2. Ensures the `Admin` role exists
3. Seeds a default admin account if missing

Default admin seed values (if not overridden by configuration):

- Username: `admin`
- Email: `admin@local.test`
- Password: `Admin@12345`

You can override these values in configuration under:

```json
"AdminSeed": {
  "Username": "...",
  "Email": "...",
  "Password": "..."
}
```

## Configuration

Connection string key used:

- `ConnectionStrings:DefaultConnection`

If not provided, the app falls back to:

- `Data Source=quiz.db`

## Run Locally

1. Restore dependencies:

```bash
dotnet restore
```

2. Run the application:

```bash
dotnet run
```

3. Open the app in a browser (URL shown in console output).

## Useful Routes

- `/` – Home
- `/register` – Register
- `/login` – Login
- `/account` – Account settings
- `/quiz/select` – Quiz selector (authenticated)
- `/quiz/attempts` – Attempts history (authenticated; scoped by role)
- `/quiz/create` – Create quiz (admin)
- `/quiz/edit/{quizId}` – Edit quiz (admin)
- `/admin` – Admin dashboard (admin)

## API Endpoints

- `POST /api/auth/login`
- `GET /api/auth/logout`
- `POST /api/auth/refresh-signin`

---

This repository is intended for quiz workflows with role-based administration and interactive Blazor UI.
