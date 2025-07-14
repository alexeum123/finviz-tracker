# Free  Tampermonkey Script - Technical Documentation

## Overview

The Free  Tampermonkey script is a browser-based AI-powered screenshot analyzer that helps with coding interviews, problem-solving, and technical analysis. It creates a draggable overlay interface that can capture screenshots of any webpage and analyze them using Amazon Bedrock's Claude AI models.

## How It Works

### 1. Core Architecture

The script follows a modular architecture with several key components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   UI Overlay    │    │  Screenshot     │    │   AI Analysis   │
│   (Draggable)   │◄──►│   Service       │◄──►│   (Bedrock)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  App State      │    │  Canvas/Video   │    │  AWS Signature  │
│  Management     │    │  Processing     │    │  V4 Auth        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2. Image Processing Flow

#### Step 1: Screen Capture
```javascript
// Uses modern Web APIs to capture screen content
navigator.mediaDevices.getDisplayMedia({
    video: { mediaSource: 'screen', width: 1920, height: 1080 },
    audio: false
})
```

**Process:**
1. **Permission Request**: Browser asks user to select screen/window to capture
2. **Media Stream**: Creates a video stream of the selected content
3. **Video Element**: Temporarily creates a hidden `<video>` element
4. **Canvas Rendering**: Draws the video frame onto a `<canvas>` element
5. **Image Conversion**: Converts canvas to JPEG blob with configurable quality

#### Step 2: Base64 Encoding
```javascript
// Convert blob to base64 for storage and transmission
const reader = new FileReader();
reader.onloadend = () => {
    const base64 = reader.result.split(',')[1]; // Remove data URL prefix
    // Store both full data URL (for preview) and base64 (for API)
};
reader.readAsDataURL(blob);
```

#### Step 3: AI Analysis with Amazon Bedrock

**Authentication Process:**
The script implements AWS Signature Version 4 authentication:

```javascript
class AWSSignatureV4 {
    async sign(request, payload) {
        // 1. Create canonical request
        // 2. Generate string to sign
        // 3. Calculate HMAC-SHA256 signature
        // 4. Build authorization header
    }
}
```

**API Request Format:**
```javascript
const requestBody = {
    anthropic_version: "bedrock-2023-05-31",
    max_tokens: 4000,
    system: systemPrompt,
    messages: [{
        role: "user",
        content: [
            { type: "text", text: "Please analyze this screenshot..." },
            {
                type: "image",
                source: {
                    type: "base64",
                    media_type: "image/jpeg",
                    data: screenshot.data // Base64 image data
                }
            }
        ]
    }]
};
```

### 3. User Interface Components

#### Overlay Structure
```html
<div id="free--overlay">
    <div class="fc-header">        <!-- Draggable title bar -->
    <div class="fc-content">       <!-- Main content area -->
        <div class="fc-tabs">      <!-- Queue/Solutions tabs -->
        <div class="fc-view">      <!-- Current view content -->
    <div class="fc-status">        <!-- Status messages -->
</div>
```

#### State Management
```javascript
class AppState {
    constructor() {
        this.isVisible = false;
        this.currentView = 'queue';     // 'queue' or 'solutions'
        this.screenshots = [];          // Array of captured screenshots
        this.currentSolution = '';      // Latest AI analysis
        this.isProcessing = false;      // Loading state
        this.position = { x: 50, y: 50 }; // Overlay position
    }
}
```

## Key Features

### 1. Screenshot Management
- **Capture**: Uses `getDisplayMedia()` API for high-quality screen capture
- **Storage**: Maintains up to 10 screenshots in memory (configurable)
- **Preview**: Shows thumbnail previews with timestamps
- **Deletion**: Individual screenshot removal with instant UI updates

### 2. AI Analysis
- **Model**: Uses Claude 3.5 Sonnet (configurable to other Bedrock models)
- **Context**: Specialized prompts for coding problems, debugging, and technical analysis
- **Response Format**: Markdown-formatted responses with code blocks and explanations

### 3. User Experience
- **Keyboard Shortcuts**: 
  - `Ctrl+B`: Toggle overlay visibility
  - `Ctrl+H`: Capture screenshot
  - `Ctrl+Enter`: Analyze screenshots
  - `Escape`: Hide overlay
- **Drag & Drop**: Fully draggable overlay with viewport boundaries
- **Responsive**: Adapts to different screen sizes and content

## Technical Implementation Details

### 1. Screenshot Capture Process

```javascript
async captureScreen() {
    // 1. Request screen sharing permission
    const stream = await navigator.mediaDevices.getDisplayMedia({...});
    
    // 2. Create video element
    const video = document.createElement('video');
    video.srcObject = stream;
    
    // 3. Wait for video metadata
    video.onloadedmetadata = () => {
        video.play();
        
        // 4. Create canvas and capture frame
        const canvas = document.createElement('canvas');
        canvas.width = video.videoWidth;
        canvas.height = video.videoHeight;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(video, 0, 0);
        
        // 5. Convert to blob and cleanup
        canvas.toBlob((blob) => {
            stream.getTracks().forEach(track => track.stop());
            // Process blob...
        }, 'image/jpeg', 0.8);
    };
}
```

### 2. AWS Authentication

The script implements complete AWS Signature V4 authentication without external dependencies:

```javascript
// Key steps:
1. Create canonical request (method, URI, headers, payload hash)
2. Generate string to sign (algorithm, timestamp, scope, request hash)
3. Calculate signing key (date → region → service → request)
4. Generate signature using HMAC-SHA256
5. Build authorization header
```

### 3. Error Handling

