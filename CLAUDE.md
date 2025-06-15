# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains a Python-based proxy server that translates API requests between the Anthropic format (used by Claude Code) and the Google Gemini format. It uses FastAPI for the web server and LiteLLM to interact with the Gemini API. The main application logic is in `server.py`.

## Common Commands

### Setup

1.  **Create and activate a virtual environment**:
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

2.  **Install dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

### Running the Server

*   **Standard mode**:
    ```bash
    python server.py
    ```

*   **Development mode** (with auto-reload on file changes):
    ```bash
    uvicorn server:app --host 0.0.0.0 --port 8082 --reload
    ```

### Testing

The server includes two primary endpoints for diagnostics:

*   **Health Check**: Check the server's status and configuration.
    ```bash
    curl http://localhost:8082/health
    ```

*   **Connection Test**: Verify connectivity to the Gemini API.
    ```bash
    curl http://localhost:8082/test-connection
    ```

## Architecture

The application is a single-file FastAPI server (`server.py`) with the following key components:

*   **Configuration (`Config` class)**: Manages settings from environment variables (`.env` file), including API keys, model names, and server settings.
*   **Model Management (`ModelManager` class)**: Maps Claude Code model aliases (e.g., `haiku`, `sonnet`) to specific Gemini models defined in the environment variables.
*   **Request/Response Models (Pydantic)**: Defines the data structures for incoming Anthropic-formatted requests and outgoing Gemini-formatted responses (e.g., `MessagesRequest`, `MessagesResponse`).
*   **API Endpoints**:
    *   `/v1/messages`: The main endpoint that receives requests from Claude Code, converts them to the Gemini format, sends them to LiteLLM, and translates the response back to the Anthropic format. It supports both streaming and non-streaming modes.
    *   `/v1/messages/count_tokens`: Estimates token counts for messages.
    *   `/health`, `/test-connection`, `/`: Diagnostic and status endpoints.
*   **Conversion Logic**:
    *   `convert_anthropic_to_litellm`: Translates the incoming Anthropic Messages API format to the format expected by LiteLLM for Gemini. This includes handling system prompts, message content (text and images), and tool usage.
    *   `convert_litellm_to_anthropic`: Translates the Gemini response from LiteLLM back into the Anthropic Messages API format.
*   **Error Handling**: The server includes logic to classify and handle specific Gemini API errors, with retry mechanisms and fallbacks (e.g., from streaming to non-streaming).
*   **Streaming**: The `handle_streaming_with_recovery` function manages Server-Sent Events (SSE) for streaming responses and includes robust recovery for malformed or incomplete data chunks from the Gemini API.