# RAG-Based AI Teaching Assistant

An AI Teaching Assistant built using **Retrieval-Augmented Generation (RAG)** to answer student queries using information extracted from educational videos.

The system converts tutorial videos into searchable text, generates semantic embeddings using **BGE-M3**, retrieves the most relevant information for a student's query, and uses **GPT-5** to generate a contextual answer.

---

## Project Overview

The goal of this project is to build an AI Teaching Assistant that can understand and answer questions based on the content of educational videos.

Instead of providing the complete tutorial content to the LLM for every query, the system retrieves only the most relevant information and provides it as context to the LLM.

This reduces unnecessary context and allows the LLM to generate answers based on the relevant educational material.

---

## System Pipeline

```text
Tutorial Videos
       ↓
Video → MP3
       ↓
Whisper Transcription
       ↓
Text / JSON
       ↓
Chunking
       ↓
Merge Every 5 Chunks
       ↓
BGE-M3 Embeddings
       ↓
Embedding Storage
       ↓
       ┌─────────────────────┐
       │    Student Query   │
       └──────────┬──────────┘
                  ↓
            BGE-M3 Embedding
                  ↓
          Cosine Similarity
                  ↓
        Relevant Chunk Retrieval
                  ↓
        Query + Retrieved Context
                  ↓
                GPT-5
                  ↓
            Final Answer
```

---

## Technologies Used

* **Python**
* **FFmpeg** — video to audio conversion
* **Whisper** — speech-to-text transcription
* **Pandas** — data processing and storage
* **BGE-M3** — text embedding generation
* **Cosine Similarity** — semantic retrieval
* **Joblib** — storing processed embeddings
* **GPT-5** — answer generation
* **Ollama + Llama 3.2** — initial LLM implementation

---

## RAG Pipeline

### 1. Video Collection

Educational  videos are collected as the source of knowledge for the Teaching Assistant.

The videos are placed in the `videos` folder.

### 2. Video to Audio

The video files are converted into MP3 audio using FFmpeg.

```text
Video → MP3
```

This is handled by:

```text
video_to_mp3.py
```

### 3. Audio Transcription

The MP3 files are converted into text using Whisper.

```text
MP3 → Transcript
```

The resulting information is stored in JSON format for further processing.

### 4. Text Chunking

The transcripts are divided into smaller chunks so that relevant portions of the tutorial can be retrieved efficiently.

### 5. Embedding Generation

The text chunks are converted into numerical vector representations using **BGE-M3**.

```text
Text Chunk
    ↓
BGE-M3
    ↓
Embedding Vector
```

The embeddings capture the semantic meaning of the corresponding text.

### 6. Embedding Storage

The processed data and embeddings are stored for later retrieval.

The project uses a Pandas-based approach along with Joblib for storing the processed embedding data.

### 7. User Query

When a student asks a question, the query is converted into an embedding using the same BGE-M3 model.

### 8. Semantic Retrieval

The query embedding is compared with the stored chunk embeddings using **cosine similarity**.

The most relevant chunks are selected as context for the LLM.

```text
Student Query
      ↓
Query Embedding
      ↓
Cosine Similarity
      ↓
Rank Chunks
      ↓
Top Relevant Chunks
```

### 9. Answer Generation

The retrieved context and the student's question are provided to the LLM.

GPT-5 uses this retrieved context to generate the final answer.

---

# Optimizations

Two major optimizations were implemented in the project.

## Optimization 1 — Chunk Merging

Initially, the transcript was divided into a large number of smaller chunks.

To improve the efficiency of the retrieval pipeline, **every 5 consecutive chunks were merged into a single larger chunk** before embedding generation.

```text
Chunk 1 ─┐
Chunk 2  │
Chunk 3  ├──→ Merged Chunk 1
Chunk 4  │
Chunk 5 ─┘

Chunk 6 ─┐
Chunk 7  │
Chunk 8  ├──→ Merged Chunk 2
Chunk 9  │
Chunk 10 ┘
```

### Benefits

* Reduced the total number of chunks.
* Reduced the number of embeddings that needed to be generated.
* Reduced the retrieval search space.
* Preserved more contextual information within each retrieved chunk.

---

## Optimization 2 — LLM Response Latency

The initial implementation used **Llama 3.2 through Ollama** for answer generation.

However, the response generation was relatively slow in the existing setup, resulting in higher response latency.

Therefore, the LLM component was replaced with **GPT-5**.

```text
Before:

Retrieved Context
       ↓
Llama 3.2 + Ollama
       ↓
Answer


After:

Retrieved Context
       ↓
GPT-5
       ↓
Answer
```

The GPT-5-based implementation produced responses in a significantly shorter time in our setup, improving the responsiveness of the Teaching Assistant.

---

## Project Structure

```text
RAG-AI-Teaching-Assistant/
│
├── README.md
│
├── video_to_mp3.py
├── mp3_to_json.py
├── preprocess_json.py
├── process_incoming.py
│
├── response.txt
```

> The exact project structure may vary depending on the version of the implementation.

---

## How to Use

### Step 1 — Collect Your Videos

Place your educational tutorial videos in the appropriate video directory.

### Step 2 — Convert Videos to MP3

Run:

```text
video_to_mp3.py
```

to convert the tutorial videos into MP3 audio files.

### Step 3 — Convert MP3 to JSON

Run:

```text
mp3_to_json.py
```

to transcribe the audio and generate the required JSON data.

### Step 4 — Preprocess the Data

Run:

```text
preprocess_json.py
```

to preprocess the JSON data and generate embeddings using BGE-M3.

### Step 5 — Process the User Query

Run:

```text
process_incoming.py
```

to process the incoming query, retrieve relevant information, and generate the final response using the RAG pipeline.

---



## Future Improvements

Possible future improvements include:

* Integration with a dedicated vector database such as FAISS or Chroma.
* Improved chunking strategies based on semantic boundaries.
* Retrieval evaluation using metrics such as Recall@K and MRR.
* Reranking retrieved chunks before sending them to the LLM.
* Streaming responses for improved user experience.
* Building a web interface for students.
* Adding conversation memory for multi-turn questions.

---

## Conclusion

This project demonstrates an end-to-end **Retrieval-Augmented Generation pipeline** for building an AI Teaching Assistant from educational video content.

The project combines **speech-to-text, text processing, semantic embeddings, information retrieval, and LLM-based generation**, while also addressing practical performance considerations through **chunk optimization and LLM response-latency optimization**.
