---
sidebar_position: 4
---

# Angular

## Installation

```bash
npm install recordrtc
npm install typeless-stt-angular
```

Source code: [https://github.com/typeless-dev/typeless-stt-angular](https://github.com/typeless-dev/typeless-stt-angular)

## Quickstart

The module `typeless-stt-angular` provides a service `AudioRecordingService` that can be used to record audio and get real-time transcription results.

```ts
import { Component } from "@angular/core";
import { RouterOutlet } from "@angular/router";
import { AudioRecordingService } from "typeless-stt-angular";

@Component({
  selector: "app-root",
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: "./app.component.html",
  styleUrl: "./app.component.css",
})
export class AppComponent {
  title = "test-typeless-stt-angular";
  constructor(private audioRecordingService: AudioRecordingService) {}

  startRecording() {
    this.audioRecordingService.startRecording({
      websocketUrl: <WEBSOCKET_URL>,
      language: <SELECTED_LANGUAGE>,
      hotwords: <HOTWORDS>,
      manualPunctuation: <MANUAL_PUNCTUATION>,
      onNewResult: (result) => {
        console.log("Result: ", result);
      },
      onStop: (entireAudioBlob: Blob, callKey: string) => {
        console.log("Entire audio blob: ", entireAudioBlob);
        console.log("Call key: ", callKey);
      },
      callKey: "my-call-key",
    });
  }

  stopRecording() {
    this.audioRecordingService.stopRecording();
  }
}
```

## Controls

- `startRecording`: `function (params) => res` - Begins the audio recording.

  - `params`: `dict` - The parameters of the recording. It contains the following fields:

    - `websocketUrl`: `string` - The URL of the Websocket server.
    - `language`: `string` - Language code for speech recognition. See [Supported languages](/docs/stt/languages) for a list of supported languages.
    - `hotwords`: `string[]` - Custom hotwords for enhanced recognition.
    - `manualPunctuation`: `boolean` - Enable or disable manual punctuation.
    - `onNewResult`: `function (result) => void` - Callback for new transcription results.
      - `result`: `dict` - The new transcription result.
        **Either**:
        - `{voice_activity: boolean}` - Voice activity detection result.
        - `{transcript: Transcript[], uid: string, manual_punctuation: boolean, call_key: string}` - Transcription result.
          - `Transcript`: `dict` - A transcription result.
            - `transcript`: `string` - The transcription text.
            - `timestamp`: `number` - The timestamp of the transcription in seconds.
          - `uid`: `string` - The unique identifier of the transcription.

    :::tip Tip
    Use this function to update the UI with the latest transcription results.
    :::

    - `onStop`: `function (entireAudioBlob, callKey) => void` - Callback when recording stops.
      - `entireAudioBlob`: `Blob` - The entire audio recording as a Blob.
      - `callKey`: `string` - The unique call key for the recording.

    :::tip Tip
    You can use this callback for analytics purposes (audio duration, call key, etc.).
    :::

    - `callKey`: `string` - The unique call key for the recording.

  - **Returns**:
    - `res`: `Promise<{error: "already_recording" | "already_starting" | "stopping" | "", microphoneLabel: string}>`
      - `microphoneLabel`: The label of the microphone used for recording or empty string if startRecording fails.
      - `error`: The reason of the error if startRecording fails, empty string otherwise.

- `stopRecording`: `function () => stopStatus` - Stops the audio recording and triggers the onStop callback.
  - **Returns**:
    - `stopStatus`: `Promise<"already_stopping" | "finishing" | "instant_kill" | "not_started">` - The status of the stop operation. If the stop operation is successful, the status will be `finishing`.

:::tip Tip

- `callkey` can be used to simplify bringing the new transcription results to the right place, and for potential analytics purposes, by identifying the button that triggered the recording.
  :::

## Parameters
