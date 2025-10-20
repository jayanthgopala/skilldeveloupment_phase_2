# Notes Management Feature

## Overview
This feature allows professors to upload PDF course notes which are automatically summarized using AI (Groq's Llama 3.3 70B model). Students can then access both the original PDF files and the AI-generated summaries.

## Features

### For Professors (Upload)
- **PDF Upload**: Upload course notes in PDF format (max 10MB)
- **Metadata**: Add title, description, subject, and tags
- **AI Summarization**: Automatic summary generation using advanced AI
- **Real-time Processing**: Instant feedback on upload and processing status

### For Students (View)
- **Browse Notes**: View all available course notes
- **Search & Filter**: Search by title, description, tags, or filter by subject
- **View Summaries**: Read AI-generated summaries without downloading PDFs
- **Download PDFs**: Download original PDF files
- **Grid/List View**: Toggle between different viewing modes

## API Endpoints

### Backend Endpoints

#### 1. Upload Note
```
POST /api/notes/upload
Content-Type: multipart/form-data

Fields:
- pdf (file): PDF file to upload
- title (string): Note title
- description (string): Note description (optional)
- subject (string): Subject name (optional)
- tags (string): Comma-separated tags (optional)
- uploaded_by (string): Name of uploader

Response:
{
  "note": {
    "id": 1,
    "title": "Introduction to Data Structures",
    "filename": "notes.pdf",
    "file_size": 2048576,
    "uploaded_by": "Professor",
    ...
  },
  "summary": {
    "id": 1,
    "note_id": 1,
    "summary_text": "...",
    "model_used": "llama-3.3-70b-versatile",
    ...
  }
}
```

#### 2. Get All Notes
```
GET /api/notes
Query Parameters:
- subject (optional): Filter by subject
- uploaded_by (optional): Filter by uploader

Response: Array of notes with summaries
```

#### 3. Get Single Note
```
GET /api/notes/:id

Response: Note object with all summaries
```

#### 4. Download Note
```
GET /api/notes/:id/download

Response: PDF file download
```

#### 5. Delete Note
```
DELETE /api/notes/:id

Response: 204 No Content
```

## Database Schema

### Notes Table
```sql
CREATE TABLE notes (
  id SERIAL PRIMARY KEY,
  title TEXT NOT NULL,
  description TEXT,
  filename TEXT NOT NULL,
  file_path TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type TEXT NOT NULL,
  uploaded_by TEXT NOT NULL,
  subject TEXT,
  tags TEXT[],
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);
```

### Note Summaries Table
```sql
CREATE TABLE note_summaries (
  id SERIAL PRIMARY KEY,
  note_id INTEGER REFERENCES notes(id) ON DELETE CASCADE,
  summary_text TEXT NOT NULL,
  summary_type TEXT DEFAULT 'brief',
  generated_by TEXT DEFAULT 'AI',
  model_used TEXT,
  tokens_used INTEGER,
  metadata JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  UNIQUE(note_id, summary_type)
);
```

## Frontend Routes

- `/notes/upload` - Professor interface for uploading notes
- `/notes` - Student interface for viewing and downloading notes

## Installation

### Backend Dependencies
```bash
cd backend
npm install multer pdf-parse
```

### Database Migration
Run the updated schema:
```bash
# For PostgreSQL
psql -d your_database -f db/schema.sql

# For Supabase
# Copy contents of db/supabase_schema.sql to Supabase SQL editor and run
```

## Usage

### Professors
1. Navigate to `/notes/upload`
2. Fill in note details (title is required)
3. Select a PDF file
4. Click "Upload & Generate Summary"
5. Wait for AI processing (usually 10-30 seconds)
6. View the uploaded note and generated summary

### Students
1. Navigate to `/notes`
2. Browse available notes in grid or list view
3. Use search bar to find specific notes
4. Filter by subject if needed
5. Click "View Summary" to see AI-generated summary
6. Click "Download PDF" to get the original file

## Environment Variables

Ensure these are set in your backend `.env`:
```
GROQ_API_KEY=your_groq_api_key
DATABASE_URL=your_postgresql_connection_string
```

## File Storage

- PDF files are stored in `backend/uploads/` directory
- Files are named with timestamp and random suffix for uniqueness
- Original filenames are preserved in database metadata

## Security Considerations

1. **File Size Limit**: 10MB max per file
2. **File Type Validation**: Only PDF files accepted
3. **SQL Injection**: Parameterized queries used throughout
4. **Path Traversal**: Secure file path handling
5. **CORS**: Configured for specific origins

## AI Summary Generation

- **Model**: Groq Llama 3.3 70B Versatile
- **Content Limit**: First 15,000 characters of PDF
- **Temperature**: 0.3 (for consistent results)
- **Max Tokens**: 2,000
- **Summary Structure**:
  - Main Topics
  - Key Concepts
  - Comprehensive Overview
  - Important Points

## Future Enhancements

1. Multiple summary types (brief, detailed, key points)
2. User authentication and authorization
3. Note versioning
4. Collaborative annotations
5. Quiz generation from notes
6. Audio/video transcription support
7. Multi-language support
8. Export summaries as PDF/Word

## Troubleshooting

### Upload Fails
- Check file size (must be < 10MB)
- Verify file is valid PDF
- Check GROQ_API_KEY is set
- Ensure database connection is active

### Summary Not Generated
- Check Groq API key validity
- Verify API rate limits not exceeded
- Check backend logs for errors

### Download Fails
- Verify file exists in uploads directory
- Check file permissions
- Ensure correct note ID is used

## Support

For issues or questions, check:
1. Backend logs: `backend/` console output
2. Frontend console: Browser developer tools
3. Database logs: PostgreSQL/Supabase logs
