# Alex AI - Intelligent Meeting Participant
## Amazon Nova AI Hackathon Project

**Project Category:** Voice AI + Agentic AI  
**Submission Date:** TBD  
**Team Size:** 4 Developers  
**Hashtag:** #AmazonNova

---

## 🎯 Executive Summary

**Alex AI** is a revolutionary voice-powered AI meeting participant that actively contributes to conversations using Amazon Nova 2 Sonic and Nova 2 Lite. Unlike passive note-taking tools, Alex joins meetings as an intelligent participant, responds to questions naturally with human-like speech patterns, and provides real-time insights using speech-to-speech technology.

### Problem Statement
Current AI meeting tools (Otter.ai, Fireflies.ai) are passive observers that only transcribe and summarize. They don't:
- Provide real-time input when asked
- Challenge weak ideas or reinforce strong ones
- Participate in brainstorming
- Offer diverse perspectives during discussions

### Solution
Alex AI actively participates in meetings by:
- Listening to conversations in real-time
- Responding when addressed ("Hey Alex, what do you think?")
- Using natural speech with filler words and pauses
- Offering constructive feedback and alternative perspectives
- Maintaining conversation context throughout the meeting
- Supporting both synchronous contribution and post-meeting summaries

### Market Differentiation
| Feature | Traditional AI Tools | Alex AI |
|---------|---------------------|---------|
| Transcription | ✅ | ✅ |
| Summarization | ✅ | ✅ |
| Real-time responses | ❌ | ✅ |
| Natural conversation | ❌ | ✅ |
| Active participation | ❌ | ✅ |
| Speech-to-speech | ❌ | ✅ |

---

## 🏗️ Technical Architecture

### High-Level System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Frontend Layer (Web App)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Video UI   │  │  Audio Input │  │ AI Controls  │         │
│  │   (WebRTC)   │  │  (WebAudio)  │  │  (React UI)  │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↕ WebSocket (WSS)
┌─────────────────────────────────────────────────────────────────┐
│                    Real-Time Gateway Layer                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │    AWS API Gateway (WebSocket API)                       │  │
│  │    - Connection management                               │  │
│  │    - Audio stream routing                                │  │
│  │    - Message broadcasting                                │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                     Processing Layer (Lambda/ECS)               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Audio       │  │  Context     │  │  Response    │         │
│  │  Processor   │→ │  Manager     │→ │  Generator   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                    Amazon Nova AI Layer                         │
│  ┌─────────────────────┐         ┌─────────────────────┐       │
│  │   Nova 2 Sonic      │         │   Nova 2 Lite       │       │
│  │  Speech-to-Speech   │         │   Reasoning Engine  │       │
│  │  - Voice recognition│         │   - Context analysis│       │
│  │  - Voice synthesis  │         │   - Response logic  │       │
│  │  - Real-time audio  │         │   - Decision making │       │
│  └─────────────────────┘         └─────────────────────┘       │
└─────────────────────────────────────────────────────────────────┾
                              ↕
┌─────────────────────────────────────────────────────────────────┐
│                       Data & Storage Layer                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  DynamoDB    │  │   S3 Bucket  │  │  ElastiCache │         │
│  │  (State)     │  │   (Audio)    │  │  (Sessions)  │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Detailed Component Architecture

### 1. Frontend Application (React + WebRTC)

**Technology Stack:**
- **Framework:** React 18 with TypeScript
- **Audio/Video:** WebRTC (RTCPeerConnection)
- **State Management:** Zustand or Redux Toolkit
- **UI Library:** Tailwind CSS + Shadcn/ui
- **Build Tool:** Vite
- **Deployment:** AWS Amplify Hosting

**Key Components:**

```typescript
// Core Application Structure
src/
├── components/
│   ├── MeetingRoom/
│   │   ├── VideoGrid.tsx          // Participant video tiles
│   │   ├── AudioVisualizer.tsx    // Waveform when AI speaks
│   │   ├── ParticipantCard.tsx    // Individual user display
│   │   └── AIStatusIndicator.tsx  // Shows AI thinking/speaking
│   ├── Controls/
│   │   ├── AudioControls.tsx      // Mute/unmute
│   │   ├── VideoControls.tsx      // Camera on/off
│   │   └── AISettings.tsx         // Personality toggles
│   └── Chat/
│       ├── TranscriptView.tsx     // Real-time transcript
│       └── SummaryPanel.tsx       // AI-generated summary
├── services/
│   ├── websocket.service.ts       // WebSocket connection
│   ├── webrtc.service.ts          // Peer connections
│   └── audio.service.ts           // Audio processing
├── hooks/
│   ├── useAudioStream.ts
│   ├── useMeetingState.ts
│   └── useAIParticipant.ts
└── types/
    ├── meeting.types.ts
    └── ai.types.ts
```

**Key Features:**
- Real-time audio streaming via WebRTC
- Visual indicators for active speakers
- AI "thinking" status display
- Live transcript with speaker identification
- Post-meeting summary generation

---

### 2. Backend Services (AWS Lambda + ECS)

**Service Architecture:**