```javascript
// Comprehensive error handling at each stage:
- Permission denied for screen capture
- Network failures during API calls
- Invalid AWS credentials
- Malformed API responses
- Browser compatibility issues
```

## Configuration Options

### AWS Settings
```javascript
const CONFIG = {
    AWS_ACCESS_KEY_ID: 'your-access-key',
    AWS_SECRET_ACCESS_KEY: 'your-secret-key',
    AWS_REGION: 'us-east-1',
    BEDROCK_MODEL_ID: 'anthropic.claude-3-5-sonnet-20241022-v2:0'
};
```

### Performance Settings
```javascript
const CONFIG = {
    MAX_SCREENSHOTS: 10,           // Maximum screenshots to store
    SCREENSHOT_QUALITY: 0.8,       // JPEG quality (0.0-1.0)
    OVERLAY_Z_INDEX: 999999,       // CSS z-index for overlay
    ANIMATION_DURATION: 300        // Animation duration in ms
};
```

## Supported Bedrock Models

You can change the `BEDROCK_MODEL_ID` to use different Claude models:

| Model | ID | Best For |
|-------|----|---------| 
| Claude 3.5 Sonnet | `anthropic.claude-3-5-sonnet-20241022-v2:0` | General purpose, coding |
| Claude 3 Haiku | `anthropic.claude-3-haiku-20240307-v1:0` | Fast responses, simple tasks |
| Claude 3 Opus | `anthropic.claude-3-opus-20240229-v1:0` | Complex reasoning, detailed analysis |

## Installation & Setup

### 1. Install Tampermonkey
- Install the [Tampermonkey browser extension](https://www.tampermonkey.net/)

### 2. Add the Script
- Copy the contents of `free--tampermonkey.js`
- Open Tampermonkey dashboard
- Click "Create a new script"
- Paste the code and save

### 3. Configure AWS Credentials
```javascript
// Update these values in the script:
AWS_ACCESS_KEY_ID: 'YOUR_AWS_ACCESS_KEY_ID_HERE',
AWS_SECRET_ACCESS_KEY: 'YOUR_AWS_SECRET_ACCESS_KEY_HERE',
AWS_REGION: 'us-east-1' // Your preferred region
```

### 4. AWS Bedrock Setup
1. **Enable Bedrock**: Go to AWS Console → Bedrock → Model access
2. **Request Access**: Enable access to Anthropic Claude models
3. **Create IAM User**: Create user with `bedrock:InvokeModel` permission
4. **Generate Keys**: Create access key/secret for the IAM user

## Usage Examples

### 1. Coding Interview Problem
1. Navigate to coding platform (LeetCode, HackerRank, etc.)
2. Press `Ctrl+B` to show overlay
3. Press `Ctrl+H` to capture the problem statement
4. Press `Ctrl+Enter` to get AI analysis
5. Receive detailed solution with complexity analysis

### 2. Debug Code Issues
1. Capture screenshot of error messages or problematic code
2. AI analyzes the issue and provides:
   - Bug identification
   - Explanation of the problem
   - Corrected code
   - Improvement suggestions

### 3. Technical Documentation
1. Capture screenshots of technical diagrams or documentation
2. Get AI-generated summaries and explanations
3. Receive actionable next steps

## Security Considerations

### 1. Credential Storage
- AWS credentials are stored in the script (not ideal for production)
- Consider using temporary credentials or IAM roles when possible
- Never commit credentials to version control

### 2. Data Privacy
- Screenshots are processed locally and sent to AWS Bedrock
- No data is stored on external servers beyond AWS
- Images are not persisted after analysis

### 3. Permissions
- Script requires screen capture permissions
- Only works on HTTPS sites (due to `getDisplayMedia` requirements)
- Limited by browser security policies

## Troubleshooting

### Common Issues

1. **"Permission denied" for screen capture**
   - Ensure you're on an HTTPS site
   - Check browser permissions for screen sharing
   - Try refreshing the page

2. **AWS authentication errors**
   - Verify credentials are correct
   - Check IAM permissions for Bedrock access
   - Ensure model access is enabled in Bedrock console

3. **Overlay not appearing**
   - Check browser console for errors
   - Verify Tampermonkey is enabled
   - Try different websites

### Debug Mode
Enable debug logging by adding to the script:
```javascript
const DEBUG = true;
if (DEBUG) console.log('Debug message');
```

## Performance Optimization

### 1. Image Quality
- Lower `SCREENSHOT_QUALITY` for faster processing
- Reduce `MAX_SCREENSHOTS` for memory efficiency

### 2. API Calls
- Batch multiple screenshots for single analysis
- Implement request caching for similar images
- Use faster models (Haiku) for simple tasks

### 3. UI Responsiveness
- Async/await for all API calls
- Loading states during processing
- Debounced user interactions

## Browser Compatibility

| Browser | Support | Notes |
|---------|---------|-------|
| Chrome | ✅ Full | Best performance |
| Firefox | ✅ Full | Good performance |
| Safari | ⚠️ Limited | Some API limitations |
| Edge | ✅ Full | Chromium-based |

## Future Enhancements

- **Multi-model Support**: Switch between different AI providers
- **Batch Processing**: Analyze multiple screenshots simultaneously
- **Export Options**: Save analyses as PDF or markdown
- **Custom Prompts**: User-defined analysis prompts
- **OCR Integration**: Extract text before AI analysis
- **History**: Persistent storage of past analyses

## Contributing

To modify or extend the script:
1. Update the configuration section for new features
2. Add new UI components to the `UIComponents` class
3. Extend the `AIService` for different AI providers
4. Test across different browsers and websites

## License

This script is provided as-is for educational and personal use. Please ensure compliance with your organization's policies and AWS terms of service when using in professional environments. 