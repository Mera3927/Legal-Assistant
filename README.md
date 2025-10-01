# Legal Document Analysis & Chat Workflow - README

## Overview

This n8n workflow provides an intelligent legal and tender document analysis system that:
- Monitors a Google Drive folder for new or updated legal documents
- Processes and indexes documents (DOCX, PDF, Excel) into a Pinecone vector database
- Provides a chat interface for querying document content using AI
- Generates structured legal analysis reports based on document content
- Includes human-in-the-loop approval for AI responses
- Exports final reports as PDF

**Use Cases:**
- Tender document analysis
- Contract review and summarization
- Legal compliance checking
- Regulatory document analysis
- Due diligence document processing

## Architecture

**Key Components:**
1. **Document Ingestion**: Google Drive triggers automatically detect new/updated files
2. **Vector Storage**: Pinecone stores document embeddings for semantic search
3. **AI Analysis**: Google Gemini models analyze documents and generate responses
4. **Quality Control**: Automated proofreading and human approval workflow
5. **Output Generation**: Converts responses to HTML/PDF format

---

## Prerequisites

- Node.js (v18 or higher)
- npm or yarn
- Git

---

## Part 1: Setting Up n8n Locally

### Install n8n

```bash
# Install n8n globally
npm install n8n -g
```

### Run n8n Locally

#### Windows (PowerShell)

```powershell
$env:N8N_CORS_ORIGIN="*"; $env:N8N_CORS_CREDENTIALS="true"; $env:NODE_FUNCTION_ALLOW_EXTERNAL="*"; n8n start
```

#### Linux/Mac (Terminal)

```bash
N8N_CORS_ORIGIN="*" N8N_CORS_CREDENTIALS="true" NODE_FUNCTION_ALLOW_EXTERNAL="*" n8n start
```

### Environment Variables Explained

- **`N8N_CORS_ORIGIN="*"`**: Allows the chat interface to be accessed from any origin (required for webhook chat)
- **`N8N_CORS_CREDENTIALS="true"`**: Enables credentials in CORS requests
- **`NODE_FUNCTION_ALLOW_EXTERNAL="*"`**: Allows the Code nodes to use external npm packages (required for Excel/PDF processing with libraries like `xlsx` and `pdfkit`)

**Security Note**: The `*` wildcard is convenient for local development. For production, replace with specific domains:

```powershell
# Production example
$env:N8N_CORS_ORIGIN="https://yourdomain.com"
```

### Verify n8n is Running

Open your browser and navigate to: **http://localhost:5678**

You should see the n8n interface.

---

## Part 2: Obtaining API Keys

### 1. Google Gemini API Key

**Steps:**
1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Click "Get API Key" or "Create API Key"
4. Copy the generated API key
5. **Pricing**: Free tier available (60 requests/minute)

### 2. Pinecone API Key

