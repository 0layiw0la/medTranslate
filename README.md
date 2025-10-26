# 🩺 MedTranslate API Documentation  
**Version:** 0.1 (Internal Prototype)  
**Purpose:** Enables real-time speech-to-speech translation for doctor–patient conversations.

---

## 🔊 Endpoint Overview

**Endpoint:**  
`POST /api/v1/translate/speech-to-speech`

**Description:**  
Takes a speech input (audio file or stream) in the doctor’s language and returns translated speech output (audio + text) in the patient’s language.

---

## 🧩 Request Structure

### HTTP Method
`POST`

### Headers
| Key | Type | Required | Description |
|------|------|-----------|--------------|
| `Content-Type` | `multipart/form-data` | ✅ | Indicates file upload and parameters. |
| `Accept` | `application/json` | ✅ | Specifies that client expects a JSON response. |
| `Authorization` | `Bearer <token>` | ❌ | (Optional) For authenticated deployments. |

---

### Body Parameters (Multipart Form)

| Field | Type | Required | Description |
|--------|------|-----------|--------------|
| `audio` | `file` (WAV/MP3/FLAC) | ✅ | Speech input in doctor’s language. |
| `from_language` | `string` | ✅ | ISO code for doctor’s language (e.g. `"en"`, `"fr"`, `"es"`). |
| `to_language` | `string` | ✅ | ISO code for patient’s language (e.g. `"es"`, `"hi"`, `"ar"`). |
| `session_id` | `string` | ❌ | Unique session identifier for tracking conversation context. |
| `metadata` | `json` | ❌ | Optional JSON metadata (e.g., speaker role, timestamp). |

---

## 📤 Example Request (cURL)

```bash
curl -X POST https://api.medtranslate.health/v1/translate/speech-to-speech \
  -H "Content-Type: multipart/form-data" \
  -F "audio=@doctor_input.wav" \
  -F "from_language=en" \
  -F "to_language=es"
````

---

## 📥 Example Successful Response

```json
{
  "status": "success",
  "from_language": "en",
  "to_language": "es",
  "detected_text": "How are you feeling today?",
  "translated_text": "¿Cómo se siente hoy?",
  "audio_output_url": "https://cdn.medtranslate.health/output/123456.wav",
  "session_id": "sess_abc123",
  "timestamp": "2025-10-26T14:12:45Z"
}
```

### Notes

* `audio_output_url` provides the location of the translated speech file (can be streamed or downloaded).
* The client (front-end app) should **play this output audio** automatically in the patient’s language.

---

## ⚠️ Error Responses

| HTTP Code | Meaning                       | Example                                       |
| --------- | ----------------------------- | --------------------------------------------- |
| `400`     | Missing or invalid parameters | `{"error":"Missing audio file"}`              |
| `415`     | Unsupported media type        | `{"error":"Audio format not supported"}`      |
| `500`     | Server error                  | `{"error":"Translation service unavailable"}` |

---

## 🔁 Workflow Example (Front-End Integration)

1. Doctor presses **“Doctor Speaking”** → app records voice.
2. Client sends audio to `/translate/speech-to-speech`.
3. API returns:

   * Transcribed doctor speech (text)
   * Translated patient speech (audio + text)
4. App displays both texts in chat feed:

   ```
   Doctor (EN): How are you feeling today?
   Patient (ES): ¿Cómo se siente hoy?
   ```
5. App auto-plays translated audio via returned `audio_output_url`.

---

## 💡 Implementation Notes

* **Latency target:** < 3 seconds per round trip for real-time experience.
* **Audio formats supported:** WAV (preferred), MP3, FLAC.
* **Translation model:** Multilingual speech-to-speech (e.g. Whisper + TTS stack).
* **Security:** No persistent storage unless explicitly enabled (session ephemeral).

---

## 🧠 Example Integration Snippet (JavaScript/Frontend)

```js
async function translateSpeech(audioBlob, fromLang, toLang) {
  const formData = new FormData();
  formData.append("audio", audioBlob, "input.wav");
  formData.append("from_language", fromLang);
  formData.append("to_language", toLang);

  const res = await fetch("/api/v1/translate/speech-to-speech", {
    method: "POST",
    body: formData
  });

  const data = await res.json();
  // Display and play translated audio
  displayText(data.detected_text, data.translated_text);
  playAudio(data.audio_output_url);
}
```

---

## 🧾 Future Enhancements

* 🔁 **Streaming mode:** live bi-directional translation (WebSocket or WebRTC).
* 🩻 **Medical terminology tuning:** use healthcare-specific glossary.
* 🧠 **Auto-summary generation:** appended to session metadata for summary screen.

```
```
