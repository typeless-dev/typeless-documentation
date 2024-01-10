---
sidebar_position: 2
---

# REST API

This REST API serves as an interface for transcribing audio files.

## URL

`https://speech.typeless.ch/rest-api`

## Routes

### 1. **Health Check**

- Endpoint: `/health`
- Method: `GET`
- Description: Verifies if the API is running.

```
GET /health
```

**Response:**

```json
{
  "status": "UP"
}
```

### 2. **Transcription**

- Endpoint: `/transcription`
- Method: `POST`
- Description: Initiates a transcription task for the uploaded audio file.

```
POST /transcription
```

**Parameters:**

- `file` (multipart/form-data): The audio file to transcribe (`.wav` or `.mp3`). Must be under 50MB.
- `language` (form data): The language of the transcription. See [Supported languages](/docs/stt/languages) for a list of supported languages.
- `prompt` (form data, optional): Additional text data, maximum of 500 characters.
- `time_stretch` (form data, optional): Adjusts the speed of the audio playback. Acceptable range is `1.0` to `2.0`.
- `expiration_s` (form data, optional): The expiration time in seconds for the task.
- `only_single_pass` (form data, optional): If set to `true`, the transcription will be performed in a single pass which will be faster but result in a less accurate transcription with no punctuation. If set to `false`, the transcription will be performed in two passes, which will result in a more accurate transcription. The default value is `false`.
- `hotwords` (form data, optional): Comma-separated list of hotwords to be detected in the audio file.

:::tip Tip

Use the `hotwords` parameter to specify words that are specific to the user's domain. For example, the user's institution name, current patient name, etc.
:::

**Response:**

```json
{
  "report_id": "<string>"
}
```

### 3. **Transcription Result**

- Endpoint: `/transcription/<report_id>`
- Method: `GET`
- Description: Retrieves the result of the transcription task. The task can have the following statuses:
  - `PENDING`: The task is still in progress, and results are not available.
  - `SUCCESS`: The task has completed successfully, and results are available.
  - `FAILED`: The task has failed. The failure reason will be provided.

```
GET /transcription/<report_id>
```

**Parameters:**

- `report_id` (path): The ID of the transcription task.

**Response:**

When the task is `PENDING`:

```json
{
  "task_id": "<string>",
  "task_status": "PENDING",
  "task_result": null
}
```

When the task is `SUCCESS`:

```json
{
  "task_id": "<string>",
  "task_status": "SUCCESS",
  "task_result": {
    "language": "<string>",
    "transcript": "<string>"
  }
}
```

When the task is `FAILED`:

```json
{
  "task_id": "<string>",
  "task_status": "FAILED",
  "task_result": null
}
```

### Error Handling

1. If an invalid Content-Type is provided to the Transcription endpoint:

   ```json
   {
     "error": "Invalid Content-Type, must be multipart/form-data"
   }
   ```

2. If no file is selected:

   ```json
   {
     "error": "No selected file"
   }
   ```

3. If an invalid file type is uploaded:

   ```json
   {
     "error": "Invalid file type"
   }
   ```

4. If the file size exceeds 50MB:

   ```json
   {
     "error": "File size exceeds 50MB"
   }
   ```

5. If an invalid or missing language parameter is provided:

   ```json
   {
     "error": "Invalid or missing language parameter"
   }
   ```

6. If an invalid `time_stretch` parameter is provided:

   ```json
   {
     "error": "Invalid time_stretch parameter"
   }
   ```

7. If an invalid `report_id` format is provided:

   ```json
   {
     "error": "Invalid report_id format"
   }
   ```

### Examples

1. **Health Check**
   ```
   GET /health
   ```
   **Response:**
   ```json
   {
     "status": "UP"
   }
   ```
2. **Transcription**

   ```
   POST /transcription
   ```

   **Parameters:**

   - `file`: `<attached_audio_file>`
   - `language`: `fr`

   **Response:**

   ```json
   {
     "report_id": "12345"
   }
   ```

3. **Transcription Result**

   a. When the task is `PENDING`:

   ```
   GET /transcription/12345
   ```

   **Response:**

   ```json
   {
     "task_id": "12345",
     "task_status": "PENDING",
     "task_result": null
   }
   ```

   b. When the task is `SUCCESS`:

   ```
   GET /transcription/12345
   ```

   **Response:**

   ```json
   {
     "task_id": "12345",
     "task_status": "SUCCESS",
     "task_result": {
       "language": "fr",
       "transcript": "Exemple de transcription."
     }
   }
   ```

   c. When the task is `FAILED`:

   ```
   GET /transcription/12345
   ```

   **Response:**

   ```json
   {
     "task_id": "12345",
     "task_status": "FAILED",
     "task_result": null
   }
   ```
