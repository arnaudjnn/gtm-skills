---
name: messaging
description: Send and read LinkedIn messages via connected accounts.
---

# LinkedIn Messaging

Send and read LinkedIn messages through connected accounts. Use this for outreach, follow-ups, and reading conversation history.

## Tools Used

- `send_linkedin_message` (5 tokens): send a message. Params: `senderUsername` (connected account), `recipientUsername` (recipient), `message` (1-8000 chars).
- `list_linkedin_conversations` (5 tokens): list recent conversations with participant info and full message history. Params: `username` (connected account), `count` (optional, 1-40, default 10).

See `references/tools-reference.md` for exact commands.

## Workflow

### Sending a message

1. **Verify sender account is connected**
   - Use `list_connected_linkedin_accounts` to confirm the sender account is available
   - The sender must be a connected LinkedIn account

2. **Compose the message**
   - Draft a personalized, relevant message (1-8000 characters)
   - Keep it concise and professional

3. **Confirm with the user before sending**
   - Always show the full message content and recipient to the user
   - Wait for explicit approval before calling `send_linkedin_message`

4. **Send the message**
   - Call `send_linkedin_message` with senderUsername, recipientUsername, and message
   - Confirm delivery to the user

### Reading conversations

1. **Verify the inbox account is connected**
   - Use `list_connected_linkedin_accounts` to confirm the account

2. **Fetch conversations**
   - Call `list_linkedin_conversations` with the account username and optional count
   - Returns recent conversations with participant info (name, headline, profile ID) and full message history for each

## Safety Notes

- Always confirm message content with the user before sending
- Respect LinkedIn rate limits
- Keep messages personal and relevant
- The sender/inbox owner must be a connected LinkedIn account

## Use Cases

- "Send a LinkedIn message to johndoe from my account"
- "Follow up with this prospect on LinkedIn"
- "What did janedoe message me on LinkedIn?"
- "Show me my conversation with the VP of Sales"
