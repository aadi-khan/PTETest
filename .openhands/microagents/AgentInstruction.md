# OpenHands PTE SaaS Development Prompts - Your Kotlin Multiplatform Project

## Overview
This document contains concrete, location-specific prompts for building your PTE (Pearson Test of English) SaaS application using your existing Kotlin Multiplatform project structure. Each prompt follows OpenHands best practices and is scoped appropriately.

**Project Structure:**
- `composeApp/` - Frontend (Android, iOS, Desktop, WASM)
- `server/` - Backend with Ktor
- `shared/` - Common business logic
- Package: `org.example.project`

---

## Phase 1: Project Dependencies Setup

### Prompt 1: Update Dependencies for PTE App
```
Update gradle/libs.versions.toml to add PTE-specific dependencies:
- Add kotlinx-serialization version "1.7.3"
- Add ktor-client-core version "3.2.3" for multiplatform HTTP client
- Add ktor-server-content-negotiation and ktor-serialization-kotlinx-json for server
- Add kotlinx-datetime version "0.6.0"
- Add ktor-server-auth-jwt version "3.2.3" for JWT authentication
- Add bcrypt version "0.10.2" for password hashing

Also add the corresponding library entries in the [libraries] section.
```

### Prompt 2: Configure Shared Module for PTE Business Logic
```
Update shared/build.gradle.kts sourceSets commonMain.dependencies to include:
- implementation("org.jetbrains.kotlinx:kotlinx-serialization-core:1.7.3")
- implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.6.0")
- implementation("io.ktor:ktor-client-core:3.2.3")
- implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.2")

Add the kotlinx-serialization plugin to the plugins block.
```

### Prompt 3: Setup Server Dependencies
```
Update server/build.gradle.kts dependencies to add:
- implementation("io.ktor:ktor-server-content-negotiation-jvm:3.2.3")
- implementation("io.ktor:ktor-serialization-kotlinx-json-jvm:3.2.3")
- implementation("io.ktor:ktor-server-auth-jvm:3.2.3")
- implementation("io.ktor:ktor-server-auth-jwt-jvm:3.2.3")
- implementation("io.ktor:ktor-server-cors-jvm:3.2.3")
- implementation("at.favre.lib:bcrypt:0.10.2")

These will enable JSON serialization, JWT authentication, and CORS support.
```

---

## Phase 2: Data Models in Shared Module

### Prompt 4: Create User Data Models
```
Create shared/src/commonMain/kotlin/org/example/project/models/User.kt with:
- User data class with id, email, passwordHash, subscriptionTier, createdAt, lastLogin
- AuthRequest data class for login/signup with email and password fields
- AuthResponse data class with token, user info, and expiresIn
- SubscriptionTier enum with FREE, BASIC, PREMIUM, PRO values
Use @Serializable annotations for all data classes.
```

### Prompt 5: Create PTE Question Models
```
Create shared/src/commonMain/kotlin/org/example/project/models/Question.kt with:
- Question data class with id, type, content, difficulty, topic, correctAnswer, audioUrl (optional)
- QuestionType enum covering all PTE question types: READ_ALOUD, REPEAT_SENTENCE, DESCRIBE_IMAGE, RETELL_LECTURE, ANSWER_SHORT_QUESTION, SUMMARIZE_WRITTEN_TEXT, WRITE_ESSAY, etc.
- DifficultyLevel enum with BEGINNER, INTERMEDIATE, ADVANCED
- QuestionResponse data class for user answers with questionId, userAnswer, audioData (ByteArray?), submissionTime
```

### Prompt 6: Create Scoring and Progress Models
```
Create shared/src/commonMain/kotlin/org/example/project/models/Scoring.kt with:
- SpeakingScore data class with pronunciationScore, fluencyScore, contentScore, totalScore, feedback
- WritingScore data class with grammarScore, vocabularyScore, coherenceScore, totalScore, feedback
- MockTestResult data class with userId, testId, moduleScores, totalScore, completionTime, detailedFeedback
- UserProgress data class with userId, questionId, attempts, bestScore, lastAttempted
```

---

## Phase 3: Repository Layer in Shared Module

### Prompt 7: Create HTTP Client Configuration
```
Create shared/src/commonMain/kotlin/org/example/project/network/HttpClientFactory.kt with:
- A function createHttpClient() that returns a configured Ktor HttpClient
- Install ContentNegotiation with JSON serialization
- Configure timeout settings (request: 30s, connect: 10s)
- Add default headers for Content-Type application/json
- Include error handling for network exceptions
```

