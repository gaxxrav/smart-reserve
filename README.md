# SmartReserve - AI Voice Assistant for Restaurant Bookings

<img src="https://github.com/gaxxrav/smart-reserve/blob/main/flowchart.png" alt="Architecture diagram" width="700">

SmartReserve is an AI-powered voice assistant that automatically handles restaurant phone calls and manages bookings. It integrates Twilio for voice processing, Dialogflow for intent recognition, and Google Gemini for natural conversation generation.

## 🚀 Features

- **Voice Call Handling**: Automatically answers restaurant calls via Twilio
- **Intent Recognition**: Uses Google Dialogflow to understand customer requests
- **Natural Responses**: Generates human-like responses using Google Gemini
- **Booking Management**: Stores and manages restaurant reservations in SQLite
- **Multi-Intent Support**: Handles bookings, cancellations, hours, and location inquiries

## 🏗️ Architecture

```
Incoming Call (Twilio)
   ↓
Speech-to-Text (Twilio STT)
   ↓
FastAPI /twilio/voice endpoint
   ↓
Dialogflow → detect intent + entities
   ↓
Gemini → generate natural response
   ↓
FastAPI → save booking if needed
   ↓
Text-to-Speech (Twilio TTS)
   ↓
User hears the reply
```

## 📋 Prerequisites

Before running SmartReserve, you'll need:

1. **Twilio Account**: For voice call handling
2. **Google Cloud Account**: For Dialogflow CX
3. **Google AI Studio Account**: For Gemini API access
4. **Python 3.8+**: For running the application

## 🛠️ Setup Instructions

### 1. Clone and Install Dependencies

```bash
# Clone the repository
git clone <your-repo-url>
cd chatbot

# Install Python dependencies
pip install -r requirements.txt
```

### 2. Environment Configuration

Copy the example environment file and fill in your API keys:

```bash
cp env.example .env
```

Edit `.env` with your actual API credentials:

```env
# Twilio Configuration
TWILIO_AUTH_TOKEN=your_twilio_auth_token_here
TWILIO_ACCOUNT_SID=your_twilio_account_sid_here
TWILIO_PHONE_NUMBER=your_twilio_phone_number_here

# Dialogflow Configuration
DIALOGFLOW_PROJECT_ID=your_dialogflow_project_id_here
DIALOGFLOW_LOCATION=us-central1
DIALOGFLOW_AGENT_ID=your_dialogflow_agent_id_here
GOOGLE_APPLICATION_CREDENTIALS=path/to/your/service-account-key.json

# Google Gemini Configuration
GEMINI_API_KEY=your_gemini_api_key_here

# Database Configuration
DATABASE_URL=sqlite:///./smartreserve.db
```

### 3. Twilio Setup

