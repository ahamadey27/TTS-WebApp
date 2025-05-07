# Project: Simplified Text-to-Speech Web Application with Custom Neural Voice Lite

**Goal:** To develop a web application that generates audio mimicking a specific voice ("Alex Hamadey") using text input (up to 500 characters). The application will feature a simple user interface (text input area, generation button) and utilize Microsoft Azure AI Speech services, specifically the Custom Neural Voice (CNV) Lite feature. This document serves as a comprehensive technical blueprint for the development, detailing system architecture, components, voice training, deployment, and key considerations to guide the development process and ensure all requirements are met. The project aims for simplification, leveraging CNV Lite's reduced data and training overhead.

# Components

## Environment/Hosting
- **Local Development Machine (Assumed)**
  - .NET SDK (.NET 9 or higher)
  - IDE: Visual Studio / Visual Studio Code (or preferred .NET IDE)
  - Git for version control
- **Cloud Hosting:** Azure App Service (F1 Free Tier recommended for initial deployment)
- **Text-to-Speech Service:** Azure AI Speech
- **Custom Voice Management:** Azure Speech Studio

## Software Components

### Web Application Backend & Frontend
- **Backend Framework:** ASP.NET Core with .NET 9
- **Frontend Framework:** ASP.NET Core Razor Pages
- **Language:** C#
- **Frontend Rendering:** Razor Syntax, HTML5, CSS, JavaScript

### Text-to-Speech Integration
- **Service:** Azure AI Speech
- **SDK:** `Microsoft.CognitiveServices.Speech`
- **Custom Voice Model:** Azure Custom Neural Voice (CNV) Lite (trained for "Alex Hamadey")

---

# System Architecture & Data Flow

## High-Level Architecture
The system comprises a client-side interface (web browser), a backend web application hosted on Azure App Service, and the Azure AI Speech service for TTS conversion using the custom neural voice.

## Data Flow for Voice Generation
1.  **User Input:** User types text (max 500 characters) into a `<textarea>` on an ASP.NET Razor Page.
2.  **Request Initiation:** User clicks a "Generate Voice" button. Client-side JavaScript captures the text.
3.  **API Call:** An asynchronous HTTP POST request (with JSON payload `{"text": "User input"}`) is sent from the client to a backend ASP.NET Core API endpoint (e.g., `/api/generate-voice`) hosted on Azure App Service.
4.  **Backend Processing:** The API controller receives the request, validates input.
5.  **Speech Service Interaction:**
    * Initializes `SpeechConfig` with Azure AI Speech subscription key, region, custom voice name ("Alex Hamadey" specific), and its unique `EndpointId`.
    * Creates a `SpeechSynthesizer` instance.
    * Calls `SpeakTextAsync()` with the user's text.
6.  **Audio Synthesis:** Azure AI Speech service processes the request at the custom voice endpoint, synthesizes audio using the "Alex Hamadey" CNV Lite model.
7.  **Audio Data Return:** `SpeakTextAsync()` returns `SpeechSynthesisResult`. If successful, `result.AudioData` (byte[]) contains the audio.
8.  **API Response:** The API controller returns the audio data (e.g., `audio/wav`) in the HTTP response.
9.  **Client-Side Playback:**
    * Client-side JavaScript receives the audio data (as a Blob).
    * Creates an object URL using `URL.createObjectURL(audioBlob)`.
    * Sets this URL as the `src` for an HTML5 `<audio>` element.
    * Calls `audioElement.play()` or offers a download.

---

# Scope

## In Scope:
- Simple web UI: single text box (approx. 5 rows, max 500 chars), one generation button.
- Client-side logic: capture text, initiate backend request.
- ASP.NET Core backend API: receive text, interact with Azure AI Speech.
- CNV Lite Integration ("Alex Hamadey's" voice):
    - Guidance on CNV Lite training data (20-50 utterances from Microsoft scripts).
    - Voice talent consent statement process.
    - CNV Lite model training in Azure Speech Studio.
    - CNV Lite model deployment to a custom voice endpoint.
- Audio generation using the deployed custom voice.
- Streaming/providing audio to client for playback/download.
- Deployment to Azure App Service (F1 Free tier initially).
- Basic error handling and user feedback.

## Out of Scope (for this iteration):
- Advanced user authentication/account management.
- Persistent storage of generated audio files (beyond immediate use).
- Complex UI features beyond specified elements.
- Speech-to-text (transcription).
- Multi-language support (beyond the training language, assumed English).
- Automated CI/CD pipelines (manual/VS Publish is acceptable).
- Advanced SSML features (beyond basic text synthesis with custom voice name).