### Prompt 8: Create Authentication Repository
```
Create shared/src/commonMain/kotlin/org/example/project/repository/AuthRepository.kt with:
- Interface AuthRepository defining login(), register(), refreshToken() suspend functions
- AuthRepositoryImpl class with HttpClient dependency
- login() method that POST to /api/auth/login endpoint
- register() method that POST to /api/auth/register endpoint  
- Proper error handling with sealed Result class for network responses
- Token storage interface for platform-specific implementations
```

### Prompt 9: Create Question Repository
```
Create shared/src/commonMain/kotlin/org/example/project/repository/QuestionRepository.kt with:
- Interface defining getQuestionsByType(), getRandomQuestions(), submitAnswer() functions
- QuestionRepositoryImpl with methods that call appropriate server endpoints
- getQuestionsByType() calling GET /api/questions/{type} with pagination
- submitAnswer() calling POST /api/questions/submit with user response
- Include caching logic for offline question access
```

---

## Phase 4: Server API Implementation

### Prompt 10: Configure Ktor Server with Plugins
```
Update server/src/main/kotlin/org/example/project/Application.kt to configure:
- ContentNegotiation with JSON serialization
- CORS allowing localhost origins for development
- JWT authentication with secret key and validation
- Routing plugin
- StatusPages for error handling with proper HTTP status codes
- CallLogging for debugging
Configure server to run on port 8080 with proper exception handling.
```

### Prompt 11: Create Authentication Routes
```
Create server/src/main/kotlin/org/example/project/routes/AuthRoutes.kt with:
- Route function that defines /api/auth endpoints
- POST /api/auth/register endpoint: validate email format, check if user exists, hash password with BCrypt, save user, return JWT token
- POST /api/auth/login endpoint: validate credentials, verify password with BCrypt, generate JWT with user info
- POST /api/auth/refresh endpoint: validate existing token and return new token
- Include proper error responses (400, 401, 409, 500) with descriptive messages
```

### Prompt 12: Create Question Management Routes  
```
Create server/src/main/kotlin/org/example/project/routes/QuestionRoutes.kt with:
- GET /api/questions/{type} endpoint with query parameters for difficulty and pagination
- GET /api/questions/random/{type} endpoint returning random questions for practice
- POST /api/questions/submit endpoint accepting user answers and returning scores
- POST /api/questions (admin only) for adding new questions with JWT authentication
- Include request validation and proper error handling for all endpoints
```

---

## Phase 5: AI Scoring Engine in Shared Module

### Prompt 13: Create Speaking Assessment Algorithm
```
Create shared/src/commonMain/kotlin/org/example/project/scoring/SpeakingScorer.kt with:
- SpeakingScorer class with methods: analyzePronunciation(), assessFluency(), evaluateContent()
- analyzePronunciation() function that processes audio data and returns pronunciation score (0-90)
- assessFluency() function measuring speech rate, pause frequency, and rhythm patterns
- evaluateContent() function for describe image/retell lecture tasks comparing keywords
- combineScores() method that applies PTE weighting to calculate final score
- Return detailed SpeakingScore object with feedback strings for improvement
```

### Prompt 14: Create Writing Assessment Algorithm
```
Create shared/src/commonMain/kotlin/org/example/project/scoring/WritingScorer.kt with:
- WritingScorer class with methods for grammar, vocabulary, and coherence analysis
- analyzeGrammar() function that identifies common error types and counts them
- assessVocabulary() function checking academic word usage and lexical diversity
- evaluateCoherence() function analyzing paragraph structure and logical flow
- calculateWritingScore() method combining all metrics into 0-90 scale score
- Include specific feedback generation for different error categories
```

### Prompt 15: Create Mock Test Engine
```
Create shared/src/commonMain/kotlin/org/example/project/test/MockTestEngine.kt with:
- MockTestEngine class that manages complete PTE test sessions
- generateTest() method creating balanced test with proper question distribution per PTE specifications
- MockTestSession class tracking current question, remaining time, user responses
- calculateFinalScore() method combining module scores using official PTE weightings
- saveTestResult() method persisting results with detailed performance breakdown
- Include proper timing controls and test navigation logic
```

---

## Phase 6: Compose UI Components

