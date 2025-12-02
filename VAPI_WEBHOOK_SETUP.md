# Vapi Webhook Configuration for Memory Persistence

This document explains how to configure Vapi webhooks to ensure call transcripts from phone calls are saved to your database, enabling memory persistence across both web and phone interactions.

## Overview

When a user assigns a phone number to their voice agent in Vapi and receives calls, you need to configure webhooks so that call events trigger your `save-call-transcript` edge function.

## Webhook Setup in Vapi Dashboard

1. **Navigate to your Vapi Dashboard**
   - Go to your agent's configuration page
   - Find the "Webhooks" or "Server URL" section

2. **Configure the Server URL**
   ```
   https://ugwlhipkdpqcakxxasex.supabase.co/functions/v1/save-call-transcript
   ```

3. **Add Authorization Header**
   - In the webhook headers configuration, add:
     ```
     Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InVnd2xoaXBrZHBxY2FreHhhc2V4Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjI1MTY2MjgsImV4cCI6MjA3ODA5MjYyOH0.Uy-FblSCQaQZY7qKUa1PSKXr4XBzu15dG1ZhARBcL5U
     ```

4. **Configure Call End Event**
   - Enable the "call.ended" or "endOfCallReport" event
   - This ensures the webhook fires when a call completes

## Expected Webhook Payload

Your `save-call-transcript` function expects the following payload from Vapi:

```json
{
  "vapi_agent_id": "your-vapi-agent-id",
  "session_id": "unique-call-session-id",
  "transcript": [
    {
      "role": "assistant",
      "content": "Hello! How can I help you today?"
    },
    {
      "role": "user",
      "content": "I'd like to know about upcoming events."
    }
  ]
}
```

## How Memory Persistence Works

1. **Phone Call Ends**
   - Vapi sends webhook to `save-call-transcript`
   - Transcript and AI-generated summary are saved to `agent_call_transcripts` table

2. **Next Call (Web or Phone)**
   - When user initiates a new call, `load-agent-memory` retrieves the latest transcript
   - AI rephrases the summary into a natural conversational opener
   - The opener is injected as the first system message in the new call

3. **Cross-Platform Consistency**
   - Memory persists whether user calls via web interface or phone number
   - All transcripts are linked by `vapi_agent_id`

## Testing the Integration

1. **Make a test phone call** to your Vapi agent's assigned number
2. **Check Supabase logs** for `save-call-transcript`:
   ```
   supabase functions logs save-call-transcript --tail
   ```
3. **Verify database entry**:
   ```sql
   SELECT * FROM agent_call_transcripts 
   ORDER BY created_at DESC 
   LIMIT 1;
   ```
4. **Make a follow-up call** and verify memory is loaded

## Troubleshooting

- **Webhook not firing**: Check Vapi dashboard webhook configuration
- **Authorization errors**: Verify the Bearer token matches your project
- **Missing transcripts**: Ensure Vapi is sending the correct payload format
- **Memory not loading**: Check `load-agent-memory` function logs

## Security Notes

- The webhook uses JWT authentication via the `Authorization` header
- The `save-call-transcript` function requires JWT verification (see `supabase/config.toml`)
- User ID is extracted from the JWT token to ensure proper data isolation