#### 2.1 Connection Manager (AWS Lambda)
```python
# Lambda: connection-handler
# Triggered by: API Gateway WebSocket events

def lambda_handler(event, context):
    route_key = event['requestContext']['routeKey']
    connection_id = event['requestContext']['connectionId']
    
    if route_key == '$connect':
        return handle_connect(connection_id)
    elif route_key == '$disconnect':
        return handle_disconnect(connection_id)
    elif route_key == 'audio-stream':
        return handle_audio_stream(connection_id, event['body'])
```

**Responsibilities:**
- Manage WebSocket connections
- Route audio streams to processing pipeline
- Broadcast messages to all participants
- Handle connection lifecycle

#### 2.2 Audio Processing Pipeline (ECS Container)
```python
# Container: audio-processor
# Running on: AWS ECS Fargate

class AudioProcessor:
    def __init__(self):
        self.nova_sonic = NovaS2SClient()
        self.buffer_manager = AudioBufferManager()
        self.vad = VoiceActivityDetector()
    
    async def process_stream(self, audio_chunk, session_id):
        # 1. Buffer incoming audio
        self.buffer_manager.add_chunk(audio_chunk, session_id)
        
        # 2. Detect speech activity
        if self.vad.is_speech_active(audio_chunk):
            # 3. Send to Nova 2 Sonic for transcription
            transcript = await self.nova_sonic.transcribe(
                audio_chunk,
                language='en-US',
                enable_speaker_diarization=True
            )
            
            # 4. Pass to Context Manager
            await self.context_manager.add_utterance(
                session_id, 
                transcript
            )
        
        # 5. Check if AI needs to respond
        if self.should_ai_respond(transcript):
            await self.trigger_ai_response(session_id, transcript)
```

**Key Technologies:**
- **Container Orchestration:** AWS ECS Fargate
- **Audio Format:** Opus codec (low latency)
- **VAD Library:** WebRTC VAD or Silero VAD
- **Streaming Protocol:** WebSocket with binary frames

#### 2.3 Context Manager (AWS Lambda)
```python
# Lambda: context-manager
# Purpose: Maintain conversation state

class ContextManager:
    def __init__(self):
        self.dynamodb = boto3.resource('dynamodb')
        self.table = self.dynamodb.Table('meeting-contexts')
        self.redis = ElastiCacheClient()
    
    def add_utterance(self, session_id, utterance):
        """Add new speech to context window"""
        context = self.get_context(session_id)
        context['messages'].append({
            'speaker': utterance['speaker'],
            'text': utterance['text'],
            'timestamp': time.time(),
            'sentiment': self.analyze_sentiment(utterance['text'])
        })
        
        # Keep only last 50 messages for context
        context['messages'] = context['messages'][-50:]
        
        # Update cache and DB
        self.redis.set(f"context:{session_id}", context, ex=3600)
        self.table.put_item(Item=context)
        
        return context
    
    def should_ai_respond(self, context, latest_utterance):
        """Determine if AI should speak"""
        text_lower = latest_utterance['text'].lower()
        
        # Wake words
        wake_words = ['hey alex', 'alex', 'ai', 'what do you think']
        if any(word in text_lower for word in wake_words):
            return True
        
        # Question detection
        if text_lower.endswith('?') and len(context['messages']) > 5:
            return True
        
        # Pause detection (nobody spoke for 3 seconds)
        if context['silence_duration'] > 3.0:
            return True
        
        return False
```

**Database Schema (DynamoDB):**
```json
{
  "session_id": "string (partition key)",
  "timestamp": "number (sort key)",
  "meeting_metadata": {
    "title": "string",
    "participants": ["user1", "user2", "alex_ai"],
    "start_time": "timestamp",
    "ai_personality": "string"
  },
  "messages": [
    {
      "speaker": "string",
      "text": "string",
      "timestamp": "number",
      "sentiment": "positive|neutral|negative",
      "audio_url": "s3://path/to/audio.opus"
    }
  ],
  "ai_state": {
    "last_response_time": "timestamp",
    "topics_discussed": ["topic1", "topic2"],
    "pending_questions": []
  }
}
```