### Prompt 16: Create Authentication Screens
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/screens/auth/LoginScreen.kt with:
- LoginScreen composable with email and password text fields
- Form validation showing error states for invalid email or empty fields  
- Login button with loading state during authentication
- Navigation to SignupScreen and main app after successful login
- Use Material3 design with proper spacing and typography
- Include "Remember Me" checkbox and "Forgot Password" link styling
```

### Prompt 17: Create Main Navigation Structure
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/navigation/PteNavigation.kt with:
- Navigation graph using Compose Navigation
- Define routes for Practice, MockTest, Progress, Profile screens
- BottomNavigationBar with icons for main sections
- Proper back stack handling and deep linking support
- Include authentication checks before accessing protected screens
- Use sealed class for navigation destinations and type-safe arguments
```

### Prompt 18: Create Practice Question Card Component
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/components/QuestionCard.kt with:
- QuestionCard composable displaying question text, timer, and difficulty badge
- Support for different question types with appropriate layouts
- AudioPlayerButton component for listening questions with play/pause states
- RecordingButton component for speaking questions with visual feedback
- SubmitButton component that handles answer submission and loading states
- Proper accessibility labels and 48dp minimum touch targets
```

---

## Phase 7: Platform-Specific Audio Implementation

### Prompt 19: Create Audio Recording Interface
```
Create shared/src/commonMain/kotlin/org/example/project/audio/AudioRecorder.kt with:
- AudioRecorder interface defining startRecording(), stopRecording(), getAudioData() methods
- AudioRecordingState sealed class with IDLE, RECORDING, STOPPED states
- AudioRecordingResult sealed class for success/error handling
- RecordingSettings data class with sampleRate, bitRate, format configuration
- Include proper coroutine support for async recording operations
```

### Prompt 20: Implement Android Audio Recording
```
Create composeApp/src/androidMain/kotlin/org/example/project/audio/AndroidAudioRecorder.kt with:
- AndroidAudioRecorder class implementing AudioRecorder interface
- Use MediaRecorder API with proper lifecycle management
- Request RECORD_AUDIO permission with ActivityResultContracts
- Configure audio: 44.1kHz sample rate, AAC encoding, 128kbps bitrate
- Handle recording interruptions (phone calls, other apps) gracefully
- Save recordings to app internal storage with timestamp-based naming
- Include proper error handling for permission denied and storage full scenarios
```

### Prompt 21: Implement iOS Audio Recording
```
Create composeApp/src/iosMain/kotlin/org/example/project/audio/IOSAudioRecorder.kt with:
- IOSAudioRecorder class implementing AudioRecorder interface using interop
- Configure AVAudioSession with AVAudioSessionCategoryPlayAndRecord
- Request microphone permission and handle user response
- Use AVAudioRecorder with same quality settings as Android (44.1kHz, AAC)
- Handle audio session interruptions and route changes
- Implement proper memory management for audio buffers and file handles
```

---

## Phase 8: ViewModels and State Management

### Prompt 22: Create Authentication ViewModel
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/viewmodel/AuthViewModel.kt with:
- AuthViewModel extending ViewModel with AuthRepository dependency
- LoginState data class with loading, error, and success states
- login() suspend function handling user authentication with error handling
- register() suspend function for user registration with validation
- MutableStateFlow for UI state management and StateFlow for UI observation
- Proper coroutine scope management and cancellation handling
```

### Prompt 23: Create Practice Session ViewModel
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/viewmodel/PracticeViewModel.kt with:
- PracticeViewModel with QuestionRepository and AudioRecorder dependencies
- PracticeState data class tracking current question, user answer, recording state, score
- loadQuestion() method fetching questions by type and difficulty
- submitAnswer() method sending responses to server and receiving scores
- Audio recording state management with proper cleanup
- Timer functionality for timed questions with countdown display
```

### Prompt 24: Create Progress Analytics ViewModel
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/viewmodel/ProgressViewModel.kt with:
- ProgressViewModel managing user performance data and analytics
- ProgressState containing weekly scores, weak areas, recommendations
- loadProgressData() method fetching user statistics from server
- calculateTrends() method analyzing performance over time
- generateRecommendations() method suggesting focus areas based on scores
- Chart data preparation for progress visualization components
```

---

## Phase 9: Testing Implementation

### Prompt 25: Create Repository Tests
```
Create shared/src/commonTest/kotlin/org/example/project/repository/AuthRepositoryTest.kt with:
- Unit tests for AuthRepositoryImpl using MockEngine for HTTP client
- Test successful login with correct credentials returning valid AuthResponse
- Test failed login with invalid credentials returning appropriate error
- Test network error scenarios with proper error handling
- Test user registration with email validation and duplicate user scenarios
- Use kotlinx-coroutines-test for testing suspend functions
```

