---
name: messaging
description: Send LinkedIn direct messages via connected accounts.
---

# LinkedIn Messaging

Send LinkedIn direct messages through connected accounts. Use this for outreach, follow-ups, and prospect engagement.

## Tools Used

- `send_linkedin_dm` (5 tokens): send a direct message. Params: `senderUsername` (connected account), `recipientUsername` (recipient), `message` (1-8000 chars).

See `references/tools-reference.md` for exact commands.

## Workflow

1. **Verify sender account is connected**
   - Use `list_connected_linkedin_accounts` to confirm the sender account is available
   - The sender must be a connected LinkedIn account

2. **Compose the message**
   - Draft a personalized, relevant message (1-8000 characters)
   - Keep it concise and professional

3. **Confirm with the user before sending**
   - Always show the full message content and recipient to the user
   - Wait for explicit approval before calling `send_linkedin_dm`

4. **Send the DM**
   - Call `send_linkedin_dm` with senderUsername, recipientUsername, and message
   - Confirm delivery to the user

## Safety Notes

- Always confirm message content with the user before sending
- Respect LinkedIn rate limits
- Keep messages personal and relevant
- The sender must be a connected LinkedIn account

## Use Cases

- "Send a LinkedIn message to johndoe from my account"
- "Follow up with this prospect on LinkedIn"
- "Message the VP of Sales at acme from my connected account"
