---
sidebar_position: 4
---

# Java

## Get started

Please clone the code: [https://github.com/typeless-dev/typeless-stt-java](https://github.com/typeless-dev/typeless-stt-java)

Folder contains the following files:

- `WebsocketManager.java`: The main class for recording audio and getting real-time transcription results.
- `MainActivity.kt`: A sample Android activity that uses the `WebsocketManager` class to record audio and get real-time transcription results. It logs the transcription results to the console.

## WebsocketManager

### Dependencies

- OkHttp: `implementation("com.squareup.okhttp3:okhttp:4.12.0")`

### Constructor

To use this class, please instantiate it with the following constructor:

```java
WebsocketManager(
  String websocketUrl,
  String language,
  String[] hotwords,
  boolean manualPunctuation,
  String domain,
  String endUserId,
  Websocket.MessageHandler onNewResult
)
```

- `websocketUrl`: `String` - The URL of the Websocket server.
- `language`: `String` - Language code for speech recognition. See [Supported languages](/docs/stt/languages) for a list of supported languages.
- `hotwords`: `String[]` - Custom hotwords for enhanced recognition.
- `manualPunctuation`: `boolean` - Enable or disable manual punctuation.
- `domain`: `String` - The domain of the Websocket server. See [Supported languages](/docs/stt/languages) for a list of supported domains per language. If you are unsure, use `general`.
- `endUserId`: `String` - The unique identifier of the user. See [Authentication](/docs/stt/authentif) for more information.
- `messsageHandler`: `function (String result) => void` - Callback for new transcription results.

  - `result`: `Stringified JSON` - The new transcription result.
    See [Websocket API](/docs/stt/ws_api#output) for information on result format.

  :::tip Tip
  Use this function to update the UI with the latest transcription results.
  :::

Example usage:

```javascript
// Initialize the websocketManager lazily to ensure context and other components are ready.
private val websocketManager: WebsocketManager by lazy {
    WebsocketManager(
        "wss://speech.typeless.ch/real-time-transcription",
        "fr",
        arrayOf("typeless"),
        true,
        "gynécologie",
        "12345",
        object : WebsocketManager.MessageHandler {
            override fun handleMessage(message: String) {
                Log.d("WebSocket", "Received message: $message")
            }
        }
    )
}
```

### Methods

- `startRecording`: `function () => ()` - Begins the audio recording.
- `stopRecording`: `function () => ()` - Stops the audio recording.

## MainActivity

### Permissions

The following permissions are required:

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
```
