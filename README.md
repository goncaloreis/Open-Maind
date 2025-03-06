# M𝛂ind: Functional and Technical Specifications

This document provides a complete and prescriptive specification for **M𝛂ind**, a private, AI-powered journaling application designed to transform casual conversations into structured, immutable journal entries. It leverages large language models (LLMs) and AI-driven analysis to process mixed inputs (text, links, images, voice, files) and create a visually engaging personal timeline with AI-generated artwork. The app prioritizes privacy, usability, and scalability.

---

## Table of Contents
1. [Introduction](#1-introduction)
2. [Functional Specifications](#2-functional-specifications)
   - [User Authentication](#21-user-authentication)
   - [Chat Interface](#22-chat-interface)
   - [M𝛂ind-it Action](#23-m𝛂ind-it-action)
   - [Timeline Interface](#24-timeline-interface)
   - [Post Interaction](#25-post-interaction)
   - [AI Capabilities](#26-ai-capabilities)
   - [Search and Filtering](#27-search-and-filtering)
   - [Settings and Preferences](#28-settings-and-preferences)
3. [Technical Specifications](#3-technical-specifications)
   - [Backend](#31-backend)
   - [Frontend](#32-frontend)
   - [Database](#33-database)
   - [AI Integration](#34-ai-integration)
   - [Media Handling](#35-media-handling)
   - [Security](#36-security)
4. [User Flow](#4-user-flow)
5. [AI Processing](#5-ai-processing)
6. [Performance and Scalability](#6-performance-and-scalability)
7. [Testing](#7-testing)
8. [Deployment and Maintenance](#8-deployment-and-maintenance)
9. [Conclusion](#9-conclusion)

---

## 1. Introduction

**M𝛂ind** is an innovative journaling app that combines conversational AI with structured reflection. Users interact with the app through a chat interface, inputting thoughts via text, links, images, voice recordings, or files. An AI processes these inputs to generate draft journal entries, which users can review, edit, and confirm. Once confirmed, entries are added to a personal timeline with AI-generated artwork and become immutable. The app ensures all data remains private and secure.

### Purpose
The purpose of this document is to provide a detailed blueprint for developing M𝛂ind using Cursor. Every requirement is explicitly defined to eliminate ambiguity, ensuring the app is built exactly as envisioned.

### Key Objectives
- Deliver a conversational, LLM-powered chat interface for effortless journaling.
- Use AI to analyze chat inputs and generate structured journal entries.
- Create a visually appealing timeline with AI-generated images.
- Guarantee user privacy through robust security measures.

### Target Audience
- Individuals aged 18–45 who value self-reflection and modern technology.
- Users seeking a low-effort, conversational journaling experience.

---

## 2. Functional Specifications

This section outlines the app's features with precise, actionable requirements.

### 2.1 User Authentication

#### Requirements
- **Sign-Up**:
  - Users create an account using an email address and password.
  - Email must be unique and validated via a regex: `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`.
  - Password must be 8–32 characters, including:
    - At least 1 uppercase letter (`A-Z`).
    - At least 1 lowercase letter (`a-z`).
    - At least 1 number (`0-9`).
    - At least 1 special character (`!@#$%^&*()`).
  - After submission, send a verification email with a unique 6-digit code (e.g., `123456`).
  - Users must enter the code within 15 minutes to activate the account; otherwise, the code expires.
- **Login**:
  - Users enter their email and password in a form.
  - Maximum 5 failed login attempts allowed within 10 minutes before a 30-minute lockout.
  - Successful login redirects to the chat interface.
- **Password Reset**:
  - Users click "Forgot Password" on the login page, enter their email, and receive a reset link valid for 1 hour.
  - The link directs to a page where users enter a new password meeting the sign-up criteria.
- **Profile Management**:
  - Users can update their display name (1–50 characters, alphanumeric and spaces only).
  - Users can upload an avatar (JPEG/PNG, max 2MB, resized to 100x100px on the server).

#### UI Elements
- Sign-Up Form: Email field, password field, "Sign Up" button.
- Login Form: Email field, password field, "Login" button, "Forgot Password" link.
- Verification Page: 6-digit code input, "Verify" button.
- Profile Page: Name input, avatar upload button, "Save" button.

---

### 2.2 Chat Interface

#### Requirements
- **Access**: Displays immediately after login as the default screen.
- **Input Types**:
  - **Text**: Users type messages in a text box (max 10,000 characters per message).
  - **Links**: URLs (e.g., `https://example.com`) are detected via regex (`^https?://[^\s]+$`), rendered as clickable hyperlinks in the chat.
  - **Images**: Users click an "Upload Image" button, selecting JPEG/PNG files (max 10MB each, 5 images per message).
  - **Voice**: Users click a "Record" button, recording up to 5 minutes of audio (saved as MP3, max 10MB). Transcription uses an AI service (e.g., Whisper) and displays as text.
  - **Files**: Users click an "Upload File" button, selecting any file type (max 50MB each, 3 files per message).
- **Chat Display**:
  - Messages appear in a scrollable container, user messages on the right (blue background), AI responses on the left (gray background).
  - Each message shows a timestamp (e.g., `2023-10-15 14:30:45`).
  - Uploaded media/files display inline with thumbnails (100x100px) and download links.
- **LLM Responses**:
  - Powered by an LLM (e.g., GPT-4), responding within 3 seconds under normal load.
  - Responses are conversational, limited to 2,000 characters unless summarizing inputs.
  - Retains context of all messages in the current session (resets on logout or "M𝛂ind-it").
- **Buttons**:
  - "Send" button submits the current message.
  - "M𝛂ind-it" button triggers journal entry creation (see [2.3](#23-m𝛂ind-it-action)).

#### UI Elements
- Chat Container: Scrollable div, 80% of screen height, messages in chronological order.
- Input Bar: Text box, "Upload Image" button, "Record" button, "Upload File" button, "Send" button.
- Top Bar: "M𝛂ind-it" button, "Logout" button.

---

### 2.3 M𝛂ind-it Action

#### Requirements
- **Initiation**: Triggered by clicking the "M𝛂ind-it" button or typing `/mindit` in the chat.
- **Draft Generation**:
  - The Drafting AI processes the entire chat session (all messages since login).
  - Generates a draft with:
    - **Title**: 5–50 characters, summarizing the chat (e.g., "Reflections on Today").
    - **Content**: 50–5,000 characters, a cohesive narrative based on chat inputs.
  - Draft is displayed within 10 seconds under normal load.
- **Review and Edit**:
  - Draft appears in a modal with editable title and content fields.
  - Users can modify text; changes are saved locally until confirmation.
  - "Preview" button shows the draft with an AI-generated image (see [2.6](#26-ai-capabilities)).
- **Confirmation**:
  - "Confirm" button finalizes the post, adding it to the timeline.
  - Post becomes immutable after confirmation; no further edits are possible.
- **Reset**: After confirmation, the chat session clears, starting a new session.

#### UI Elements
- Modal: Title input, content textarea, "Preview" button, "Confirm" button, "Cancel" button.
- Loading Spinner: Displays during draft generation (max 10 seconds).

---

### 2.4 Timeline Interface

#### Requirements
- **Access**: Accessible via a "Timeline" tab from the chat interface.
- **Display**:
  - Posts listed in reverse chronological order in a scrollable container.
  - Each post includes:
    - **AI-Generated Image**: 300x200px, displayed above the title.
    - **Title**: Bold, 16px font, max 50 characters.
    - **Content**: 14px font, max 5,000 characters, truncated to 200 characters with "Read More" link.
    - **Timestamp**: `YYYY-MM-DD HH:MM:SS` format (e.g., `2023-10-15 14:30:45`).
- **Navigation**:
  - Infinite scrolling loads 10 posts initially, 5 more per scroll event (triggered at 80% scroll depth).
  - "Back to Top" button appears after scrolling past 5 posts.

#### UI Elements
- Timeline Container: Scrollable div, 80% of screen height.
- Post Card: Div with image, title, content preview, timestamp, "View Inputs" button (see [2.5](#25-post-interaction)).
- Top Bar: "Chat" tab, "Timeline" tab (active).

---

### 2.5 Post Interaction

#### Requirements
- **View Inputs**:
  - Each post has a "View Inputs" button that opens a modal.
  - Modal displays all original chat messages, links, images, voice transcripts, and files linked to the post.
  - Media/files are downloadable via clickable links.
- **Immutability**:
  - Confirmed posts cannot be edited or deleted by the user.
  - Only account deletion removes posts (see [2.8](#28-settings-and-preferences)).

#### UI Elements
- Modal: Scrollable div with chat-style message list, "Close" button.

---

### 2.6 AI Capabilities

#### Requirements
- **Chat AI**:
  - Uses GPT-4 via OpenAI API (or equivalent).
  - Responds to all input types, maintaining session context.
- **Drafting AI**:
  - Analyzes chat history using a custom prompt (e.g., "Summarize this conversation into a journal entry with a title and content").
  - Output: JSON with `title` (string) and `content` (string).
- **Image Generation AI**:
  - Uses DALL·E 3 via API (or equivalent).
  - Prompt: "Create an artistic 300x200px image reflecting: [post content]".
  - Image saved as PNG, stored in media storage.

#### Integration
- All AI calls are asynchronous, with loading indicators in the UI.

---

### 2.7 Search and Filtering

#### Requirements
- **Search**:
  - Search bar in the timeline interface accepts keywords (1–100 characters).
  - Matches against post titles and content (case-insensitive).
  - Results display in the timeline format, replacing the default view.
- **Filters**:
  - Date range picker (e.g., `2023-01-01` to `2023-12-31`).
  - Apply button triggers filtering; results update in real-time.

#### UI Elements
- Search Bar: Text input, "Search" button.
- Filter Section: Start date picker, end date picker, "Apply" button.

---

### 2.8 Settings and Preferences

#### Requirements
- **Notifications**:
  - Toggle for email notifications (e.g., weekly summary), default off.
- **Themes**:
  - Dropdown with "Light" (white background, black text) and "Dark" (black background, white text) options, default Light.
- **Account Management**:
  - Update profile (name, avatar) as per [2.1](#21-user-authentication).
  - "Delete Account" button prompts confirmation, then deletes all user data (irreversible).

#### UI Elements
- Settings Page: Notification toggle, theme dropdown, profile form, "Delete Account" button.

---

## 3. Technical Specifications

### 3.1 Backend

#### Requirements
- **Framework**: Node.js v18 with Express v4.18.
- **APIs**:
  - `POST /auth/signup`: `{ email, password }` → `{ user_id, token }`.
  - `POST /auth/login`: `{ email, password }` → `{ token }`.
  - `POST /auth/reset`: `{ email }` → 200 OK.
  - `GET /chat/:user_id`: → `{ messages: [] }`.
  - `POST /chat/:user_id`: `{ type, content }` → 200 OK.
  - `POST /posts`: `{ user_id, title, content, image_url }` → `{ post_id }`.
  - `GET /posts/:user_id`: → `{ posts: [] }`.
- **Authentication**: JWT, 24-hour expiration, stored in HTTP-only cookie.

#### Directory Structure
```
/backend
/controllers
  auth.js
  chat.js
  posts.js
/models
  user.js
  chat.js
  post.js
/routes
  auth.js
  chat.js
  posts.js
/middleware
  auth.js
index.js
```

---

### 3.2 Frontend

#### Requirements
- **Framework**: React v18 with TypeScript.
- **Components**:
  - `Login.tsx`: Login form.
  - `Chat.tsx`: Chat interface.
  - `Timeline.tsx`: Timeline display.
  - `PostModal.tsx`: Draft review and input view.
- **State Management**: Redux Toolkit v1.9, store: `{ user, chat, posts }`.

#### Directory Structure
```
/frontend
/src
  /components
    Login.tsx
    Chat.tsx
    Timeline.tsx
    PostModal.tsx
  /store
    index.ts
    userSlice.ts
    chatSlice.ts
    postsSlice.ts
  App.tsx
  index.tsx
```

---

### 3.3 Database

#### Requirements
- **Type**: PostgreSQL v15.
- **Schema**:
  - `users`:
    - `id`: UUID, primary key.
    - `email`: VARCHAR(255), unique, not null.
    - `password`: VARCHAR(255), hashed, not null.
    - `name`: VARCHAR(50), default null.
    - `avatar`: VARCHAR(255), default null.
  - `chats`:
    - `id`: UUID, primary key.
    - `user_id`: UUID, foreign key → `users(id)`.
    - `messages`: JSONB, not null (e.g., `[{ type, content, timestamp }]`).
    - `timestamp`: TIMESTAMP, not null.
  - `posts`:
    - `id`: UUID, primary key.
    - `user_id`: UUID, foreign key → `users(id)`.
    - `title`: VARCHAR(50), not null.
    - `content`: TEXT, not null.
    - `image_url`: VARCHAR(255), not null.
    - `timestamp`: TIMESTAMP, not null.
  - `inputs`:
    - `id`: UUID, primary key.
    - `post_id`: UUID, foreign key → `posts(id)`.
    - `type`: ENUM(‘text’, ‘link’, ‘image’, ‘voice’, ‘file’), not null.
    - `content`: TEXT, not null.
    - `timestamp`: TIMESTAMP, not null.

#### Indexes
- `users(email)`
- `chats(user_id, timestamp)`
- `posts(user_id, timestamp)`

---

### 3.4 AI Integration

#### Requirements
- **Chat AI**: OpenAI GPT-4 API, endpoint: `POST https://api.openai.com/v1/chat/completions`.
  - Headers: `{ Authorization: "Bearer $API_KEY" }`.
  - Body: `{ model: "gpt-4", messages: [] }`.
- **Drafting AI**: Same endpoint, custom prompt in messages.
- **Image Generation AI**: DALL·E 3 API, endpoint: `POST https://api.openai.com/v1/images/generations`.
  - Body: `{ prompt: "", size: "300x200" }`.

---

### 3.5 Media Handling

#### Requirements
- **Storage**: AWS S3 bucket `m𝛂ind-media`.
- **Upload**: Presigned URLs generated by backend, frontend uploads directly.
- **Retrieval**: URLs stored in `posts.image_url` and `inputs.content`.

---

### 3.6 Security

#### Requirements
- **Passwords**: Hashed with bcrypt (12 rounds).
- **Communication**: HTTPS with TLS 1.3.
- **Data Encryption**: AES-256 for sensitive fields (e.g., `chats.messages`) at rest.

---

## 4. User Flow

1. **Login**: Enter email/password → Chat interface.
2. **Chat**: Send messages with mixed inputs → AI responds.
3. **M𝛂ind-it**: Click button → Review draft → Confirm.
4. **Timeline**: View posts → Click "View Inputs" for details.

---

## 5. AI Processing

- **Chat AI**: Real-time, context-aware responses.
- **Drafting AI**: Analyzes chat, outputs structured JSON.
- **Image Generation AI**: Creates PNG based on post content.

---

## 6. Performance and Scalability

- **Database**: Indexes on `user_id`, `timestamp`.
- **Caching**: Redis for recent posts/chats (TTL 1 hour).
- **Async**: AI tasks use a queue (e.g., BullMQ).

---

## 7. Testing

- **Unit**: Jest for backend, React Testing Library for frontend.
- **Integration**: Test API-to-frontend flows.
- **UAT**: 10 users test full flow.

---

## 8. Deployment and Maintenance

- **Deployment**: AWS ECS, PostgreSQL on RDS, S3 for media.
- **Monitoring**: Datadog for logs and metrics.
- **Updates**: Monthly patches via CI/CD (GitHub Actions).

---

## 9. Conclusion

This specification provides a prescriptive, detailed guide for building M𝛂ind with Cursor. Every requirement is explicitly defined, ensuring a consistent and high-quality implementation.