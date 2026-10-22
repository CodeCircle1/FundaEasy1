# Funda Easy (ASP.NET Core 8 MVC)

Funda Easy, ported from the WPF desktop app to an **ASP.NET Core 8 (LTS) MVC** web app.
Upload PDF/PPTX lecture slides, get each slide explained and translated into the
11 South African languages, chat with the AI tutor, and take quizzes.

## Run it

Requirements: [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0) and SQL Server
(LocalDB comes with Visual Studio on Windows).

```bash
dotnet run
# open http://localhost:5080
```

The database and tables are created automatically on first run.

## Configuration

Everything is in `appsettings.json`, but **keep secrets out of it** (the key fields there are intentionally empty). Use user-secrets locally:

```bash
dotnet user-secrets init
dotnet user-secrets set "Ai:GeminiKey" "your-gemini-key"       # free: aistudio.google.com
# or: dotnet user-secrets set "Ai:AnthropicKey" "your-anthropic-key"
dotnet user-secrets set "Lelapa:ApiToken" "your-lelapa-token"
dotnet user-secrets set "Smtp:Host" "smtp.gmail.com"
dotnet user-secrets set "Smtp:Email" "you@gmail.com"
dotnet user-secrets set "Smtp:Password" "your-gmail-app-password"
```

On a server use environment variables instead, e.g. `Ai__GeminiKey`, `Smtp__Password`,
`ConnectionStrings__DefaultConnection`.

| Setting | Purpose |
|---|---|
| `ConnectionStrings:DefaultConnection` | SQL Server (defaults to LocalDB) |
| `Ai:Provider`, `Ai:GeminiKey`, `Ai:AnthropicKey`, `Ai:GeminiModel`, `Ai:AnthropicModel` | AI provider (without a key the app falls back to a basic offline mode) |
| `Lelapa:ApiToken` | Lelapa (Vulavula) slide translation for 8 of the 11 languages; the AI provider is used for the rest |
| `Smtp:Host/Port/Email/Password` | Password-reset OTP emails |
| `Storage:UploadPath` | Where uploaded slides are saved (default `App_Data/uploads`) |

## How the WPF screens map to MVC

| WPF | MVC |
|---|---|
| Welcome page | `Home/Index` |
| Sign up / Login / Forgot password | `Account/Register`, `Login`, `ForgotPassword` |
| Home (upload + recent) | `Home/Dashboard` + `Lectures/Upload` |
| Lecture viewer, translate, tutor | `Lectures/Open` (+ `Explain`, `Tutor`, `SetLanguage`, `DownloadTranslation`) |
| Library | `Lectures/Library` |
| Quizzes tab, quiz, results | `Quiz/Index`, `Quiz/Take` (+ `Questions`, `Finish`) |
| Profile | `Profile/Index` |

## What changed because it is now a web app

* **Sign-in** uses cookie authentication. The old static `UserSession` is gone (it would have
  been shared by every visitor). All data is scoped to the signed-in user.
* **AI settings are server-side.** The "AI connection" section on the Profile page is gone;
  API keys are never sent to the browser.
* **Uploads** are saved under `Storage:UploadPath` with random names (max 50 MB, PDF/PPTX only).
* **Hardening added**: anti-forgery tokens on every POST, login lock-out after 5 failed
  attempts (15 min), OTPs die after 5 wrong guesses, the "forgot password" page no longer
  reveals whether an email has an account.
* The database schema is unchanged apart from one new column (`PasswordResetOtps.Attempts`),
  added automatically. Existing users and password hashes keep working.
