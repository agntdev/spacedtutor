# SpacedTutor — Bot specification

**Archetype:** education

**Voice:** professional and encouraging — write every user-facing message, button label, error, and empty state in this voice.

SpacedTutor is a private, per-user Telegram bot for learning vocabulary using spaced repetition (SRS). Users can create custom cards, use starter decks, and track progress through daily reviews with custom ratings. The bot schedules reviews based on memory retention and sends reminders while maintaining privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individuals learning foreign languages who want a simple, mobile-first flashcard SRS inside Telegram.

## Success criteria

- Users complete daily review sessions with consistent progress tracking
- Users maintain active engagement through spaced repetition scheduling
- Users can create and manage custom decks and cards without external tools

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/study** (command, actor: user, command: /study) — Start a study session with due reviews and new cards
- **Add Card** (button, actor: user, callback: add_card:start) — Begin adding a new card via guided prompts
- **Starter Decks** (button, actor: user, callback: starter_decks:list) — View and copy free starter decks
- **Deck Management** (button, actor: user, callback: deck_management:list) — List, view, edit, or delete decks and cards
- **Settings** (button, actor: user, callback: settings:main) — Configure daily new-card limit, notifications, and premium features

## Flows

### onboarding
_Trigger:_ /start

1. Greet user and offer starter decks or new deck creation
2. Prompt for daily new-card limit (default 10)

_Data touched:_ User, Deck

### add_card
_Trigger:_ add_card:start

1. Prompt for word
2. Prompt for translation
3. Prompt for optional example sentence
4. Assign to a deck

_Data touched:_ Card, Deck

### study_session
_Trigger:_ /study

1. Build session queue of due reviews and new cards
2. Display card front (word)
3. User reveals answer (translation)
4. User selects rating («снова», «трудно», «хорошо», «легко»)
5. Update SRS parameters and progress
6. Continue until session complete or user exits

_Data touched:_ Card, Study session, Review log

### deck_management
_Trigger:_ deck_management:list

1. List all decks
2. View cards in selected deck
3. Edit card (front/back/example)
4. Delete card or deck
5. Copy starter deck

_Data touched:_ Deck, Card

### notifications
_Trigger:_ scheduled

1. Send daily summary/reminder at user-set time
2. Send immediate push when new reviews become due

_Data touched:_ User, Study session

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Private account tied to Telegram user id
  - fields: Telegram user id, daily new-card limit, notification settings, premium status
- **Deck** _(retention: persistent)_ — Collection of cards with title, optional description, and privacy settings
  - fields: title, description, starter vs user-created, privacy
- **Card** _(retention: persistent)_ — Flashcard with word, translation, example, and SRS metadata
  - fields: front (word), back (translation), example sentence, creation date, ease, interval, review history, next_due
- **Study session** _(retention: session)_ — Current session queue and progress tracking
  - fields: queue, position in queue, streak, new cards shown today
- **Review log** _(retention: persistent)_ — History of all card reviews with user ratings
  - fields: card id, timestamp, user rating, resulting interval

## Integrations

- **Telegram** (required) — Bot API messaging for all interactions and scheduled reminders
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure daily new-card limit
- Set notification preferences
- Manage decks and cards
- Upgrade to premium features

## Notifications

- Daily summary/reminder at user-set time
- Immediate push when new reviews become due (configurable)

## Permissions & privacy

- All data is private to the user and not shared with others
- User can delete their data at any time
- No third-party data sharing

## Edge cases

- User stops mid-session - bot saves position and allows resuming
- Empty decks or completed sessions - bot shows friendly messages and suggestions
- User exceeds daily new-card limit - bot prevents adding more cards until next day

## Required tests

- Verify spaced repetition algorithm updates intervals correctly based on user ratings
- Test session continuity when user stops mid-session and resumes
- Validate deck management operations (create, edit, delete, copy) work correctly
- Ensure notifications are sent at correct times based on user settings

## Assumptions

- Default daily new-card limit is 10
- Starter decks include 'Basic 500', 'Travel Phrases', 'Work & Office', 'Food & Restaurant', 'Phrases for Beginners'
- Notification default is off; users opt in and set time or immediate summary
- SRS algorithm uses ease-factor + interval approach with four rating outcomes
- Cards and progress are private per Telegram user
- Import format is simple CSV or newline-delimited 'word - translation - optional example' lines
- Session queue prioritizes due reviews before new cards