#### 2.4 AI Response Generator (AWS Lambda)
```python
# Lambda: ai-response-generator
# Purpose: Generate intelligent responses using Nova

class AIResponseGenerator:
    def __init__(self):
        self.bedrock = boto3.client('bedrock-runtime')
        self.nova_lite_model = 'amazon.nova-lite-v2'
        self.nova_sonic_model = 'amazon.nova-sonic-v2'
    
    async def generate_response(self, session_id, context):
        # 1. Build prompt with full context
        prompt = self.build_prompt(context)
        
        # 2. Get reasoning from Nova 2 Lite
        response = await self.bedrock.invoke_model(
            modelId=self.nova_lite_model,
            body=json.dumps({
                'messages': [
                    {
                        'role': 'system',
                        'content': self.get_system_prompt(context['ai_personality'])
                    },
                    {
                        'role': 'user',
                        'content': prompt
                    }
                ],
                'inferenceConfig': {
                    'temperature': 0.7,
                    'maxTokens': 200,
                    'topP': 0.9
                }
            })
        )
        
        # 3. Extract response text
        response_data = json.loads(response['body'].read())
        ai_text = response_data['output']['message']['content'][0]['text']
        
        # 4. Add human-like elements
        natural_text = self.add_filler_words(ai_text)
        
        # 5. Convert to speech using Nova 2 Sonic
        audio_stream = await self.text_to_speech(natural_text)
        
        return {
            'text': natural_text,
            'audio': audio_stream,
            'reasoning': response_data.get('reasoning', '')
        }
    
    def get_system_prompt(self, personality):
        """System prompt for Nova 2 Lite"""
        base_prompt = """You are Alex, an AI meeting participant. You:
        - Speak naturally with occasional filler words (um, well, you know)
        - Keep responses under 30 seconds
        - Challenge bad ideas constructively
        - Build on good ideas with specific suggestions
        - Ask clarifying questions when needed
        - Reference previous parts of the conversation
        - Show personality and emotion in your language
        """
        
        personality_prompts = {
            'supportive': 'Be encouraging and find positives in ideas.',
            'critical': 'Be a devil\'s advocate and find potential issues.',
            'technical': 'Focus on implementation details and feasibility.',
            'business': 'Focus on ROI, market fit, and business value.'
        }
        
        return base_prompt + personality_prompts.get(personality, '')
    
    def add_filler_words(self, text):
        """Make speech more natural"""
        words = text.split()
        fillers = ['um', 'uh', 'well', 'you know', 'I think', 'sort of']
        
        # Add filler at natural pause points
        enhanced = []
        for i, word in enumerate(words):
            if i > 0 and i % 10 == 0 and random.random() < 0.3:
                enhanced.append(random.choice(fillers))
            enhanced.append(word)
        
        return ' '.join(enhanced)
    
    async def text_to_speech(self, text):
        """Convert text to natural speech using Nova 2 Sonic"""
        response = await self.bedrock.invoke_model(
            modelId=self.nova_sonic_model,
            body=json.dumps({
                'text': text,
                'voiceConfig': {
                    'voiceId': 'matthew',  # Choose voice
                    'engine': 'nova-sonic',
                    'speakingRate': '1.0',
                    'pitch': '0',
                    'emotion': 'neutral'
                },
                'outputConfig': {
                    'format': 'opus',  # Low latency format
                    'sampleRate': 24000
                }
            })
        )
        
        return response['audioStream']
```

---

### 3. Amazon Nova Integration

#### 3.1 Nova 2 Sonic (Speech-to-Speech)

**Use Cases:**
1. **Speech Recognition:** Convert participant audio to text
2. **Speech Synthesis:** Convert AI responses to natural voice
3. **Real-time Processing:** Low-latency conversational flow

**Configuration:**
```python
import boto3
from botocore.config import Config

# Configure for low latency
config = Config(
    region_name='us-east-1',
    retries={'max_attempts': 3, 'mode': 'adaptive'},
    connect_timeout=5,
    read_timeout=30
)

bedrock = boto3.client('bedrock-runtime', config=config)

# Nova 2 Sonic: Speech Recognition
def transcribe_audio(audio_bytes):
    response = bedrock.invoke_model_with_response_stream(
        modelId='amazon.nova-sonic-v2',
        body=json.dumps({
            'audioInput': {
                'content': base64.b64encode(audio_bytes).decode(),
                'format': 'opus'
            },
            'task': 'transcription',
            'config': {
                'language': 'en-US',
                'enableSpeakerDiarization': True,
                'maxSpeakers': 10
            }
        })
    )
    
    # Stream results
    for event in response['body']:
        chunk = json.loads(event['chunk']['bytes'])
        yield chunk['transcript']

# Nova 2 Sonic: Speech Synthesis
def synthesize_speech(text, voice_config):
    response = bedrock.invoke_model_with_response_stream(
        modelId='amazon.nova-sonic-v2',
        body=json.dumps({
            'text': text,
            'task': 'synthesis',
            'voiceConfig': voice_config
        })
    )
    
    # Stream audio back to client
    for event in response['body']:
        audio_chunk = event['chunk']['bytes']
        yield audio_chunk
```

#### 3.2 Nova 2 Lite (Reasoning)

**Use Cases:**
1. **Context Understanding:** Analyze conversation history
2. **Response Generation:** Create intelligent replies
3. **Sentiment Analysis:** Detect meeting tone
4. **Topic Extraction:** Identify key discussion points

**Configuration:**
```python
def generate_meeting_response(context_messages, personality):
    # Build conversation history
    messages = [
        {'role': 'system', 'content': get_system_prompt(personality)}
    ]
    
    # Add recent conversation
    for msg in context_messages[-20:]:  # Last 20 messages
        messages.append({
            'role': 'user' if msg['speaker'] != 'alex' else 'assistant',
            'content': f"{msg['speaker']}: {msg['text']}"
        })
    
    # Invoke Nova 2 Lite
    response = bedrock.invoke_model(
        modelId='amazon.nova-lite-v2',
        body=json.dumps({
            'messages': messages,
            'inferenceConfig': {
                'temperature': 0.7,
                'maxTokens': 250,
                'topP': 0.9,
                'stopSequences': ['\n\n']
            }
        })
    )
    
    result = json.loads(response['body'].read())
    return result['output']['message']['content'][0]['text']
```

