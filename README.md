# SmartKeys: RAG-Powered Keyboard Plugin

## Project Overview
SmartKeys is an on-device RAG (Retrieval-Augmented Generation) keyboard plugin for Android and iOS. It indexes your personal messaging data (e.g., Outlook, WhatsApp, Teams) and suggests contextual responses using a local LLM and embeddings pipeline. The solution combines:
- A Go-based module for text processing, retrieval, and LLM orchestration using and extending [LangChain Go](https://github.com/tmc/langchaingo).
  Main Indexing metadata: mood, relationship, intent, time of the day, platform
- Native keyboard integrations on Android (Kotlin) and iOS (Swift)  
- An offline vector storage layer supporting approximate nearest neighbor (ANN) search  

## Project Structure
```
SmartKeys/
│
├── go-rag/                  # Go package: RAG logic, embedding & LLM orchestration
│   ├── processor.go
│   ├── go.mod
|   └── ...
│
├── android-keyboard/        # Android InputMethodService implementation (Kotlin)
│   ├── src/
│   └── AndroidManifest.xml
│
├── ios-keyboard/            # iOS keyboard extension (Swift)
│   ├── SmartKeyKeyboard.xcodeproj
│   └── KeyboardViewController.swift
│
├── models/                  # Scripts & configs for offline embedding models
│   ├── miniLM/              # Example: MiniLM quantized weights
|   └── gpt2/                # Example: GPT-2 quantized weights
│
├── vectorstore/             # Vector storage & ANN index
│   ├── faiss-mobile/        # FAISS bindings for mobile (optional)
│   └── go-ann/              # Custom Go ANN vectorstore implementation
│
└── tests/                   # Unit and integration tests
  ├── go-rag/              
  └── mobile-integration/  
```

## LLM & Embedding Challenges
- **Model size vs. device constraints**: Offline LLMs (e.g., GPT-2 small, DistilGPT2) must fit within 2GB; leveraging 4-bit/8-bit quantization
- **Performance & memory**: On-device inference can be slow (10–300 tokens/s); optimizing with NNAPI/CoreML/NPU acceleration
- **Embedding models**: Lightweight transformers (MiniLM, DistilBERT) producing 384–768-dimension vectors; quantizing to reduce footprint
- **Threading**: Running compute-intensive inference in background threads to keep UI responsive

## Vector Store Challenges
- **On-device ANN indexing**: Implementing FAISS-mobile or ObjectBox HNSW, with careful attention to bundle size
- **Custom Go ANN vectorstore**: Developing a lightweight [HNSW implementation in Go](https://github.com/coder/hnsw) with gomobile bindings
- **Persistence & updates**: Efficiently managing index serialization and incremental updates within storage/memory limits
- **Similarity metrics**: Balancing precision vs. recall when working with quantized vectors that may impact cosine similarity accuracy

## Testing Setup
- **Go tests**: `go test ./go-rag/...` for validating processor logic and ANN index
- **Mobile unit tests**:
  - Android: AndroidJUnitRunner instrumented tests
  - iOS: XCTest in the keyboard extension target
- **Integration tests**: Emulator/simulator end-to-end flows verifying Go-to-native bindings and response quality
- **CI**: GitHub Actions configured to run tests on each push

## Notes and Limitations
- **Keyboard extension constraints**: Limited CPU/memory allocation with strict sandboxing (iOS "Full Access" permission required)
- **Data access**: OS sandbox prevents direct chat DB reads; solution relies on user exports or cloud APIs
- **Privacy**: All processing remains local; vectors and models are encrypted with explicit user consent
- **Quality trade-offs**: Small LLMs may produce simplistic responses; implementing fallback strategies and optional cloud LLM API integration
- **Battery & thermal management**: Implementing throttling and caching to minimize battery drain and heat generation
- **Thread recognition**: Developing methods to identify chat/email context from keyboard input
