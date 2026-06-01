---
name: messaging
description: Send and read LinkedIn messages via connected accounts.
---

# LinkedIn Messaging

Send and read LinkedIn messages through connected accounts. Use this for outreach, follow-ups, and reading conversation history.

## Tools Used

- `send_linkedin_message` (5 tokens): send a message. Params: `senderUsername` (connected account), `recipientUsername` (recipient), `message` (1-8000 chars).
- `send_linkedin_invitation` (5 tokens): send a connection request with optional message. Params: `senderUsername` (connected account), `recipientUsername` (recipient), `message` (optional, max 300 chars).
- `list_linkedin_conversations` (5 tokens): list the 25 most recent conversations with participant info and up to 100 messages per conversation. Params: `username` (connected account).

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

### Sending a connection invitation

1. **Verify sender account is connected**
   - Use `list_connected_linkedin_accounts` to confirm the sender account is available

2. **Optionally compose a message**
   - A short personal note (max 300 characters) can be included with the invitation
   - If omitted, LinkedIn sends a default invitation

3. **Confirm with the user before sending**
   - Always show the recipient and optional message to the user
   - Wait for explicit approval before calling `send_linkedin_invitation`

4. **Send the invitation**
   - Call `send_linkedin_invitation` with senderUsername, recipientUsername, and optional message
   - Confirm delivery to the user

### Reading conversations

1. **Verify the inbox account is connected**
   - Use `list_connected_linkedin_accounts` to confirm the account

2. **Fetch conversations**
   - Call `list_linkedin_conversations` with the account username
   - Returns 25 most recent conversations with participant info (name, headline, profile ID) and up to 100 messages each

## Safety Notes

- Always confirm message content with the user before sending
- Respect LinkedIn rate limits
- Keep messages personal and relevant
- The sender/inbox owner must be a connected LinkedIn account

## Use Cases

- "Send a LinkedIn message to johndoe from my account"
- "Follow up with this prospect on LinkedIn"
- "Send a connection request to janedoe with a note"
- "What did janedoe message me on LinkedIn?"
- "Show me my conversation with the VP of Sales"
