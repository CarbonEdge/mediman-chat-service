# Chat Service

AI chat microservice handling conversations and RunPod integration.

## Responsibility

- **Chat API**: RESTful endpoints for chat functionality
- **RunPod Integration**: Connect to RunPod GPU endpoints
- **Conversation Management**: Store and retrieve chat history
- **Message Processing**: Handle user messages and AI responses

## Technology Options

- **Option 1**: FastAPI (Python) - Recommended for RunPod integration
- **Option 2**: Django REST Framework
- **Option 3**: Go with Gin framework

## Port

**8001**

## API Endpoints

### POST /api/chat/send
Send a message to the AI

**Request:**
```json
{
  "message": "How do I organize patient files?",
  "conversation_id": "optional-uuid"
}
```

**Response:**
```json
{
  "conversation_id": "uuid",
  "message": "I can help you organize patient files...",
  "timestamp": "2025-12-27T20:00:00Z"
}
```

### GET /api/chat/conversations
List user's conversations

**Response:**
```json
{
  "conversations": [
    {
      "id": "uuid",
      "title": "Patient File Organization",
      "last_message": "I can help...",
      "updated_at": "2025-12-27T20:00:00Z"
    }
  ]
}
```

### GET /api/chat/conversations/{id}
Get conversation details

**Response:**
```json
{
  "id": "uuid",
  "messages": [
    {
      "role": "user",
      "content": "How do I organize files?",
      "timestamp": "2025-12-27T19:59:00Z"
    },
    {
      "role": "assistant",
      "content": "I can help...",
      "timestamp": "2025-12-27T20:00:00Z"
    }
  ]
}
```

## Database Schema

### Conversation
- id (UUID)
- user_id (foreign key)
- title (string)
- created_at (timestamp)
- updated_at (timestamp)

### Message
- id (UUID)
- conversation_id (foreign key)
- role (user/assistant)
- content (text)
- created_at (timestamp)

## Environment Variables

```env
PORT=8001
DATABASE_URL=postgresql://user:pass@localhost:5432/chat_db

# RunPod
RUNPOD_API_KEY=your_api_key
RUNPOD_ENDPOINT_ID=your_endpoint_id

# Authentication
JWT_SECRET=your_jwt_secret
```

## Implementation (FastAPI Example)

```python
# app/main.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import requests

app = FastAPI()

class ChatRequest(BaseModel):
    message: str
    conversation_id: str | None = None

@app.post("/api/chat/send")
async def send_message(request: ChatRequest):
    # Call RunPod
    response = requests.post(
        f"https://api.runpod.io/v2/{RUNPOD_ENDPOINT_ID}/run",
        json={"input": {"prompt": request.message}},
        headers={"Authorization": f"Bearer {RUNPOD_API_KEY}"}
    )

    # Save to database
    # ... (implementation)

    return {
        "conversation_id": "uuid",
        "message": response.json()["output"],
        "timestamp": "2025-12-27T20:00:00Z"
    }
```

## Running

```bash
# Install dependencies
pip install -r requirements.txt

# Run with uvicorn
uvicorn app.main:app --host 0.0.0.0 --port 8001 --reload
```

## Docker

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8001"]
```

## TODO

- [ ] Implement FastAPI app structure
- [ ] Add RunPod service integration
- [ ] Create database models
- [ ] Add authentication middleware
- [ ] Write unit tests
- [ ] Add API documentation