### Prompt 26: Create Scoring Algorithm Tests
```
Create shared/src/commonTest/kotlin/org/example/project/scoring/SpeakingScorerTest.kt with:
- Unit tests for SpeakingScorer with mock audio data
- Test pronunciation scoring with different audio quality samples
- Test fluency calculation with various speech patterns (fast, slow, many pauses)
- Test edge cases: empty audio, very short recordings, background noise
- Verify score ranges are within 0-90 bounds
- Test feedback message generation for different score ranges
```

### Prompt 27: Create Server Route Tests
```
Create server/src/test/kotlin/org/example/project/routes/AuthRoutesTest.kt with:
- Integration tests for authentication endpoints using Ktor testApplication
- Test POST /api/auth/register with valid data returns 201 and JWT token
- Test POST /api/auth/login with correct credentials returns 200 and token  
- Test authentication with invalid data returns appropriate 400/401 errors
- Test JWT token validation on protected endpoints
- Use in-memory database or mocking for isolated testing
```

---

## Phase 10: Advanced Features Implementation

### Prompt 28: Create Offline Data Synchronization
```
Create shared/src/commonMain/kotlin/org/example/project/sync/OfflineSyncManager.kt with:
- OfflineSyncManager class handling data synchronization between local and server
- queueOfflineAction() method storing user actions when network is unavailable
- syncWhenOnline() method uploading queued data when connection is restored
- downloadForOfflineUse() method caching questions and user data locally
- ConflictResolution sealed class handling data conflicts during sync
- Use kotlinx-coroutines Flow for reactive sync status updates
```

### Prompt 29: Create Real-time Feedback System  
```
Create shared/src/commonMain/kotlin/org/example/project/feedback/RealtimeFeedback.kt with:
- RealtimeFeedbackEngine providing instant scoring during practice
- processAudioInRealtime() method analyzing speech as user speaks
- generateInstantWritingFeedback() method checking grammar while typing
- FeedbackType sealed class with PRONUNCIATION, GRAMMAR, FLUENCY, VOCABULARY variants
- LiveFeedback data class with score, suggestions, and confidence level
- Debouncing mechanism to avoid excessive API calls during real-time analysis
```

### Prompt 30: Create Performance Analytics Dashboard
```
Create composeApp/src/commonMain/kotlin/org/example/project/ui/screens/AnalyticsScreen.kt with:
- AnalyticsScreen composable displaying comprehensive performance metrics
- ScoreProgressChart component showing score trends over time using Canvas API
- WeakAreasCard component highlighting areas needing improvement
- StudyTimeCard component showing daily/weekly practice statistics
- RecommendationsList component with personalized study suggestions
- Use LazyColumn for smooth scrolling and proper state management
```

---

## Usage Instructions for OpenHands

### Execution Order
Execute the prompts in the suggested order to maintain proper dependencies:

1. **Phase 1-2**: Setup dependencies and data models foundation
2. **Phase 3-4**: Implement repository layer and server APIs  
3. **Phase 5**: Add AI scoring algorithms
4. **Phase 6-7**: Build UI components and platform-specific features
5. **Phase 8**: Add state management with ViewModels
6. **Phase 9**: Implement comprehensive testing
7. **Phase 10**: Add advanced features

### Best Practices
- **Commit frequently**: After each prompt completion, commit your changes
- **Test incrementally**: Run tests after implementing each component
- **Verify builds**: Ensure all targets (Android, iOS, Desktop, Server) build successfully
- **Check dependencies**: Verify new dependencies are properly added to all required modules

### Example Conversation Flow
```
You: "Update gradle/libs.versions.toml to add PTE-specific dependencies..."
[OpenHands completes the task]

You: "Create shared/src/commonMain/kotlin/org/example/project/models/User.kt with..."
[OpenHands completes the task]

You: [Continue with subsequent prompts]
```

### Troubleshooting Tips
- If a prompt seems too complex, break it down into smaller, more focused tasks
- Provide additional context about your specific requirements when needed
- Reference existing files in your project when asking for modifications
- Use the exact file paths shown in your project structure

### Project-Specific Notes
- Your project uses `org.example.project` package structure - all prompts are tailored accordingly
- The server module is already configured with Ktor - build upon that foundation
- ComposeApp targets multiple platforms - ensure common code goes in shared module
- Use your existing libs.versions.toml pattern for dependency management