#### 3.3 Optional: Nova Multimodal Embeddings

**Use Case:** Screen sharing analysis

If time permits, add ability for AI to "see" shared screens:

```python
def analyze_shared_screen(image_bytes, context):
    """Use Nova multimodal embeddings to understand shared content"""
    response = bedrock.invoke_model(
        modelId='amazon.nova-multimodal-embedding-v1',
        body=json.dumps({
            'inputImage': base64.b64encode(image_bytes).decode(),
            'inputText': f"Analyze this screen in context of: {context['current_topic']}"
        })
    )
    
    # Get embeddings and compare with text context
    embeddings = json.loads(response['body'].read())
    return analyze_image_relevance(embeddings, context)
```

---

## 📊 Data Flow Diagram

### Real-Time Conversation Flow

```
Participant speaks
    ↓
[Browser captures audio]
    ↓
[WebSocket sends audio chunks]
    ↓
[API Gateway routes to ECS]
    ↓
[Audio Processor buffers & detects speech]
    ↓
[Nova 2 Sonic transcribes speech]
    ↓
[Context Manager adds to conversation history]
    ↓
[Decision: Should AI respond?]
    ↓ YES
[Query Nova 2 Lite with full context]
    ↓
[Generate response text with personality]
    ↓
[Add natural filler words & pauses]
    ↓
[Nova 2 Sonic synthesizes speech]
    ↓
[Stream audio back via WebSocket]
    ↓
[Browser plays AI audio]
    ↓
[All participants hear AI response]
```

**Target Latency:**
- Audio capture → transcription: **< 500ms**
- Transcription → AI thinking: **< 1000ms**
- AI response → speech synthesis: **< 800ms**
- **Total: < 2.3 seconds** (acceptable for natural conversation)

---

## 🎨 User Experience Design

### Meeting Room UI

```
┌─────────────────────────────────────────────────────────────┐
│  Alex AI Meeting Room                    [⚙️ Settings] [🚪 Leave] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  User 1  │  │  User 2  │  │  User 3  │  │   Alex   │  │
│  │  Video   │  │  Video   │  │  Video   │  │   AI     │  │
│  │  🎤 ON   │  │  🎤 OFF  │  │  🎤 ON   │  │  🤖💭    │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Live Transcript:                                           │
│  ─────────────────────────────────────────────────────────  │
│  User1: "Should we add dark mode?"                          │
│  User2: "Seems like extra work..."                          │
│  User3: "Hey Alex, what's your take?"                       │
│  Alex: "Well... I see both sides. Dark mode is pretty       │
│         expected in 2026, especially for productivity       │
│         apps. But if resources are tight, maybe just        │
│         respect system preferences first?"                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
│  [🎤 Mute] [🎥 Camera] [💬 Chat] [📊 Summary] [⚡ AI: Active] │
└─────────────────────────────────────────────────────────────┘
```

### AI Personality Settings

```
Alex's Personality:
○ Supportive      - Encouraging, finds positives
● Critical        - Devil's advocate, challenges ideas
○ Technical       - Focuses on implementation
○ Business        - Focuses on ROI and value

Participation Level:
[━━━━━━⚪━━━━] 60%
Less Active ←────────→ More Active

Features:
☑ Enable wake word ("Hey Alex")
☑ Allow proactive suggestions
☑ Show AI reasoning
☐ Record meeting (with consent)
```

---

## 🚀 Deployment Architecture

### AWS Infrastructure

```yaml
# Infrastructure as Code (AWS CDK)
Resources:
  
  # Frontend Hosting
  AmplifyApp:
    Type: AWS::Amplify::App
    Properties:
      Name: alex-ai-frontend
      Repository: github.com/yourteam/alex-ai
      BuildSpec: |
        version: 1
        frontend:
          phases:
            build:
              commands:
                - npm install
                - npm run build
          artifacts:
            baseDirectory: dist
            files:
              - '**/*'
  
  # WebSocket API
  WebSocketAPI:
    Type: AWS::ApiGatewayV2::Api
    Properties:
      Name: alex-ai-websocket
      ProtocolType: WEBSOCKET
      RouteSelectionExpression: "$request.body.action"
  
  # Lambda Functions
  ConnectionHandlerFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.lambda_handler
      Code: ./lambda/connection-handler
      Environment:
        Variables:
          DYNAMODB_TABLE: !Ref MeetingContextTable
          REDIS_ENDPOINT: !GetAtt ElastiCacheCluster.RedisEndpoint
  
  ContextManagerFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.lambda_handler
      Code: ./lambda/context-manager
      Timeout: 30
      MemorySize: 1024
  
  ResponseGeneratorFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.11
      Handler: index.lambda_handler
      Code: ./lambda/response-generator
      Timeout: 30
      MemorySize: 2048
      Environment:
        Variables:
          BEDROCK_REGION: us-east-1
  
  # ECS Cluster for Audio Processing
  AudioProcessingCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: alex-ai-audio-processor
  
  AudioProcessingTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: audio-processor
      NetworkMode: awsvpc
      RequiresCompatibilities:
        - FARGATE
      Cpu: 2048
      Memory: 4096
      ContainerDefinitions:
        - Name: audio-processor
          Image: !Sub ${AWS::AccountId}.dkr.ecr.${AWS::Region}.amazonaws.com/alex-ai-audio:latest
          Essential: true
          PortMappings:
            - ContainerPort: 8080
  
  # DynamoDB Table
  MeetingContextTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: alex-ai-meeting-contexts
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: session_id
          AttributeType: S
        - AttributeName: timestamp
          AttributeType: N
      KeySchema:
        - AttributeName: session_id
          KeyType: HASH
        - AttributeName: timestamp
          KeyType: RANGE
      StreamSpecification:
        StreamViewType: NEW_AND_OLD_IMAGES
  
  # ElastiCache for Redis
  ElastiCacheCluster:
    Type: AWS::ElastiCache::CacheCluster
    Properties:
      CacheNodeType: cache.t3.micro
      Engine: redis
      NumCacheNodes: 1
  
  # S3 Bucket for Audio Storage
  AudioStorageBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: alex-ai-audio-storage
      LifecycleConfiguration:
        Rules:
          - Id: DeleteOldAudio
            Status: Enabled
            ExpirationInDays: 30
  
  # Bedrock Access IAM Role
  BedrockAccessRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: BedrockInvokeModel
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - bedrock:InvokeModel
                  - bedrock:InvokeModelWithResponseStream
                Resource:
                  - arn:aws:bedrock:*::foundation-model/amazon.nova-lite-v2
                  - arn:aws:bedrock:*::foundation-model/amazon.nova-sonic-v2
```