---

# Development Plan (MVP)

## Phase 1: Azure Resource Setup & CNV Lite Creation
- [ ] Create/Confirm Azure Subscription.
- [ ] Create Azure AI Speech service resource (consider F0 tier for setup, noting cost uncertainties for CNV Lite).
- [ ] Access Speech Studio.
- [ ] Create a new CNV Lite project for "Alex Hamadey's" voice (e.g., en-US).
- [ ] Perform microphone setup and online recording of 20-50 utterances using Microsoft-provided scripts (Voice Talent: Alex Hamadey).
- [ ] Train the CNV Lite model.
- [ ] Review and test the trained model's audio output in Speech Studio.
- [ ] Record and submit Alex Hamadey's voice talent consent statement.
- [ ] Deploy the trained CNV Lite model to a dedicated endpoint.
- [ ] Securely note the Endpoint ID, Speech Resource Key, and Region for backend configuration.

## Phase 2: Backend Development (ASP.NET Core API)
- [ ] Create a new ASP.NET Core Web App (Razor Pages) project (.NET 9).
- [ ] Add an API Controller (e.g., `VoiceController.cs`) with a POST endpoint (e.g., `/api/generate-voice`).
- [ ] Define a request model for text input (e.g., `TextInputModel` with `Text` property and `[MaxLength(500)]` validation).
- [ ] Integrate `Microsoft.CognitiveServices.Speech` SDK.
- [ ] Implement `SpeechConfig` initialization using `IConfiguration` to load `SPEECH_KEY`, `SPEECH_REGION`, `CUSTOM_VOICE_NAME`, `CUSTOM_VOICE_ENDPOINT_ID` from App Settings.
- [ ] Implement logic to create `SpeechSynthesizer` (with `AudioConfig` as `null` for in-memory audio).
- [ ] Implement `SpeakTextAsync()` call and process `SpeechSynthesisResult`.
- [ ] Handle successful audio generation: return `File(result.AudioData, "audio/wav", "generated_voice.wav")`.
- [ ] Implement error handling for `ResultReason.Canceled` (log `SpeechSynthesisCancellationDetails`) and other exceptions, returning appropriate HTTP status codes and error messages.
- [ ] Configure `Program.cs` for API controllers.

## Phase 3: Frontend Development (ASP.NET Core Razor Page)
- [ ] Develop the UI on `Pages/Index.cshtml`:
    - [ ] `<textarea id="textToSpeak" rows="5" maxlength="500">`.
    - [ ] `<button id="generateVoiceButton">`.
    - [ ] `<audio id="audioPlayback" controls>`.
    - [ ] `<div id="statusMessage">` for feedback.
- [ ] Implement client-side JavaScript (e.g., in `wwwroot/js/site.js` or inline script in `Index.cshtml`):
    - [ ] Event listener for `generateVoiceButton` click.
    - [ ] Client-side input validation (non-empty, length check as a fallback).
    - [ ] Asynchronous `fetch` POST request to `/api/generate-voice` with JSON body `{"text": "..."}`.
    - [ ] Handle API response:
        - [ ] Success (200 OK): get audio blob, create object URL, set to `audioPlayback.src`, call `audioPlayback.play()`.
        - [ ] Error: display error message in `statusMessage`.
    - [ ] Implement loading indicators (disable button, "Generating..." message).
- [ ] Basic styling for the page.

## Phase 4: Deployment to Azure App Service
- [ ] Create Azure App Service Plan (F1 Free tier, Windows or Linux).
- [ ] Create Azure App Service instance configured for .NET 9.
- [ ] Configure Application Settings in Azure App Service:
    - [ ] `SPEECH_KEY`
    - [ ] `SPEECH_REGION`
    - [ ] `CUSTOM_VOICE_NAME`
    - [ ] `CUSTOM_VOICE_ENDPOINT_ID`
- [ ] Deploy the ASP.NET Core application (e.g., using Visual Studio Publish, Azure CLI).
- [ ] Configure `Program.cs` for HTTPS redirection, static files, and Razor Pages routing.

