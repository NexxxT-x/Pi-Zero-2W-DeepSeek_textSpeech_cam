#!/bin/bash

# Raspberry Pi Zero W Camera+Voice+AI Integration Script
# Requires: arecord, espeak, curl, jq, raspistill

# Configuration
GOOGLE_API_KEY="your_google_cloud_api_key"  # Google Speech-to-Text API
DEEPSEEK_API_KEY="your_deepseek_api_key"    # DeepSeek API
IMAGE_PATH="/home/pi/capture_$(date +%s).jpg"
AUDIO_PATH="/home/pi/audio_$(date +%s).wav"

# Capture image from camera
echo "📸 Capturing image..."
raspistill -o "$IMAGE_PATH" -n -t 1
echo "✅ Image saved to: $IMAGE_PATH"

# Audio recording setup
echo "🎤 Recording audio for 5 seconds (press Ctrl+C to stop early)..."
arecord -D plughw:1,0 -d 5 -f S16_LE -r 16000 -c 1 "$AUDIO_PATH"

# Convert speech to text using Google Cloud API
echo "🔍 Converting speech to text..."
TRANSCRIPT=$(curl -s -X POST \
  --data-binary @"$AUDIO_PATH" \
  --header "Content-Type: audio/l16; rate=16000; channels=1;" \
  "https://speech.googleapis.com/v1/speech:recognize?key=$GOOGLE_API_KEY" \
  | jq -r '.results[0].alternatives[0].transcript')

[ -z "$TRANSCRIPT" ] && TRANSCRIPT="No speech detected"
echo "💬 You said: $TRANSCRIPT"

# Process with DeepSeek API
echo "🧠 Processing with DeepSeek..."
DEEPSEEK_RESPONSE=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $DEEPSEEK_API_KEY" \
  -d "$(jq -n --arg prompt "$TRANSCRIPT" '{
    model: "deepseek-chat",
    messages: [{
      role: "user",
      content: $prompt
    }]
  }')" \
  "https://api.deepseek.com/v1/chat/completions")

# Extract and process response
RESPONSE_TEXT=$(echo "$DEEPSEEK_RESPONSE" | jq -r '.choices[0].message.content')
echo "🤖 DeepSeek Response: $RESPONSE_TEXT"

# Text-to-speech output
echo "🔊 Speaking response..."
espeak -v en-us+m7 -s 150 --stdout "$RESPONSE_TEXT" | aplay -q