### Deployment Pipeline

```yaml
# GitHub Actions Workflow
name: Deploy Alex AI

on:
  push:
    branches: [main]

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm run build
      - run: aws amplify push

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: aws-actions/setup-sam@v2
      - run: sam build
      - run: sam deploy --no-confirm-changeset

  deploy-containers:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: docker build -t alex-ai-audio ./audio-processor
      - run: aws ecr get-login-password | docker login
      - run: docker push $ECR_REGISTRY/alex-ai-audio:latest
      - run: aws ecs update-service --force-new-deployment
```

---

## 👥 Team Roles & Responsibilities

### Equal Split: 25% per person (100% total)
### Each role divided into 10 milestones × 10% each = 100%

---

## 🎭 **ROLE 1: AI/ML Engineer & Nova Integration Lead**
**Developer:** TBD  
**Primary Focus:** Amazon Nova integration, AI reasoning, prompt engineering

### Responsibilities Breakdown (10 Milestones × 10% each)

| # | Milestone | Description | Deliverable | Deadline | Weight |
|---|-----------|-------------|-------------|----------|--------|
| 1 | Nova Account Setup | Set up AWS account, enable Bedrock, get Nova access | AWS credentials configured | Day 1 | 10% |
| 2 | Nova 2 Sonic Integration | Implement speech-to-text functionality | Working transcription service | Day 3 | 10% |
| 3 | Nova 2 Sonic TTS | Implement text-to-speech with voice synthesis | Working TTS service | Day 5 | 10% |
| 4 | Nova 2 Lite Setup | Configure reasoning model with proper prompts | Response generation working | Day 7 | 10% |
| 5 | Personality System | Implement 4 AI personality modes (supportive, critical, technical, business) | Personality switching works | Day 9 | 10% |
| 6 | Context Management | Build conversation context tracking and memory | Context history maintained | Day 11 | 10% |
| 7 | Response Logic | Implement wake word detection and response triggers | AI knows when to speak | Day 13 | 10% |
| 8 | Natural Language | Add filler words, pauses, human-like speech patterns | Responses feel natural | Day 15 | 10% |
| 9 | Latency Optimization | Reduce response time to < 2.5 seconds | Performance benchmarks met | Day 17 | 10% |
| 10 | Testing & Tuning | Test various scenarios, tune prompts and parameters | AI performs reliably | Day 19 | 10% |

**Required Skills:** Python, AWS Bedrock SDK, Prompt Engineering, ML Model Integration  
**Key Tools:** Amazon Bedrock, Nova 2 Sonic, Nova 2 Lite, boto3

---

## 💻 **ROLE 2: Backend Engineer & Infrastructure Lead**
**Developer:** TBD  
**Primary Focus:** AWS architecture, Lambda functions, WebSocket API, database

### Responsibilities Breakdown (10 Milestones × 10% each)

| # | Milestone | Description | Deliverable | Deadline | Weight |
|---|-----------|-------------|-------------|----------|--------|
| 1 | Infrastructure Planning | Design AWS architecture, create IaC templates | Architecture diagram + CDK code | Day 2 | 10% |
| 2 | WebSocket API Setup | Deploy API Gateway with WebSocket support | WebSocket endpoint live | Day 4 | 10% |
| 3 | Lambda Functions | Create connection handler, context manager, response generator | All Lambdas deployed | Day 6 | 10% |
| 4 | DynamoDB Schema | Design and deploy database for meeting contexts | Database operational | Day 8 | 10% |
| 5 | ElastiCache Setup | Configure Redis for session management | Cache layer working | Day 10 | 10% |
| 6 | S3 Audio Storage | Set up bucket for audio recordings with lifecycle | Storage configured | Day 12 | 10% |
| 7 | ECS Audio Processor | Deploy containerized audio processing service | Container running | Day 14 | 10% |
| 8 | API Integration | Connect all services end-to-end | Full backend working | Day 16 | 10% |
| 9 | Security & IAM | Configure proper roles, policies, and encryption | Security audit passed | Day 18 | 10% |
| 10 | Monitoring & Logs | Set up CloudWatch, alarms, and logging | Monitoring dashboard live | Day 20 | 10% |