1. Create a Twilio account at [twilio.com](https://twilio.com)
2. Get a phone number with voice capabilities
3. Note your Account SID and Auth Token
4. Configure webhook URL (see Testing section)

### 4. Dialogflow Setup

1. Create a Google Cloud project
2. Enable Dialogflow CX API
3. Create a Dialogflow CX agent
4. Download service account JSON key
5. Create the following intents:

#### Intents to Create:

**BookTable**
- Training phrases: "I want to book a table", "Make a reservation", "Table for 2 at 7 PM"
- Entities: `name`, `date`, `time`, `party_size`

**CancelBooking**
- Training phrases: "Cancel my reservation", "I need to cancel"
- Entities: `name`, `date`, `time`

**CheckOpeningHours**
- Training phrases: "What are your hours?", "When are you open?"

**GetRestaurantLocation**
- Training phrases: "Where are you located?", "What's your address?"

### 5. Google Gemini Setup

1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Sign in with your Google account
3. Create a new API key
4. Copy the API key to your `.env` file

## 🚀 Running the Application

### Local Development

```bash
# Start the FastAPI server
python main.py

# Or using uvicorn directly
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

The API will be available at `http://localhost:8000`

### Testing with ngrok (for Twilio webhooks)

```bash
# Install ngrok
npm install -g ngrok

# Expose your local server
ngrok http 8000

# Use the ngrok URL as your Twilio webhook
# Example: https://abc123.ngrok.io/twilio/voice
```

## 📞 Testing the Voice Assistant

### 1. Configure Twilio Webhook

In your Twilio console:
1. Go to Phone Numbers → Manage → Active Numbers
2. Click on your phone number
3. Set the webhook URL to: `https://your-ngrok-url.ngrok.io/twilio/voice`
4. Set HTTP method to POST
5. Save configuration

### 2. Test the Voice Call

1. Call your Twilio phone number
2. Say something like: "I want to book a table for 2 people at 7 PM"
3. The assistant should respond naturally and confirm your booking

### 3. Frontend Testing

**Option A: Using the Frontend Server (Recommended)**
```bash
# In one terminal, start the backend
python start.py

# In another terminal, start the frontend server
python serve_frontend.py

# The frontend will automatically open at http://localhost:5500/simple_frontend.html
```

**Option B: Manual HTTP Server**
```bash
# Start the backend
python start.py

# In another terminal, serve the frontend
python -m http.server 5500

# Open http://localhost:5500/simple_frontend.html in your browser
```

### 4. API Testing

Test individual components:

```bash
# Test Dialogflow
curl "http://localhost:8000/test/dialogflow?text=I%20want%20to%20book%20a%20table"

# Test Gemini
curl "http://localhost:8000/test/gemini"

# View all bookings
curl "http://localhost:8000/bookings"
```

## 📁 Project Structure

```
chatbot/
├── main.py                 # FastAPI application and Twilio webhook
├── db.py                   # Database operations and models
├── dialogflow_utils.py     # Dialogflow integration
├── gemini_utils.py         # Gemini response generation
├── requirements.txt        # Python dependencies
├── env.example            # Environment variables template
├── README.md              # This file
└── smartreserve.db        # SQLite database (created automatically)
```

## 🔧 API Endpoints

### Voice Endpoints
- `POST /twilio/voice` - Main webhook for Twilio voice calls

### Management Endpoints
- `GET /` - Health check
- `GET /health` - Detailed health status
- `GET /bookings` - List all bookings
- `POST /bookings` - Create a new booking

### Testing Endpoints
- `GET /test/dialogflow` - Test Dialogflow integration
- `GET /test/gemini` - Test Gemini integration

## 🎯 Supported Intents

1. **BookTable**: Make restaurant reservations
   - Extracts: name, date, time, party size
   - Saves to database
   - Confirms booking details

2. **CancelBooking**: Cancel existing reservations
   - Extracts: name, date, time
   - Processes cancellation

3. **CheckOpeningHours**: Get restaurant hours
   - Returns: "Monday through Sunday, 11 AM to 10 PM"

4. **GetRestaurantLocation**: Get restaurant address
   - Returns: "123 Main Street, Downtown"

## 🚀 Deployment

### Using Render

1. Connect your GitHub repository to Render
2. Set environment variables in Render dashboard
3. Deploy as a web service
4. Update Twilio webhook URL to your Render URL

### Using Railway

1. Connect your GitHub repository to Railway
2. Set environment variables
3. Deploy automatically
4. Update Twilio webhook URL

### Database Migration

For production, consider migrating from SQLite to PostgreSQL:

```python
# Update DATABASE_URL in .env
DATABASE_URL=postgresql://user:password@host:port/database
```

## 🐛 Troubleshooting

### Common Issues

1. **Dialogflow Authentication Error**
   - Ensure service account JSON is properly configured
   - Check GOOGLE_APPLICATION_CREDENTIALS path

2. **Gemini API Error**
   - Verify API key is correct
   - Check Google AI Studio account has API access

3. **Twilio Webhook Not Working**
   - Ensure ngrok is running
   - Check webhook URL is accessible
   - Verify HTTP method is POST

4. **Database Errors**
   - Check file permissions for SQLite database
   - Ensure database directory is writable

### Debug Mode

Enable debug logging:

```bash
export PYTHONPATH=.
python -c "import logging; logging.basicConfig(level=logging.DEBUG)"
python main.py
```

## 📝 Example Conversations

### Booking a Table
**Customer**: "Hi, I'd like to book a table for 2 people at 7 PM tomorrow"
**Assistant**: "Perfect! I've confirmed your reservation for party of 2 on [date] at 7:00 PM. Your confirmation number is 123. We look forward to seeing you!"

### Checking Hours
**Customer**: "What are your opening hours?"
**Assistant**: "We're open Monday through Sunday from 11 AM to 10 PM. We're closed on major holidays. Is there anything else I can help you with?"

### Getting Location
**Customer**: "Where are you located?"
**Assistant**: "We're located at 123 Main Street in Downtown. We're right next to the city park, with plenty of street parking available. Would you like directions or any other information?"

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🆘 Support

For support and questions:
- Create an issue in the GitHub repository
- Check the troubleshooting section above
- Review the API documentation at `/docs` when running locally

---

**SmartReserve** - Making restaurant reservations smarter, one call at a time! 🍽️🤖
# smart-reserve
