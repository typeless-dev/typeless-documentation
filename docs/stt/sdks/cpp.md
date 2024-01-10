---
sidebar_position: 5
---

# C++

## Install

First of all, clone [https://github.com/typeless-dev/typeless-stt-js](https://github.com/typeless-dev/typeless-stt-cpp).

Then, clone https://github.com/zaphoyd/websocketpp which is a header-only library.

Then, install portaudio and openssl:

```bash
brew install portaudio
brew install openssl
```

## Build

```bash
mkdir build && cd build

# Compiling
g++ -c ../src/STTTranscription.cpp ../src/main.cpp \
-I<PATH_TO_INCLUDES> \
-I<PATH_TO_CLONED_WEBSOCKETPP_REPO> \
-std=c++11

# Linking
g++ ./STTTranscription.o ./main.o -o client \
-L<PATH_TO_LIBS> \
-lssl -lcrypto -lportaudio \
-std=c++11

# Example:
g++ -c ../src/STTTranscription.cpp ../src/main.cpp \
-I/opt/homebrew/include \
-I/opt/homebrew/opt/openssl/include \
-I/Users/aamorel/typeless/websocketpp \
-std=c++11

g++ ./STTTranscription.o ./main.o -o client \
-L/opt/homebrew/opt/openssl/lib \
-L/opt/homebrew/opt/portaudio/lib \
-lssl -lcrypto -lportaudio \
-std=c++11
```

## Run

```bash
./client <URL> <LANGUAGE> <MANUAL_PUNCTUATION> (0|1)

# Example: ./client wss://40.114.149.255/real-time-transcription/ fr 0
```