**Required Skills:** AWS (Lambda, ECS, DynamoDB), Python/Node.js, Infrastructure as Code, WebSockets  
**Key Tools:** AWS CDK, SAM, CloudFormation, Docker

---

## 🎨 **ROLE 3: Frontend Engineer & UX Lead**
**Developer:** TBD  
**Primary Focus:** React application, WebRTC, user interface, user experience

### Responsibilities Breakdown (10 Milestones × 10% each)

| # | Milestone | Description | Deliverable | Deadline | Weight |
|---|-----------|-------------|-------------|----------|--------|
| 1 | Project Setup | Initialize React app with TypeScript, Vite, Tailwind | Boilerplate ready | Day 1 | 10% |
| 2 | UI Design System | Create component library and design tokens | Reusable components | Day 3 | 10% |
| 3 | Meeting Room Layout | Build video grid and participant cards | UI layout complete | Day 5 | 10% |
| 4 | WebRTC Integration | Implement peer-to-peer video/audio connections | Video calls working | Day 7 | 10% |
| 5 | Audio Controls | Build mute/unmute, volume, speaker selection | Audio controls functional | Day 9 | 10% |
| 6 | WebSocket Client | Connect to backend WebSocket API for real-time updates | Real-time messaging works | Day 11 | 10% |
| 7 | Live Transcript UI | Display real-time transcript with speaker labels | Transcript visible | Day 13 | 10% |
| 8 | AI Visualizations | Show AI status (listening, thinking, speaking) with animations | AI indicators working | Day 15 | 10% |
| 9 | Settings Panel | Build personality selection and AI control panel | Settings functional | Day 17 | 10% |
| 10 | Polish & Testing | Responsive design, error handling, cross-browser testing | Production-ready UI | Day 19 | 10% |

**Required Skills:** React, TypeScript, WebRTC, CSS/Tailwind, WebSocket Client  
**Key Tools:** React 18, Vite, WebRTC API, Zustand, Shadcn/ui

---

## 🎬 **ROLE 4: DevOps, Documentation & Demo Lead**
**Developer:** TBD  
**Primary Focus:** Deployment pipeline, testing, documentation, demo video

### Responsibilities Breakdown (10 Milestones × 10% each)

| # | Milestone | Description | Deliverable | Deadline | Weight |
|---|-----------|-------------|-------------|----------|--------|
| 1 | GitHub Repository | Set up repo structure, branching strategy, CI/CD | Repo configured | Day 1 | 10% |
| 2 | CI/CD Pipeline | Configure GitHub Actions for automated deployment | Pipeline working | Day 4 | 10% |
| 3 | Environment Setup | Create dev, staging, prod environments | Environments deployed | Day 6 | 10% |
| 4 | Integration Testing | Write end-to-end tests for critical flows | Test suite passing | Day 8 | 10% |
| 5 | Performance Testing | Load testing, latency measurement, bottleneck analysis | Performance report | Day 10 | 10% |
| 6 | Technical Documentation | Write architecture docs, API docs, setup guides | Docs published | Day 12 | 10% |
| 7 | User Documentation | Create user guide, FAQ, troubleshooting guide | User docs complete | Day 14 | 10% |
| 8 | Demo Video Script | Write script, plan scenes, coordinate team for recording | Script approved | Day 16 | 10% |
| 9 | Demo Video Production | Record, edit, and produce 3-minute demo video | Video ready | Day 18 | 10% |
| 10 | Submission Package | Prepare DevPost submission with all materials | Submitted to hackathon | Day 20 | 10% |

**Required Skills:** DevOps, CI/CD, Testing, Technical Writing, Video Production  
**Key Tools:** GitHub Actions, AWS CDK, Pytest, Postman, Video editing software

---

## 📋 Contribution-Based Performance System

### Core Rules

1. **Equal Base Split:** Each developer starts with 25% ownership
2. **Milestone Tracking:** Each role has 10 milestones worth 10% of that role's contribution
3. **Deadline System:** Each milestone has a specific deadline date
4. **Penalty Structure:**
   - Complete on time: Full 10% credit
   - 1-2 days late: 7% credit (3% redistributed)
   - 3-5 days late: 5% credit (5% redistributed)
   - 6+ days late: 2% credit (8% redistributed)
   - Not completed: 0% credit, milestone reassigned

### Redistribution Logic

When someone misses a deadline, the penalty is redistributed:
- **50%** goes to whoever completes the work (if reassigned)
- **50%** split equally among other team members who met their deadlines

### Emergency Reallocation

- 5% of total project equity is held in reserve
- Used to compensate anyone taking on extra work due to emergencies
- Distributed at project end based on team vote

### Blocking Dependencies

