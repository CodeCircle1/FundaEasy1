# Changes in this version

## Security
- Removed the Gemini API key from `appsettings.json`. **Revoke the old key** and set a new one with
  `dotnet user-secrets set "Ai:GeminiKey" "..."` (or the `Ai__GeminiKey` environment variable).
- Quiz scoring is now done on the server. The browser no longer receives `correctIndex` or the
  explanation up front; it asks `POST /Quiz/Check` per question. `POST /Quiz/Finish` calculates the
  score from the stored answers, and an attempt can only be saved once.
- Error messages: only `UserFacingException` messages (written for students) are shown. Any other
  exception is logged and the browser sees a generic message. Applies to Explain, Tutor, Quiz,
  Register, Login, password reset and Upload.

## Correctness / cleanup
- `Program.cs`: exception handler and HSTS only outside Development; the developer exception page is on in Development.
- `GeminiModel` in `appsettings.json` is blank so the code default (`gemini-flash-latest`) is used.
- Removed unused `LectureService.GetLatestLecture`.

## Not changed (deliberately)
- Async data access, EF Core, service interfaces/splitting, DATETIME2 dates, distributed session:
  larger refactors that need a compiler and a data migration. See the chat notes.
