# Getting Started with Questbee

Welcome to Questbee! This guide will walk you through setting up your first Questbee instance for development or testing.

## Prerequisites

- Docker and Docker Compose installed
- Git (to clone the repository)
- At least 2GB of RAM and 10GB disk space
- Basic command-line familiarity

## Quick Start (5 Minutes)

### 1. Clone the Repository
```bash
git clone https://github.com/your-org/questbee.git
cd questbee
```

### 2. Start the Stack
```bash
docker-compose up --build
```

This will start:
- **FastAPI Backend** (http://localhost:8000)
- **PostgreSQL Database** (port 5432, password: `postgres`)
- **Next.js Web Dashboard** (http://localhost:3000)

### 3. Access the Dashboard
Open your browser and navigate to:
```
http://localhost:3000
```

Default credentials:
- **Email:** admin@example.com
- **Password:** admin123

## Creating Your First Form

### Step 1: Log In
Use the default credentials above to log into the dashboard.

### Step 2: Create a Project
1. Click **New Project**
2. Enter a project name (e.g., "Survey Demo")
3. Click **Create**

### Step 3: Create a Form
1. Go to your Project
2. Click **New Form**
3. Fill in basic details:
   - **Form Name:** e.g., "Field Inspection Checklist"
   - **Description:** Brief description
4. Click **Create**

### Step 4: Build Your Form
1. Drag-and-drop fields from the left panel:
   - Text (short / long)
   - Number
   - Email
   - Dropdown
   - Radio Buttons
   - Checkbox
   - Date / Time
   - GPS Location
   - Photo / Video
   - Signature
   - Barcode / QR Scanner

2. Configure each field:
   - Set **Required** or **Optional**
   - Add validation rules
   - Add help text

3. Click **Publish** to finalize the form.

## Mobile App Setup

### Pairing the App with Your Server

1. **Generate a Pairing QR Code**
   - In the dashboard, go to **Settings > Mobile Pairing**
   - Click **Generate QR Code**
   - The QR code contains your server URL and a pairing token

2. **Install the Mobile App**
   - Download the Questbee app from:
     - **iOS:** (Link to TestFlight / App Store)
     - **Android:** (Link to Google Play / APK)

3. **Pair the App**
   - Open the app
   - Tap **Scan QR Code**
   - Scan the QR code from the dashboard
   - Tap **Connect**

4. **Download Forms**
   - The app will automatically download assigned forms
   - You can now fill them out offline

## Submitting Data

### Via Mobile App
1. Select a form from the **Forms** tab
2. Fill in the data (offline-capable)
3. Tap **Submit**
4. Once online, data will auto-sync to the server

### Via Web
1. In the dashboard, go to your form
2. Click **Collect Data Online**
3. Share the public link with respondents
4. Submissions will appear in real-time

## Viewing Results

1. Navigate to your form
2. Click **View Submissions** or **View Data**
3. See all submitted responses in a table format
4. **Export as CSV** for analysis in Excel or other tools

## Next Steps

- Read the [Architecture Guide](architecture.md) to understand the system design
- Check out the [API Reference](api-reference.md) for headless integrations
- Review [Deployment Guide](deployment.md) for production setups
- Explore [Partner Resources](../../partner_pilot_offer.md) if you're integrating for a client

## Troubleshooting

### Dashboard Not Loading?
```bash
# Check Docker logs
docker-compose logs web
```

### Database Connection Error?
```bash
# Verify PostgreSQL is running
docker-compose ps

# Restart the database
docker-compose restart db
```

### Mobile App Can't Sync?
- Ensure the server URL is correct (check the pairing QR code)
- Verify network connectivity
- Check Docker logs for API errors: `docker-compose logs api`

## Support

For more help:
- Check the [GitHub Issues](https://github.com/your-org/questbee/issues)
- Post on GitHub Discussions
- Contact partnerships@questbee.io for enterprise support

---

Happy form building! 🎉