**Steps:**
1. Sign up at [Pinecone](https://www.pinecone.io/)
2. Verify your email and log in
3. Go to "API Keys" in the left sidebar
4. Click "Create API Key"
5. Copy both the **API Key** and **Environment** value
6. Create an index named `tender-test`:
   - Click "Create Index"
   - Name: `tender-test`
   - Dimensions: **768** (for Google Gemini embeddings)
   - Metric: **Cosine**
   - Pod Type: Starter (free tier)

**Note**: Free tier includes 1 pod and 100K vectors.

### 3. PDFShift API Key

**Steps:**
1. Sign up at [PDFShift](https://pdfshift.io/)
2. Go to your [Dashboard](https://pdfshift.io/dashboard)
3. Copy your API key
4. **Pricing**: 250 free conversions, then paid plans

### 4. Google Drive & Gmail OAuth

**Steps:**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable APIs:
   - Google Drive API
   - Gmail API
4. Go to "Credentials" → "Create Credentials" → "OAuth client ID"
5. Configure OAuth consent screen if prompted:
   - User Type: External
   - Add your email as a test user
6. Choose "Web application" as application type
7. Add authorized redirect URI:
   ```
   http://localhost:5678/rest/oauth2-credential/callback
   ```
8. Copy **Client ID** and **Client Secret**

---

## Part 3: Importing the Workflow

### Import Steps

1. **Open n8n**: Navigate to `http://localhost:5678`
2. **Create new workflow**: Click "+ New workflow" or "Add workflow"
3. **Import JSON**:
   - Click the three dots (⋮) in top right corner
   - Select "Import from File"
   - Upload the `My workflow 2 (4).json` file
4. Click "Save" and give it a name

---

## Part 4: Configuring Credentials

### Configure Google Gemini (3 instances)

You'll need to configure Google Gemini credentials for multiple nodes:

1. Click on any "Google Gemini Chat Model" node
2. Click "Create New Credential" next to the credentials field
3. Enter:
   - **API Key**: Your Google Gemini API key
4. Save and name it (e.g., "Google Gemini API")
5. Repeat for other Gemini nodes or select the same credential

**Nodes using Gemini:**
- Google Gemini Chat Model (AI Generative Agent)
- Google Gemini Chat Model1 (AI Edit Agent)
- Google Gemini Chat Model 2
- Google Gemini Chat Model (unnamed)
- Embeddings Google Gemini (multiple instances)

### Configure Pinecone (2 instances)

1. Click on "Pinecone Vector Store" node
2. Click "Create New Credential"
3. Enter:
   - **API Key**: Your Pinecone API key
   - **Environment**: Your Pinecone environment (e.g., `us-east-1-aws`)
4. Save
5. Select the same credential for all Pinecone nodes

**Nodes using Pinecone:**
- Pinecone Vector Store (ingestion)
- Pinecone Vector Store (Retrieval)
- Pinecone Vector Store (Retrieval)1

### Configure PDFShift

1. Click on the "pdf generator" node
2. Click on the credential field
3. Select "HTTP Basic Auth"
4. Click "Create New Credential"
5. Enter:
   - **Username**: `api`
   - **Password**: Your PDFShift API key
6. Save

### Configure Google Drive OAuth

1. Click on "Google Drive File Created" node
2. Click "Create New Credential" for Google Drive OAuth2 API
3. Enter:
   - **Client ID**: From Google Cloud Console
   - **Client Secret**: From Google Cloud Console
   - **Auth URI**: `https://accounts.google.com/o/oauth2/auth`
   - **Token URI**: `https://oauth2.googleapis.com/token`
   - **Scopes**: `https://www.googleapis.com/auth/drive`
4. Click "Connect"
5. Authorize the application in the popup
6. Save
7. Use the same credential for:
   - Google Drive File Updated
   - Download File From Google Drive

### Configure Gmail OAuth

1. Click on "Mail sender (Human in the Middle)" node
2. Click "Create New Credential" for Gmail OAuth2
3. Enter:
   - **Client ID**: Same as Google Drive
   - **Client Secret**: Same as Google Drive
4. Click "Connect"
5. Authorize Gmail access in the popup
6. Save

---

## Part 5: Workflow Configuration

### Update Google Drive Folder

1. Click on "Google Drive File Created" node
2. In "Folder to Watch" parameter:
   - Click on the dropdown
   - Select "From List" and choose your folder
   - OR paste your folder ID directly
3. Repeat for "Google Drive File Updated" node

**To get Folder ID:**
- Open Google Drive in browser
- Navigate to your folder
- Copy the ID from the URL: `https://drive.google.com/drive/folders/[FOLDER_ID]`

### Update Email Address for Approvals

1. Click on "Mail sender (Human in the Middle)" node
2. Change the "Send To" field from `mera3927@gmail.com` to **your email address**
3. Save

### Verify Pinecone Index Name

1. Ensure your Pinecone index is named exactly: `tender-test`
2. If you used a different name, update it in these nodes:
   - Pinecone Vector Store
   - Pinecone Vector Store (Retrieval)
   - Pinecone Vector Store (Retrieval)1

---

## Part 6: How to Use the Workflow

### Phase 1: Document Setup & Indexing

1. **Activate the workflow**: Toggle the "Active" switch in the top right corner
2. **Upload documents** to your monitored Google Drive folder:
   - Supported formats: PDF, DOCX, XLSX, XLS
   - Documents will be automatically detected
3. **Automatic processing**:
   - Files are downloaded
   - Excel files are converted to PDF
   - All documents are split into chunks
   - Embeddings are created and stored in Pinecone
4. **Wait for indexing**: Check execution logs to confirm successful processing

### Phase 2: Querying Documents via Chat

1. **Access the chat interface**:
   - Click on "When chat message received" node
   - Look for the "Webhook URL" in the node details
   - Copy the URL (format: `http://localhost:5678/webhook/[unique-id]`)
   - Open this URL in your browser

2. **Ask questions about your legal documents**, for example:
   - "What are the key stakeholders in this tender?"
   - "What are the submission deadlines?"
   - "Summarize the financial requirements"
   - "What compliance standards are mentioned?"
   - "What are the risks and challenges identified?"
   - "List all the requirements and specifications"

3. **AI Processing**:
   - The system searches the vector database
   - Retrieves relevant document sections
   - Generates structured responses with 8 sections:
     - Project Overview
     - Key Stakeholders
     - Timeline and Deadlines
     - Financial Information
     - Requirements and Specifications
     - Risks and Challenges
     - Recommendations and Actions
     - Compliance and Standards

### Phase 3: Quality Control & Approval

1. **Automated Quality Check**:
   - Response is converted to Markdown
   - Automated proofreader validates:
     - All 8 sections are present
     - Content is sufficient (not placeholder text)
     - Proper formatting
     - No repetitive content
   - Maximum 3 iterations of quality checking

2. **Human Approval Workflow**:
   - If quality check passes: Response goes directly to PDF generation
   - If quality check fails after 3 iterations: You receive an email

3. **Email Approval**:
   - Check your email for "Approval Required for AI response"
   - Review the HTML-formatted response
   - Choose:
     - **Approve**: Proceeds to PDF generation
     - **Reject**: Provide feedback in "Reasons" field

4. **AI Revision** (if rejected):
   - Your feedback is sent to the AI Edit Agent
   - AI revises the response using your guidance
   - Response goes through quality check again
   - You may receive another approval email if needed

### Phase 4: Output Generation

1. **PDF Creation**:
   - Approved responses are formatted as HTML
   - Converted to PDF using PDFShift API
   - Timestamp and unique ID are added to filename

2. **File Storage**:
   - PDFs are saved to `/out/` directory in your n8n installation folder
   - Filename format: `pdf_YYYY-MM-DDTHH-MM-SS_[uniqueid].pdf`

3. **Accessing PDFs**:
   - Navigate to your n8n installation directory
   - Find the `/out/` folder
   - Open or download the generated PDF

---

## Workflow Execution Flow

```
┌─────────────────────────────────────────┐
│  Document Upload (Google Drive)         │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Download File                          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Excel to PDF Conversion (if needed)    │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Text Splitting & Embedding             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Store in Pinecone Vector Database      │
└─────────────────────────────────────────┘

        [Document indexed and ready]

┌─────────────────────────────────────────┐
│  User asks question via Chat            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  AI Generative Agent                    │
│  - Retrieves from Pinecone              │
│  - Analyzes documents                   │
│  - Generates structured JSON            │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  JSON to Markdown Converter             │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Automated Proofreader                  │
│  (Max 3 iterations)                     │
└──────────────┬──────────────────────────┘
               │
         ┌─────┴─────┐
         │           │
      Passed      Failed
         │           │
         ▼           ▼
   ┌─────────┐  ┌──────────────┐
   │ Convert │  │ Human Email  │
   │ to HTML │  │ Approval     │
   └────┬────┘  └──────┬───────┘
        │              │
        │         ┌────┴────┐
        │         │         │
        │      Approve   Reject
        │         │         │
        │         │         ▼
        │         │    ┌─────────────┐
        │         │    │ AI Edit     │
        │         │    │ Agent       │
        │         │    └─────┬───────┘
        │         │          │
        │         │     (Loop back)
        │         │
        └─────────┴──────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  PDF Generation (PDFShift)              │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Save to /out/pdf_[timestamp].pdf       │
└─────────────────────────────────────────┘
```

---

## Output Format

### Structured Response Sections

Each AI-generated response includes:

1. **Project Overview**: Overall project description, objectives, and scope
2. **Key Stakeholders**: Important people, organizations, or roles
3. **Timeline and Deadlines**: Important dates, milestones, and schedules
4. **Financial Information**: Budget, costs, and financial figures
5. **Requirements and Specifications**: Technical requirements and criteria
6. **Risks and Challenges**: Identified risks and concerns
7. **Recommendations and Actions**: Recommended next steps
8. **Compliance and Standards**: Regulatory requirements and standards

---

## Troubleshooting

### Installation & Setup Issues

**Issue**: `n8n: command not found`
- **Solution**: Reinstall n8n globally: `npm install n8n -g`
- Check PATH: `npm config get prefix` and ensure it's in your PATH

**Issue**: Port 5678 already in use
- **Solution**: Kill the process using port 5678 or change the port:
  ```powershell
  $env:N8N_PORT="5679"; n8n start
  ```

### Authentication Issues

**Issue**: "Pinecone index not found"
- **Solution**: 
  - Verify index name is exactly `tender-test`
  - Check API key is correct
  - Ensure index dimensions are 768

**Issue**: "Google Drive authentication failed"
- **Solution**: 
  - Verify redirect URI in Google Cloud Console
  - Ensure APIs are enabled
  - Check if you're logged in with the correct Google account
  - Add your email as a test user in OAuth consent screen

**Issue**: "Gmail won't authorize"
- **Solution**: 
  - Use the same OAuth credentials as Google Drive
  - Add Gmail API to your Google Cloud project
  - Check OAuth consent screen is configured

### Workflow Execution Issues

**Issue**: Documents not being processed
- **Solution**: 
  - Check workflow is activated (toggle in top right)
  - Verify Google Drive folder ID is correct
  - Check execution logs for errors
  - Ensure file formats are supported (PDF, DOCX, XLSX)

**Issue**: Chat webhook not responding
- **Solution**: 
  - Ensure workflow is active
  - Check webhook URL is correct
  - Verify CORS settings are applied
  - Try accessing from incognito mode

**Issue**: "NODE_FUNCTION_ALLOW_EXTERNAL" error in Code nodes
- **Solution**: 
  - Restart n8n with the environment variable
  - Check if environment variable is set: `echo $env:NODE_FUNCTION_ALLOW_EXTERNAL`

**Issue**: PDF generation fails
- **Solution**: 
  - Check PDFShift API key is correct
  - Verify account has available conversions
  - Check if HTML content is valid
  - Review error logs in the node

**Issue**: Email not received for approval
- **Solution**: 
  - Check email address in "Mail sender" node
  - Verify Gmail OAuth is authorized
  - Check spam/junk folder
  - Review execution log for errors

### Quality Check Issues

**Issue**: Response stuck in revision loop
- **Solution**: 
  - After 3 iterations, system forces approval
  - Check email for manual review
  - Review proofreader logs for specific issues

**Issue**: Sections missing from response
- **Solution**: 
  - Ensure documents contain relevant information
  - Try rephrasing your question
  - Check if documents were properly indexed
  - Review Pinecone retrieval results

### Vector Database Issues

**Issue**: Search returns no results
- **Solution**: 
  - Wait for documents to finish indexing
  - Check Pinecone dashboard for vector count
  - Verify embedding model is consistent (Google Gemini 768 dimensions)
  - Try more specific queries

**Issue**: Out of memory errors
- **Solution**: 
  - Process fewer/smaller documents at once
  - Increase Node.js memory: `node --max-old-space-size=4096`

---

## File Storage

### Generated PDF Location

PDFs are saved to:
```
[n8n installation directory]/out/pdf_[timestamp]_[uniqueid].pdf
```

**Example**:
```
C:/Users/YourName/.n8n/out/pdf_2025-10-01T14-30-45_a3f7d2.pdf
```

**Finding your n8n directory**:
- Windows: `C:\Users\[YourName]\.n8n\`
- Linux/Mac: `~/.n8n/`

### Storage Management

The `/out/` directory will accumulate PDFs over time. To manage:

```powershell
# Navigate to output directory
cd ~/.n8n/out

# List all PDFs
ls *.pdf

# Delete old PDFs (Windows PowerShell)
Remove-Item *.pdf -Confirm

# Delete PDFs older than 30 days (Linux/Mac)
find . -name "*.pdf" -mtime +30 -delete
```

---

## Configuration Tips

### Optimizing Performance

1. **Chunk Size**: Adjust in "Recursive Character Text Splitter" node
   - Default: 1000 characters with 100 overlap
   - Smaller chunks: Better precision, more vectors
   - Larger chunks: Less vectors, more context

2. **Temperature Settings**: Control AI creativity
   - Current: 0.1 (very focused)
   - Higher (0.7-1.0): More creative responses
   - Lower (0.0-0.3): More deterministic responses

3. **Max Tokens**: Adjust response length
   - Current: 1000 tokens
   - Increase for longer responses
   - Decrease for quicker responses

### Customizing Response Format

Edit the JSON schema in "Structured Output Parser" nodes to modify output sections:

```javascript
{
    "Project_Overview": "Description here...",
    "Custom_Section": "Add new sections...",
    // Add or remove sections as needed
}
```

### Adjusting Quality Thresholds

In "AI response manual proofreader" Code node:

```javascript
// Line ~45: Change quality score threshold
const passed = qualityScore >= 8.0; // Change 8.0 to your preference

// Line ~51: Change max iterations
const maxIterations = 3; // Change to allow more/fewer revisions
```

---

## Limitations & Notes

### Rate Limits

- **Google Gemini Free Tier**: 60 requests/minute
- **Pinecone Free Tier**: 100K vectors, 1 pod
- **PDFShift Free Tier**: 250 conversions/month

### File Constraints

- **Supported Formats**: PDF, DOCX, XLSX, XLS
- **File Size**: Recommended under 10MB per file
- **Processing Time**: Large files may take 30-60 seconds

### Workflow Constraints

- **Max Revision Iterations**: 3 automatic retries
- **Concurrent Processing**: One document at a time
- **Vector Dimensions**: Fixed at 768 (Google Gemini)

### Known Issues

1. **Excel Conversion**: Complex Excel formulas may not render correctly in PDF
2. **Large Documents**: Files over 10MB may timeout during processing
3. **Token Limits**: Very long responses may be truncated
4. **Webhook Timeout**: Chat may timeout after 2 minutes of inactivity

---

## Security Considerations

### Local Development

The current setup uses wildcard CORS (`*`) which is suitable for local development but **not for production**.

### Production Deployment

If deploying to production:

1. **Restrict CORS**:
   ```powershell
   $env:N8N_CORS_ORIGIN="https://yourdomain.com"
   ```

2. **Secure Webhooks**: Enable webhook authentication
3. **Use HTTPS**: Set up SSL certificates
4. **Environment Variables**: Use `.env` file instead of command-line
5. **API Key Management**: Use environment variables for all API keys

### Data Privacy

- Documents are stored in Pinecone (cloud)
- Ensure compliance with data regulations (GDPR, HIPAA, etc.)
- Review Pinecone's data retention policies
- Consider self-hosted vector database for sensitive legal documents

**⚠️ Important for Legal Documents**: 
- Review confidentiality requirements before uploading to cloud services
- Consider client confidentiality and attorney-client privilege
- Ensure compliance with legal professional responsibility rules

---

## Advanced Configuration

### Creating a Startup Script

#### Windows (PowerShell)

Create `start-n8n.ps1`:
```powershell
$env:N8N_CORS_ORIGIN="*"
$env:N8N_CORS_CREDENTIALS="true"
$env:NODE_FUNCTION_ALLOW_EXTERNAL="*"
$env:N8N_PORT="5678"
n8n start
```

Run: `.\start-n8n.ps1`

#### Linux/Mac (Bash)

Create `start-n8n.sh`:
```bash
#!/bin/bash
export N8N_CORS_ORIGIN="*"
export N8N_CORS_CREDENTIALS="true"
export NODE_FUNCTION_ALLOW_EXTERNAL="*"
export N8N_PORT="5678"
n8n start
```

Make executable and run:
```bash
chmod +x start-n8n.sh
./start-n8n.sh
```

### Using .env File

Create `.env` file in your project directory:
```env
N8N_CORS_ORIGIN=*
N8N_CORS_CREDENTIALS=true
NODE_FUNCTION_ALLOW_EXTERNAL=*
N8N_PORT=5678
```

Load and run:
```bash
# Linux/Mac
export $(cat .env | xargs) && n8n start

# Windows PowerShell
Get-Content .env | ForEach-Object { $var = $_.Split('='); Set-Item -Path "Env:$($var[0])" -Value $var[1] }; n8n start
```

---

## Maintenance

### Regular Tasks

1. **Monitor Pinecone Usage**: Check vector count and remaining quota
2. **Clean PDFs**: Delete old PDFs from `/out/` directory
3. **Check API Limits**: Monitor Google Gemini and PDFShift usage
4. **Update Credentials**: Refresh OAuth tokens if expired
5. **Backup Workflow**: Export workflow JSON regularly

### Updating the Workflow

1. Click the three dots (⋮) in top right
2. Select "Export"
3. Save the updated JSON file
4. Keep versioned backups

### Logs and Debugging

- **Execution Logs**: Click on workflow execution to view detailed logs
- **Node Logs**: Click individual nodes to see their output
- **Error Details**: Red nodes indicate failures - click for error messages
- **Console Logs**: Check terminal where n8n is running for system errors

---

## Credits & Dependencies

### Core Technologies

- **[n8n](https://n8n.io/)**: Open-source workflow automation platform
- **[Google Gemini](https://ai.google.dev/)**: AI language model and embeddings
- **[Pinecone](https://www.pinecone.io/)**: Vector database for semantic search
- **[PDFShift](https://pdfshift.io/)**: HTML to PDF conversion API

### Libraries Used

- **SheetJS (xlsx)**: Excel file processing
- **PDFKit**: PDF document generation
- **Papaparse**: CSV parsing
- **MathJS**: Mathematical operations
- **Lodash**: Utility functions

---

## Support & Resources

### Documentation

- **n8n Docs**: https://docs.n8n.io/
- **Google Gemini API**: https://ai.google.dev/docs
- **Pinecone Docs**: https://docs.pinecone.io/
- **PDFShift API**: https://pdfshift.io/documentation

### Community Support

- **n8n Community**: https://community.n8n.io/
- **n8n Discord**: https://discord.gg/n8n

### Getting Help

For issues with:
1. **n8n Platform**: [n8n Community Forum](https://community.n8n.io/)
2. **This Workflow**: Check the troubleshooting section above
3. **API Services**: Contact respective service support

---

## License

This workflow is provided as-is for personal and commercial use. Individual services (Google Gemini, Pinecone, PDFShift) have their own terms of service.

---

## Disclaimer

This tool is designed to assist with document analysis and should not be considered a substitute for professional legal advice. Always verify AI-generated analysis with qualified legal professionals, especially for critical legal matters.

---

## Version History

- **v1.0**: Initial release with document ingestion, chat interface, and PDF export
- Features: Google Drive integration, Pinecone vector storage, AI analysis with quality control

---

**Last Updated**: October 2025
