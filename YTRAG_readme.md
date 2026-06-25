YouTube RAG Q&A System (n8n)

This repository contains two interconnected n8n workflows that together implement a YouTube Retrieval-Augmented Generation (RAG) system:

YT RAG – Ingests YouTube videos, fetches transcripts, summarizes content, chunks and embeds data, and stores it in Pinecone.

Q/A YT Copy – Exposes a webhook that answers user questions by searching the vector database of YouTube transcripts.

Architecture Overview
Google Sheets (Video URLs)
        ↓
YT RAG Workflow
        ↓
YouTube API + Supadata
        ↓
Transcript Chunking + Embeddings
        ↓
Pinecone Vector Store
        ↓
Q/A YT Copy Webhook
        ↓
User Q&A Responses

Workflow 1: YT RAG (Video Ingestion & Indexing)
Purpose

Automatically processes YouTube videos and prepares them for semantic search and Q&A.

Trigger

Google Sheets Trigger

Polls a spreadsheet every minute

Processes rows with status = pending or empty

Key Steps

Extract Video URL

Parses YouTube video ID

Tracks processing start time

Fetch YouTube Metadata

Title, author, duration, category

Thumbnail, views, likes, comments

Update Google Sheet (Processing State)

Marks row as processing

Saves metadata

Fetch Transcript

Uses Supadata API

Language: English

Validate Transcript

Ensures transcript exists and is usable

Calculates word and character counts

Summarize Video

Uses Google Gemini

Produces:

Overview

Key Points

Takeaways

Chunk Transcript

~1000 characters per chunk

Sentence-aware splitting

Metadata attached to each chunk:

Video ID

Title

Author

Category

Estimated timestamps

Embed & Store

Embeddings via Ollama (nomic-embed-text)

Stored in Pinecone

Finalize Processing

Updates Google Sheet:

status = completed

chunk count

processing time

summary

Output

Fully indexed YouTube video transcripts

Searchable semantic vectors in Pinecone

Rich metadata and summaries in Google Sheets

Workflow 2: Q/A YT (Question Answering API)
Purpose

Answer user questions based on indexed YouTube video content.

Trigger

Webhook (POST)

Expected Input (JSON)
{
  "question": "What are the key takeaways from the video about AI agents?"
}

Key Steps

Webhook Input Validation

Extracts question

Rejects short or empty questions

AI Agent (LangChain)

Uses vector search to find relevant transcript chunks

Enforced behavior:

Always search the vector store

Cite sources internally

Provide concise, conversational answers

Vector Search

Pinecone vector store

Embeddings: nomic-embed-text

Answer Formatting

Cleans and normalizes AI output

Returns structured JSON response

Output (JSON)
{
  "success": true,
  "answer": "The video explains how AI agents operate by combining tools, memory, and planning...",
  "timestamp": "2026-01-08T12:34:56.789Z"
}

Technology Stack

Automation: n8n

LLMs:

Google Gemini (summarization & reasoning)

Ollama (local embeddings)

Vector Database: Pinecone

Transcript API: Supadata

Data Store: Google Sheets

Embedding Model: nomic-embed-text

Setup Requirements
Accounts & Credentials

You will need:

Google Sheets OAuth

YouTube Data API

Pinecone API Key

Supadata API Key

Ollama running locally (with nomic-embed-text)

Google Gemini (PaLM) API

Google Sheet Columns (Required)

Row ID

video_url

status

video_title

author

category

duration

summary

chunks_count

indexed_at

processing_s

How the Two Workflows Connect
Workflow	Role
YT RAG	Builds the knowledge base
Q/A YT Copy	Queries the knowledge base

Important:
The Q/A workflow depends entirely on the Pinecone index created by the YT RAG workflow.

Common Use Cases

Chat with your YouTube content

Internal knowledge base from video libraries

Creator tools for audience Q&A

AI search across long-form video content

Notes & Best Practices

Avoid re-processing videos already marked as completed

Monitor Pinecone index size for large libraries

Keep transcript chunk size consistent with embedding model limits

Use webhook authentication if exposing publicly

License

This workflow configuration is provided as-is.
You are responsible for API usage, data handling, and compliance with YouTube’s terms of service