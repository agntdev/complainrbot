# Account Takedown Assistant — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that helps users submit structured account takedown complaints to external services using preconfigured templates, linked outbound accounts, and private history tracking. Users manage credentials, customize complaint content, and confirm each submission before execution.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individuals
- small groups
- non-technical users

## Success criteria

- User successfully submits 1+ verified complaints to target services
- User manages at least one linked outbound account
- User views and confirms complaint history

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding flow
- **Link account** (button, actor: user, callback: account:link) — Add new outbound account credentials
  - inputs: email/username, password
  - outputs: linked account confirmation
- **New complaint** (button, actor: user, callback: complaint:new) — Start complaint creation flow
  - inputs: template selection, custom fields
  - outputs: draft preview
- **History** (button, actor: user, callback: history:view) — View private complaint history
  - inputs: filter parameters
  - outputs: sent complaint records

## Flows

### Onboarding
_Trigger:_ /start

1. Display terms and explanation
2. Request user registration
3. Store user profile

_Data touched:_ User

### Complaint submission
_Trigger:_ complaint:new

1. Select template
2. Edit fields
3. Preview draft
4. Confirm approval
5. Send via linked account
6. Report result

_Data touched:_ ComplaintDraft, SentComplaint

### Account management
_Trigger:_ account:link

1. Collect credentials
2. Validate login
3. Store encrypted account

_Data touched:_ LinkedAccount

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile with preferences
  - fields: telegram_id, preferred_language, notification_prefs
- **LinkedAccount** _(retention: persistent)_ — Outbound account credentials for sending complaints
  - fields: email/username, encrypted_password, service_type
- **ComplaintTemplate** _(retention: persistent)_ — Predefined violation checklists
  - fields: category, fields_schema, target_service
- **ComplaintDraft** _(retention: session)_ — Unsent complaint with editable content
  - fields: template_id, custom_fields, target_address
- **SentComplaint** _(retention: persistent)_ — Confirmed complaint record with delivery status
  - fields: timestamp, content_hash, delivery_status, target_service

## Integrations

- **Telegram** (required) — Bot API messaging
- **Email API** (required) — Send complaints via abuse@ addresses
- **Web Forms API** (required) — Submit complaints to target websites
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Manage complaint templates
- Configure outbound account linking rules
- Set template categories

## Notifications

- Step confirmation messages (approval/edit/cancel)
- Delivery success/failure reports
- History updates with timestamps

## Permissions & privacy

- All data is user-isolated
- No shared admin access to complaint content
- Credentials stored encrypted

## Edge cases

- Failed email delivery with retry options
- Invalid credentials during account linking
- User cancellation at any step
- Template field validation errors

## Required tests

- End-to-end complaint submission flow
- Credential encryption/decryption validation
- Private history isolation between users

## Assumptions

- Users always confirm before sending
- Target services accept email/web form submissions
- Templates are preconfigured by owner
