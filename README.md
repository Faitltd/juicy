# Houzz to Zoho Integration

A comprehensive solution for automatically converting Houzz PDF estimates into Zoho Books estimates.

## Features

- **Automated PDF Processing**: Monitors a Google Drive folder for new PDF estimates
- **Intelligent PDF Parsing**: Extracts line items, prices, and descriptions from Houzz PDFs
- **OCR Capabilities**: Uses Google Cloud Vision for scanned PDFs without embedded text
- **Fallback Mechanism**: Multiple parsing methods ensure reliable extraction
- **Zoho Books Integration**: Automatically creates estimates in Zoho Books
- **Gmail Integration**: Optional component to save PDF attachments from Gmail to Google Drive
- **Cloud Scheduler**: Automated execution at regular intervals
- **Duplicate Prevention**: Prevents creating duplicate estimates

## Architecture

The integration consists of several components:

1. **PDF Processing Service**: A Node.js application with Python modules for PDF parsing
2. **Google Drive Integration**: Monitors a folder for new PDFs and moves processed files
3. **Zoho Books API Client**: Creates estimates in Zoho Books
4. **Cloud Scheduler**: Triggers the service at regular intervals
5. **Gmail Integration**: Optional component to save PDF attachments to Google Drive

## Setup and Deployment

### Prerequisites

- Google Cloud account with billing enabled
- Zoho Books account with API access
- Google Drive folders for monitoring and processed files
- Gmail account for receiving PDF estimates (optional)

### Deployment Options

#### Option 1: Cloud Run Deployment (Recommended)

For a fully automated, serverless deployment, follow the instructions in [cloud-run-deployment.md](cloud-run-deployment.md).

This option provides:
- Serverless execution
- Automatic scaling
- Scheduled processing
- No infrastructure management

#### Option 2: Manual Script Execution

For testing or development purposes, you can run the scripts locally:

1. Install dependencies:
   ```bash
   pip install pdfplumber google-api-python-client google-auth requests pdf2image google-cloud-vision
   ```

2. Configure the scripts as described in the sections below.

3. Run the scripts manually as needed.

### Gmail to Drive Setup

To automatically save PDF attachments from Gmail to Google Drive:

1. Create or use an existing Gmail account (e.g., sendtozohobooks@gmail.com)
2. Enable the Gmail and Drive APIs in Google Cloud Console
3. Create OAuth credentials for a desktop application
4. Save the credentials as `gmail_credentials.json`
5. Configure `email_to_drive.py` with your Drive folder ID
6. Run the script to authorize access and process emails

```bash
python houzz_to_zoho/email_to_drive.py
```

The script will:
- Find unread emails with PDF attachments
- Save the attachments to your Google Drive folder
- Mark the emails as read

### PDF Processing Configuration

Configure the PDF processing by setting the following environment variables:

```
ZOHO_CLIENT_ID=your_zoho_client_id
ZOHO_CLIENT_SECRET=your_zoho_client_secret
ZOHO_REFRESH_TOKEN=your_zoho_refresh_token
ZOHO_ORGANIZATION_ID=your_zoho_org_id
ZOHO_CUSTOMER_ID=your_zoho_customer_id
DRIVE_FOLDER_ID=your_drive_folder_id
PROCESSED_FOLDER_ID=your_processed_folder_id
ENABLE_EMAIL_NOTIFICATIONS=false
ZOHO_MOCK_MODE=true  # Set to false for production
```

## Testing

### Manual Testing

1. Upload a test PDF to the monitored Google Drive folder
2. Trigger the processing:
   - For Cloud Run: `curl -X GET https://houzz-to-zoho-YOUR_PROJECT_ID.us-central1.run.app/process-drive`
   - For local scripts: `python houzz_to_zoho/sync_estimates.py`
3. Check the logs to verify processing
4. Verify the PDF was moved to the processed folder
5. Check Zoho Books for the new estimate (if mock mode is disabled)

### Email Testing

1. Send a test PDF to your Gmail account (e.g., sendtozohobooks@gmail.com)
2. Run the email processing script or wait for it to run automatically
3. Verify the PDF appears in your Google Drive folder
4. Trigger the PDF processing as described above

## Advanced Features

### OCR Capabilities

The integration includes OCR capabilities using Google Cloud Vision API, which:
- Converts PDF pages to images
- Sends images to Google Cloud Vision API for text extraction
- Processes the extracted text to identify line items and prices
- Works with scanned PDFs that don't have embedded text

### Fallback Mechanism

The service implements a robust fallback mechanism:
1. First attempts to parse the PDF using pdfplumber
2. If that fails, falls back to OCR using Google Cloud Vision
3. If both methods fail, logs an error and moves the file to the processed folder

This ensures maximum reliability in processing various PDF formats.

## Troubleshooting

### Common Issues

1. **PDF Parsing Failures**: Some PDF formats may not be parsable. Try converting the PDF to a different format.
2. **Authentication Issues**: Ensure your service account has the necessary permissions.
3. **Zoho API Rate Limits**: If processing many PDFs, you might hit Zoho API rate limits.

### Checking Logs

For Cloud Run deployments, check the logs:
```bash
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=houzz-to-zoho" --limit=50
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- [pdfplumber](https://github.com/jsvine/pdfplumber)
- [Google Cloud Vision API](https://cloud.google.com/vision)
- [Zoho Books API](https://www.zoho.com/books/api/v3/)
# juicy
