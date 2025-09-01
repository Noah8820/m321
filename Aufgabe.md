# Vorgehen – Schnittstellendefinition aus Prosa (OpenAPI)

> Ziel: Die kurze Prosa („Welche Endpoints? Welche Daten (Request/Response)? Welche Verben/Response Types?“) wird **in eine überprüfbare Schnittstellendefinition** überführt und formal **validiert**.

---

## 1) Format wählen

Wir wählen **OpenAPI 3.1** (statt GraphQL), weil:
- Prosa spricht explizit von **HTTP-Methoden** und **Response Types** → passt direkt zu REST/OpenAPI.
- OpenAPI liefert sofort nutzbare Artefakte (Docs, Mocks, SDKs, Validatoren).

---

## 2) Quick-Research-Check (Struktur OpenAPI 3.1)

- **Kernblöcke:** `openapi`, `info`, `servers`, `paths`, `components`.
- **Inhaltsschwerpunkte für diese Aufgabe:**
  - *Welche Endpoints (Funktionen)* → `paths` + Operationen (`get`, `post`, …)
  - *Welche Daten (Request/Response)* → `requestBody` + `components.schemas`
  - *Welche Verben (HTTP Methods)* → `get`, `post` …
  - *Welche Response Types* → `responses` inkl. Statuscodes & Medientypen

*(Hinweis: Diese Struktur ist der Standard von OpenAPI 3.1; die untenstehende Datei folgt genau diesen Regeln.)*

---

## 3) Umwandlung der Prosa in OpenAPI (Beispiel-Umsetzung)

> Datei: **`openapi.yaml`** – minimal, präzise und valide.  
> Annahme für die Beispiel-API: Ein Client meldet **Tasks** an; die Verarbeitung geschieht asynchron.  
> (Deckt „Endpoints“, „Daten“, „HTTP-Verben“ und „Response Types“ ab.)

```yaml
openapi: 3.1.0
info:
  title: Task Producer API
  version: "1.0.0"
  description: >
    API zur Annahme von Aufgaben (Tasks). Requests werden angenommen und
    asynchron verarbeitet. Enthält beispielhafte Endpunkte gemäß Prosa:
    - Welche Endpoints? /health, /tasks
    - Welche Daten? Request/Response-Schemas unten
    - Welche Verben? GET, POST
    - Welche Response Types? 200/202/400/500 mit application/json
servers:
  - url: http://localhost:8000
paths:
  /health:
    get:
      summary: Healthcheck
      description: Prüft, ob der Dienst erreichbar ist.
      operationId: getHealth
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/HealthResponse"
  /tasks:
    post:
      summary: Auftrag einreihen
      description: Nimmt einen Auftrag entgegen und stellt ihn in eine Queue (asynchron).
      operationId: enqueueTask
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/TaskIn"
      responses:
        "202":
          description: Aufgabe akzeptiert und eingereiht (asynchrone Verarbeitung)
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/TaskQueued"
        "400":
          description: Ungültige Eingabe (Validierungsfehler)
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
        "500":
          description: Unerwarteter Serverfehler
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
components:
  schemas:
    HealthResponse:
      type: object
      additionalProperties: false
      properties:
        status:
          type: string
          example: ok
      required: [status]
    TaskIn:
      type: object
      additionalProperties: false
      properties:
        type:
          type: string
          description: Aufgabentyp (z. B. email, export, report)
          example: email
        payload:
          type: object
          description: Frei definierbares Payload-Objekt für den jeweiligen Typ
          additionalProperties: true
          example:
            to: user@example.com
            subject: Hello
            body: World
      required: [type, payload]
    TaskQueued:
      type: object
      additionalProperties: false
      properties:
        task_id:
          type: string
          format: uuid
        status:
          type: string
          enum: [queued]
        queued_at:
          type: string
          format: date-time
      required: [task_id, status, queued_at]
    ErrorResponse:
      type: object
      additionalProperties: false
      properties:
        message:
          type: string
          example: Validation failed: "type" is required
      required: [message]
```

## Abgleich Prosa ⇄ Definition

- **Endpoints:** `/health`, `/tasks`  
- **Daten:** `TaskIn` (Request), `TaskQueued` / `ErrorResponse` / `HealthResponse` (Responses)  
- **Verben:** `GET`, `POST`  
- **Response Types:** `200`, `202`, `400`, `500` (jeweils `application/json`)

---

## 4) Validierung (Syntax & Regeln prüfen)

**Option A – Online Validator**  
- `openapi.yaml` im **Swagger Editor** öffnen → Fehler/Hinweise erscheinen rechts.

**Option B – CLI (schnell lokal)**  
- **Redocly CLI** (Node.js erforderlich):  
  `npx @redocly/cli@latest lint openapi.yaml`  
- **Spectral** (Linting mit Best Practices):  
  `npx @stoplight/spectral@latest lint openapi.yaml`

**Erwartung:** Keine Syntaxfehler. Typische Lint-Hinweise (z. B. fehlende `operationId`/`description`) sind hier bereits adressiert.

---

## 5) Manuelle Qualitätskontrolle (Checkliste)

- [ ] **Prosa vollständig umgesetzt?** (alle genannten Fragen beantwortet)  
- [ ] **HTTP-Semantik korrekt?** (`POST` für Erstellen/Einreihen, `GET` idempotent)  
- [ ] **Statuscodes passend?** (`202` für asynchron akzeptiert)  
- [ ] **Schemas geschlossen?** (`additionalProperties` sinnvoll gesetzt)  
- [ ] **Beispiele vorhanden?** (erleichtert Test/Verständnis)  
- [ ] **Erweiterbarkeit bedacht?** (weitere Task-Typen via `payload` möglich)

---

## 6) Iteration

- **Lint-Warnungen abarbeiten** → erneut validieren.  
- Bei fachlichen Änderungen **zuerst Prosa anpassen**, dann **OpenAPI**.  
- Optional **Mock/Client-Gen** nutzen (Swagger, Redocly, NSwag, OpenAPI Generator) für schnelle Tests.