If your work is blocked by someone else's delay:
- Document the blocker
- No penalty applied to dependent milestones
- Deadline automatically extends by the delay period
- Blocker receives doubled penalty

### Quality Gates

All deliverables must pass peer review:
- Code review by at least 1 other team member
- Functional testing verification
- Documentation completeness check

**Partial Credit:**
- Works but has bugs: 7% credit
- Incomplete functionality: 5% credit
- Technical debt/needs refactor: 8% credit

### Weekly Sync

Every 3 days:
- Progress check-in
- Blocker identification
- Risk assessment
- Reallocation discussion if needed

### Example Scenarios

**Scenario 1: Backend Engineer completes all milestones on time**
- Earns full 25% contribution

**Scenario 2: AI Engineer misses 2 milestones (3 days late each)**
- Milestone 5: 5% instead of 10% (5% penalty)
- Milestone 7: 5% instead of 10% (5% penalty)
- Total: 80% of their milestones = 20% contribution instead of 25%
- 5% redistributed: 2.5% to whoever completed the work, 2.5% split among team

**Scenario 3: Frontend Engineer blocked by Backend API**
- Backend API delivery: Day 16 (2 days late)
- Frontend milestone depending on API: Deadline auto-extends by 2 days
- No penalty to Frontend Engineer
- Backend Engineer receives penalty for their delay

---

## 📅 Project Timeline (20 Days)

### Phase 1: Setup & Foundation (Days 1-5)
- Infrastructure provisioning
- Nova access configuration
- Basic UI framework
- Repository and CI/CD setup

**Milestone:** Basic app runs with mock AI responses

### Phase 2: Core Features (Days 6-12)
- Nova 2 Sonic integration
- WebRTC implementation
- Context management
- Database operations

**Milestone:** AI can listen and respond (even if slowly)

### Phase 3: Enhancement (Days 13-17)
- Personality system
- UI polish
- Performance optimization
- Natural language improvements

**Milestone:** App feels natural and polished

### Phase 4: Demo & Launch (Days 18-20)
- Demo video production
- Documentation finalization
- Bug fixes and testing
- DevPost submission

**Milestone:** Submitted to hackathon

---

## 🎯 Success Metrics

### Technical KPIs
- **Latency:** < 2.5 seconds from question to AI response
- **Uptime:** 99% during demo period
- **Audio Quality:** Clear, no artifacts or glitches
- **Concurrent Users:** Support 4-6 participants per session

### Judging Criteria Alignment

**Technical Implementation (60%):**
- ✅ Full Nova 2 Sonic integration (speech-to-speech)
- ✅ Full Nova 2 Lite integration (reasoning)
- ✅ Real-time WebRTC + WebSocket architecture
- ✅ Scalable AWS infrastructure
- ✅ Clean, documented codebase

**Enterprise/Community Impact (20%):**
- ✅ Solves real meeting fatigue problem
- ✅ Improves remote work collaboration
- ✅ Accessibility benefits (helps non-native speakers, neurodiverse individuals)
- ✅ Cost savings (reduces need for extra meeting participants)

**Creativity & Innovation (20%):**
- ✅ First active AI participant (not passive note-taker)
- ✅ Natural conversation with human-like speech
- ✅ Personality customization
- ✅ Novel use of speech-to-speech AI

---

## 📝 Submission Checklist

### Required Materials

- [ ] **GitHub Repository** (public or shared)
  - [ ] Complete source code
  - [ ] README with setup instructions
  - [ ] Architecture documentation
  - [ ] API documentation

- [ ] **Demo Video** (3 minutes)
  - [ ] Problem statement (30s)
  - [ ] Live demo (90s)
  - [ ] Technical overview (30s)
  - [ ] Impact & use cases (30s)
  - [ ] Includes #AmazonNova hashtag

- [ ] **Text Description**
  - [ ] Project summary
  - [ ] Nova integration details
  - [ ] Target audience & impact

- [ ] **Deployment**
  - [ ] Live demo URL
  - [ ] Working application
  - [ ] Test accounts/instructions

### Bonus Submissions

- [ ] **Blog Post** (for $200 AWS credits)
  - Published on builder.aws.com
  - Discusses community impact
  - Outlines adoption plans

- [ ] **Feedback Survey** (for $50 cash)
  - Complete after submission

---

## 🏆 Competitive Advantages

### Why Alex AI Will Win

**1. Perfect Nova Fit:**
- Uses Nova 2 Sonic for core functionality (not just a check-box)
- Uses Nova 2 Lite for intelligent reasoning
- Demonstrates true speech-to-speech (not text intermediary)

**2. Novel Approach:**
- First active participant AI (not passive transcription)
- Natural conversation with personality
- Human-like speech patterns

**3. Real Business Value:**
- Clear ROI for enterprises (reduces meeting overhead)
- Measurable benefits (time savings, better decisions)
- Scalable to multiple platforms (Zoom, Meet, Teams)

**4. Technical Excellence:**
- Low-latency architecture
- Scalable AWS infrastructure
- Clean, documented code
- Production-ready deployment

**5. Strong Demo:**
- Live, working application (not just slides)
- Judges can test it themselves
- Clear before/after comparison

---

## 🔒 Risk Mitigation