## Phase 5: Testing & Refinement
- [ ] Conduct thorough end-to-end functional testing.
- [ ] Test voice quality and naturalness.
- [ ] Test error handling for various scenarios.
- [ ] Perform basic performance observation (response times).
- [ ] Monitor Azure App Service F1 tier CPU usage.
- [ ] Monitor Azure costs, especially related to CNV Lite endpoint hosting and synthesis.
- [ ] Refine UI/UX based on testing.

---

# Testing Checklist (MVP)
- [ ] Project builds and runs locally without errors.
- [ ] CNV Lite model ("Alex Hamadey") trains and deploys successfully in Speech Studio.
- [ ] Consent statement for Alex Hamadey is processed.
- [ ] Frontend UI (text area, button, audio player) renders correctly.
- [ ] Text input (up to 500 chars) is accepted.
- [ ] "Generate Voice" button triggers API call.
- [ ] Backend API receives text and calls Azure AI Speech service.
- [ ] Audio is successfully generated using the "Alex Hamadey" custom voice.
- [ ] Audio plays back in the browser.
- [ ] Client-side validation (empty text, length) provides feedback.
- [ ] Server-side validation (500 char limit) prevents oversized requests.
- [ ] Correct audio content type (e.g., `audio/wav`) is returned.
- [ ] User-friendly status messages are displayed (e.g., "Generating...", "Error...").
- [ ] Error handling:
    - [ ] Invalid Azure Speech key/region/endpoint configuration.
    - [ ] CNV Lite service errors.
    - [ ] Network issues between client and server.
- [ ] Application deploys to Azure App Service and runs.
- [ ] Application Settings on Azure App Service are correctly picked up by the application.
- [ ] HTTPS is enforced.

---

# Key Considerations

- **Character Limits & Billing:**
    - Enforce 500-character limit client-side and server-side.
    - Azure AI Speech bills per character synthesized (including punctuation). CNV Lite synthesis is typically at a higher rate than standard neural voices.
- **Error Handling:**
    - Frontend: Clear user feedback for input errors, API failures.
    - Backend: Robust try-catch, log detailed errors (especially `SpeechSynthesisCancellationDetails.ErrorDetails`), return appropriate HTTP status codes.
- **Security:**
    - **API Key Management:** Store `SPEECH_KEY`, `CUSTOM_VOICE_ENDPOINT_ID` in Azure App Service Application Settings (not in code or `appsettings.json` committed to repo).
    - **Input Sanitization:** While Azure Speech handles text, primary concern is length enforcement.
    - **HTTPS:** Enforce via `UseHttpsRedirection()`.
    - **XSRF/CSRF:** Consider anti-forgery tokens if API evolves, though less critical for this simple, single-purpose endpoint.
- **Cost Management (Crucial):**
    - **CNV Lite Training:** One-time cost (<1 compute hour).
    - **CNV Lite Endpoint Hosting:** **Recurring hourly cost while endpoint is active.** This is a significant ongoing cost.
    - **CNV Lite Synthesis:** Per-character cost, higher than standard neural voices.
    - **F0 Tier Uncertainty:**
        - It's **highly uncertain** if the F0 Speech tier's "1 free custom model endpoint hosting" applies to CNV Lite TTS endpoints. Assume it does **not** and will be billable.
        - It's **highly uncertain** if the F0 Speech tier's "0.5M free Neural TTS characters" apply to CNV Lite synthesis. Assume it does **not** and will be billable at custom voice rates.
    - **Azure App Service (F1):** Free, but with limitations (60 CPU mins/day). Monitor usage; upgrade to B1 (Basic) if needed (incurs cost).
    - **Recommendation:** **Suspend or delete the CNV Lite endpoint in Speech Studio when not actively developing/demonstrating to avoid continuous hosting charges.**
- **User Experience (UX):**
    - Provide clear feedback during audio generation (loading states, disabled button).
    - Ensure audio player is accessible.
    - Clearly communicate input constraints (500 chars).
    - User-friendly error messages.
- **CNV Lite and "Business Use" for Portfolio:**
    - Microsoft's terms for CNV Lite state that "business use" requires an application for full CNV access. A public portfolio might be interpreted as such.
    - **Recommendations:** Seek clarification from Microsoft, consider applying for full CNV access, or clearly label the project as a "technical demonstration for evaluation/personal experimentation" if used publicly.
- **Critical Next Actions:**
    - **Clarify CNV Lite Costs on F0 Tier:** Monitor Azure bill/consumption carefully after initial tests or contact Azure Support.
    - **Proactively Suspend/Delete CNV Endpoint:** Make this a regular practice when the endpoint is not in active use to manage costs.

