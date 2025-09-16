# Opal AI  

## 📦 1. Installation  

### 🚀 Getting Started  
This guide will walk you through setting up the **Opal AI** project on your local machine for development and testing.  

---

### ✅ Prerequisites  
Before you begin, ensure you have the following installed on your system:  

- **[Bun](https://bun.sh/):** This project uses Bun for package management and as its runtime. It's a fast all-in-one JavaScript toolkit.  
- **[Git](https://git-scm.com/):** For cloning the repository.  
- **A Database Server:** The project is configured to use **PostgreSQL**, but you can adapt it to another database supported by Prisma (e.g., MySQL, SQLite).  

---

## 🔹 A. Clone the Repository  

First, clone the project repository from GitHub to your local machine using the following command in your terminal:  

```bash
git clone https://github.com/rizzit11/OpalAI.git
```
Once the cloning is complete, navigate into the project directory:
```bash
bash : cd OpalAI
```
## 🔹 B. Project Setup
Follow these steps to install dependencies and configure the necessary environment variables.

1. **[Install Dependencies]**
Instead of npm or yarn, this project uses bun. Run the following command to install all the required packages listed in package.json:
```bash
bash : bun install
```
2. **[Configure Environment Variables]**

- The project requires several environment variables to connect to your database and external services (like Clerk, AWS, and AI APIs).

- Create a new file in the root of the project directory named .env.local.

- Copy the contents of the template below into your new .env.local file.

- Replace the placeholder values (e.g., "your_database_url_here") with your actual credentials and API keys.

3. **[📄 .env.local Template:]**
```bash
# Prisma Database URL
# Example for PostgreSQL: postgresql://USER:PASSWORD@HOST:PORT/DATABASE
DATABASE_URL="your_database_url_here"

# Clerk Authentication (Get these from your Clerk Dashboard)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="your_clerk_publishable_key"
CLERK_SECRET_KEY="your_clerk_secret_key"
NEXT_PUBLIC_CLERK_SIGN_IN_URL="/sign-in"
NEXT_PUBLIC_CLERK_SIGN_UP_URL="/sign-up"
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL="/dashboard"
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL="/dashboard"

# AWS Credentials (For S3 and CloudFront)
AWS_ACCESS_KEY_ID="your_aws_access_key"
AWS_SECRET_ACCESS_KEY="your_aws_secret_key"
AWS_REGION="your_aws_region"
S3_BUCKET_NAME="your_s3_bucket_name"

# AI Service API Keys
OPENAI_API_KEY="your_openai_api_key"
# Add other keys if needed for Whisper AI or VoiceFlow
VOICEFLOW_API_KEY="your_voiceflow_api_key"

# Application URL
NEXT_PUBLIC_URL="http://localhost:3000"
```

**[3. Set Up the Database]**
This project uses Prisma to manage the database schema. After configuring your DATABASE_URL in the .env.local file, run the following command to apply the schema to your database. This will create all the necessary tables.

bash : bunx prisma migrate dev
You may be prompted to give the migration a name; you can enter something like init.

## 🔹 C. Running the Application
Once you have completed the setup, you can start the development server. Run the following command in your terminal:
```bash
**[bash : bun run dev]**
```
This will start the Next.js application in development mode.
You can now open your web browser and navigate to:

**[👉 http://localhost:3000]**


---


## 📖 2. About the Project


Opal AI is a powerful platform designed to make screen recordings intelligent and interactive. At its core, it's an ecosystem that transforms standard video content into a searchable and insightful knowledge base.

The project is split into two main components for the user:

#####1. The Web App (Next.js):

- Your central dashboard.

- Manage your account, browse your library of recordings, and interact with AI-generated insights.

##### 2. The Desktop App (Electron.js):

- The tool for recording.

- Captures your entire screen (not just a browser tab) for high-quality recordings.

### 🔑 How it Works
The process works by streaming your recording in real-time to a dedicated Express.js server. Data is saved continuously as you record. Once finished, the AI pipeline automatically:

- Transcribes (Whisper AI)
- Summarizes (OpenAI)
- Analyzes (VoiceFlow)

```bash
Final output → An interactive knowledge base.
```
### ✨ Key Features
- 🎥 Native Screen Recording: High-fidelity screen + audio capture using Electron.js.

- ⚡ Real-Time Streaming: Video streamed & saved as you record, minimizing data loss.

- 🔐 Secure Authentication: User authentication via Clerk.

- 📝 AI-Powered Transcription: Whisper AI converts video-to-text automatically.

- 🧠 Intelligent Content Generation: OpenAI generates titles & summaries.

- 🤖 Interactive Knowledge Base: Video transcripts become a Q&A agent using VoiceFlow.

- ☁️ Scalable Cloud Infrastructure: Secure storage with AWS S3 + fast streaming with AWS CloudFront.

- 💻 Dynamic User Experience: Sleek, modern frontend built with Next.js.

### 🛠️ Tech Stack
- Frontend: Next.js

- Desktop App: Electron.js

- Real-time Backend: Express.js, Socket.io

- Authentication: Clerk

- Cloud Storage: AWS S3

- CDN: AWS CloudFront

- AI Transcription: Whisper AI

- AI Content Generation: OpenAI

- Knowledge Base / AI Agent: VoiceFlow

- Database: PostgreSQL with Prisma

---


# 3. Architecture
The architecture of Opal AI is designed for robustness, scalability, and real-time performance. It separates concerns between the client applications, the real-time processing server, and the cloud infrastructure.

## Architectural Breakdown
The system operates through a series of coordinated workflows, which are visualized and explained below.

### Stage 1: User Authentication
The user signs up or logs in through the Next.js or Electron application. Clerk handles the core authentication process. Upon success, a callback updates the user's profile in our database. The Electron app then syncs with this profile to get user-specific settings.

```bash
[ Web App / Desktop App ]
       |
       | 1. Login/Signup Request
       v
  [ Clerk Auth ]
       |
       | 2. Auth Webhook (user data)
       v
[ Next.js Backend API ]
       |
       | 3. Create/Update User
       v
[ Database ]
```

### Stage 2: Real-Time Screen Recording & Ingestion
The user starts a recording in the Electron app. The app captures the screen and sends video "chunks" every second over a WebSocket to our dedicated Express.js server, which writes them to a temporary file.

```bash
[ Electron Desktop App ]
       |
       | 5. Stream Video Chunks (WebSocket)
       v
[ Express.js Server ]
       |
       | 6. Write chunks to a temporary file
       v
[ Temp File System ]
```

### Stage 3: Video Processing, Storage & AI Enrichment
When the recording stops, the Express server tells the Next.js backend to create a "processing" entry in the database. The full video is then uploaded to a private AWS S3 bucket. Once the upload is complete, the AI pipeline kicks off: the video is transcribed by Whisper AI, summarized by OpenAI, and indexed by VoiceFlow. Finally, the database entry is updated to "completed" with all the new AI data, and the temporary file is deleted.

```bash
[ Express.js Server ]
       |
       | 7. API Call to Next.js -> [ Create "Processing" DB Entry ]
       |
       | 8. Upload complete video file
       v
[ AWS S3 Bucket ]
       |
       | 9. Upload is complete, start AI pipeline
       v
[ Express.js Server ]
       |
       |--> 10. Send to [ Whisper AI ] -> (Get Transcript)
       |
       |--> 11. Send Transcript to [ OpenAI ] -> (Get Title/Summary)
       |
       |--> 12. Send Transcript to [ VoiceFlow ] -> (Update Knowledge Base)
       |
       | 13. API Call to Next.js -> [ Update DB Entry to "Completed" with AI data ]
       |
       | 14. Delete temp file from [ Temp File System ]
       v
   (Process Complete)
```

### Stage 4: Video Playback
When a user on the Next.js frontend clicks to watch a video, the request is routed through AWS CloudFront. To maintain security, CloudFront is the only service with permission to fetch the video from our private S3 bucket. It then securely streams the content with low latency to the user's browser.

```bash
[ Next.js Web App ]
       |
       | 15. User requests to play a video
       v
[ AWS CloudFront (CDN) ]
       |
       | 16. Securely fetches the video from the private bucket
       v
[ AWS S3 Bucket ]
       |
       | 17. Serves the video file to CloudFront
       v
[ AWS CloudFront (CDN) ]
       |
       | 18. Streams the video with low latency back to the user
       v
[ Next.js Web App ]
```

