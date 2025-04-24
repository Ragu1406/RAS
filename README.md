from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import httpx

app = FastAPI()

# User input schema
class AgentRequest(BaseModel):
    provider: str  # "vapi" or "retell"
    name: str
    model: str
    instructions: str
    voice: str
    language: str = None  # optional for vapi
    phone_number: str = None  # optional for vapi

@app.post("/create-agent")
async def create_agent(request: AgentRequest):
    if request.provider.lower() == "vapi":
        payload = {
            "name": request.name,
            "model": request.model,
            "voice_id": request.voice,
            "instructions": request.instructions,
            "first_message_prompt": "Hello, how can I help you?"
        }
        url = "https://api.vapi.ai/assistants"  # or whatever the exact URL is
        headers = {"Authorization": "Bearer <YOUR_VAPI_API_KEY>"}

    elif request.provider.lower() == "retell":
        payload = {
            # "name": request.name,
            "model": request.model,
            "voice": request.voice,
            "instructions": request.instructions,
            "language": request.language,
            "phone_number": request.phone_number
        }
        url = "https://api.retellai.com/agents"  # replace with exact URL
        headers = {"Authorization": "Bearer <YOUR_RETELL_API_KEY>"}

    else:
        raise HTTPException(status_code=400, detail="Invalid provider. Use 'vapi' or 'retell'.")

    # Send request to the selected API
    async with httpx.AsyncClient() as client:
        response = await client.post(url, json=payload, headers=headers)

    if response.status_code != 200:
        raise HTTPException(status_code=response.status_code, detail=response.text)

    return response.json()
