---
sidebar_position: 3
---

# Websocket API

The Websocket API serves as an interface to stream audio to the backend and receive real-time transcription results.

## URL

`wss://speech.typeless.ch/real-time-transcription`

## Usage

We show simple examples using Javascript, but the API can be used with any language that supports Websockets.

:::tip Tip

We provide a list of [SDKs](/docs/category/websocket-sdks/) to interface with the Websocket API. These SDKs are designed to facilitate easy integration of our real-time speech-to-text capabilities into various medical software systems, enhancing efficiency and accessibility.
:::

### Connection

At each new recording, client should connect to the websocket.

```javascript
const URL = "wss://speech.typeless.ch/real-time-transcription";
const audioSocket = new WebSocket(URL);
```

### Input

1. **Websocket _onopen_ callback**

Client should send a stringified JSON object containing:

- `language`: The language of the transcription. See [Supported languages](/docs/stt/languages) for a list of supported languages.
- `manual_punctuation`: Boolean. If set to true, user will be able to dictate punctuation marks. If set to false, punctuation marks will be automatically added to the transcription.
- `end_user_id`: A unique identifier for the end-user. See [Authentication](/docs/stt/authentif) for more information.
- `hotwords` (optional): Comma-separated list of hotwords to be detected in the audio file.
- `domain` (optional): Medical specialty group of the end-user. See [Supported languages](/docs/stt/languages) for a list of supported specialty groups for each language. If the specialty of the end-user is not in the supported groups, do not specify this field and the transcription will be performed without domain-specific vocabulary.

```javascript
const initialMessage = JSON.stringify({
  language: "fr",
  manual_punctuation: false,
  end_user_id: "1234567890",
  hotwords: "hotword1,hotword2",
});
audioSocket.onopen = () => {
  audiosocket.send(JSON.stringify(initialMessage));
};
```

2. **Streaming the audio**

Audio should be recorded as a **.wav** file sampled at **16khz**, **mono** (one channel), and chunked every **1 second** (16000 samples).

Audio message should be a stringified JSON object containing:

- `audio`: The audio file as a base64 string.
- `uid`: A unique identifier for the audio request (returned in the corresponding response).

```javascript
const blobToBase64 = (blob) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onloadend = () => {
      const base64String = reader.result.split(",")[1]; // Split the string on ',' and get the second part
      resolve(base64String);
    };
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
};

const audioBlob = new Blob([audioBuffer], { type: "audio/wav" });
const base64Audio = await blobToBase64(audioBlob);
const audioMessage = {
  audio: base64Audio,
  uid: "1234567890",
};

audioSocket.send(JSON.stringify(audioMessage));
```

3. **Closing the connection**

When the recording is finished, client should send a stringified JSON object containing:

- `stoppedRecording`: Boolean. If set to true, the backend will stop the transcription and close the connection.

```javascript
audioSocket.send(
  JSON.stringify({
    stoppedRecording: "true",
  })
);
```

When receiving this message, the backend may still have some transcription to finish. If client wants to wait for the final transcription, it should wait for the corresponding response (see **case 3** below).

### Output

**Websocket _onmessage_**

The websocket will send a stringified JSON object containing either:

- the `transcript` and `uid` key (**case 1**)
- the `voice_activity` key (**case 2**)
- the `finished` key (**case 3**)

- **case 1**:

```python
@dataclass
class ChunkTranscriptProd:
    transcript: str
    timestamp: float
    final: bool

@dataclass
class TranscriptionResultProd:
    uid: str
    transcript: List[ChunkTranscriptProd]

```

`transcript` is an array containing, for each _speech segment_ (between user speech pauses),

- `transcript`: the current most up-to-date transcription of the _speech segment_.
- `timestamp` as a number of seconds between start of recording and start of the _speech segment_.
- `final`: boolean indicating whether the _speech segment_ is final or not. If `final` is `true`, the transcription of the _speech segment_ will not change anymore.

`uid` is the unique identifier of the audio request.

- **case 2**:

`voice_activity` is a boolean that is sent every time the voice activity detection algorithm detects a toggle in the voice activity.

- **case 3**:

  `finished` is a boolean that is sent when the transcription is finished. It is sent after the last `transcript` message. You can now gracefully close the connection.

```javascript
audioSocket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if ("transcript" in message) {
    // Do something with the transcript
    console.log(message["transcript"]);
    console.log(message["uid"]);
  }
  if ("voice_activity" in message) {
    // Do something with the voice activity
    console.log(message["voice_activity"]);
  }
};

// Example of case 1
{
  transcript: [
    {
      transcript: "Bonjour",
      timestamp: 0.5,
      final: true,
    },
    {
      transcript: "Comment allez...",
      timestamp: 1.5,
      final: false,
    },
  ],
  uid: "1234567890",
};


// Example of case 2
{
  voice_activity: true,
};

// Example of case 3
{
  finished: true,
};
```

```

```