### Identified Risks & Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Nova API quota exceeded | Medium | High | Cache responses, optimize calls, request increase early |
| High latency (>3s response) | High | High | Stream responses, optimize pipeline, use Fargate not Lambda for audio |
| WebRTC connection issues | Medium | Medium | Fallback to TURN servers, test multiple browsers |
| Team member unavailable | Medium | High | Cross-training, clear documentation, reassignment protocol |
| AWS costs exceed budget | Medium | Medium | Set billing alarms, use free tier, optimize resource usage |
| Demo day technical issues | Low | High | Pre-record fallback video, test environment 24h before |

---

## 💰 Budget Estimate

### Hackathon Development Period (20 days)

| Service | Estimated Cost | Notes |
|---------|----------------|-------|
| Amazon Bedrock (Nova) | $50-100 | ~10K requests during development |
| AWS Lambda | $5-10 | Free tier likely sufficient |
| AWS ECS Fargate | $20-30 | 1 task running |
| DynamoDB | $5-10 | Pay-per-request mode |
| ElastiCache | $15-20 | t3.micro instance |
| S3 Storage | $2-5 | Audio storage |
| API Gateway | $5-10 | WebSocket connections |
| Amplify Hosting | $0 | Free tier |
| **Total** | **$102-185** | Well within free tier + credits |

**Post-Hackathon (if continuing):**
- With $5K AWS credits from winning, can run production for 6-12 months

---

## 📚 Resources & References

### Amazon Nova Documentation
- [Nova 2 Developer Guide](https://docs.aws.amazon.com/nova/)
- [Bedrock API Reference](https://docs.aws.amazon.com/bedrock/)
- [Nova 2 Sonic Examples](https://github.com/aws-samples/nova-sonic-examples)

### Technical Resources
- [WebRTC Samples](https://webrtc.github.io/samples/)
- [AWS CDK Examples](https://github.com/aws-samples/aws-cdk-examples)
- [Real-time Audio Processing](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

### Inspiration & Research
- [Voice Activity Detection Paper](https://arxiv.org/abs/2010.12866)
- [Conversational AI Best Practices](https://aws.amazon.com/blogs/machine-learning/)
- [Meeting Collaboration Research](https://hbr.org/2022/03/how-to-run-better-virtual-meetings)

---

## 🤝 Team Agreement

### By signing below, all team members agree to:

1. Commit to the 20-day development timeline
2. Accept the contribution-based equity split system
3. Meet assigned milestones by deadlines or face documented penalties
4. Participate in bi-weekly sync meetings
5. Provide peer reviews for other team members' work
6. Communicate blockers immediately
7. Maintain code quality and documentation standards
8. Share credit equally in public presentations unless contribution split differs significantly

**Signatures:**

- Developer 1 (AI/ML Lead): _________________ Date: _______
- Developer 2 (Backend Lead): _________________ Date: _______
- Developer 3 (Frontend Lead): _________________ Date: _______
- Developer 4 (DevOps Lead): _________________ Date: _______

---

## 📞 Contact & Support

**Project Repository:** https://github.com/yourteam/alex-ai  
**Team Communication:** Slack/Discord Channel  
**Weekly Sync:** Every 3 days via Zoom  

**AWS Support:**
- Office Hours: [Hackathon Schedule]
- Technical Questions: Amazon Nova Discord

**DevPost Submission:**
- Deadline: [Hackathon End Date]
- Submission: testing@devpost.com + Amazon-Nova-hackathon@amazon.com

---

## ✨ Future Enhancements (Post-Hackathon)

If we continue after winning:

1. **Platform Integrations:**
   - Native Zoom bot
   - Google Meet integration
   - Microsoft Teams support

2. **Advanced Features:**
   - Screen sharing analysis (Nova multimodal)
   - Multi-language support
   - Sentiment tracking over time
   - Action item extraction

3. **Enterprise Features:**
   - Team analytics
   - Meeting insights dashboard
   - Custom AI training per organization
   - Security & compliance certifications

4. **Monetization:**
   - Freemium: 10 meetings/month free
   - Pro: $29/month unlimited
   - Enterprise: Custom pricing

---

**Document Version:** 1.0  
**Last Updated:** February 20, 2026  
**Status:** Ready for Team Review

---

## Appendix A: Quick Start Commands

```bash
# Clone repository
git clone https://github.com/yourteam/alex-ai.git
cd alex-ai

# Install dependencies
npm install  # Frontend
pip install -r requirements.txt  # Backend

# Set up AWS credentials
aws configure

# Deploy infrastructure
cd infrastructure
cdk deploy

# Run locally
npm run dev  # Frontend
python app.py  # Backend

# Run tests
npm test
pytest
```

---

## Appendix B: API Endpoints

```
WebSocket API: wss://api.alexai.com/ws
- /connect - Establish connection
- /audio-stream - Send audio chunks
- /get-transcript - Retrieve transcript
- /disconnect - Close connection

REST API: https://api.alexai.com/v1
- POST /meetings - Create new meeting
- GET /meetings/:id - Get meeting details
- GET /meetings/:id/summary - Get AI summary
- PUT /ai/personality - Update AI settings
```

---

*End of Technical Architecture Document*
