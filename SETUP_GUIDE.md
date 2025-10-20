# Quick Setup Guide for Notes Feature

## Step 1: Install Backend Dependencies
```powershell
cd backend
npm install
```

The required packages (multer and pdf-parse) are already in package.json and will be installed.

## Step 2: Update Database Schema

### For Local PostgreSQL:
```bash
# Connect to your database and run:
psql -d your_database_name -f db/schema.sql
```

### For Supabase:
1. Go to Supabase Dashboard
2. Navigate to SQL Editor
3. Copy contents from `backend/db/supabase_schema.sql`
4. Paste and execute in Supabase SQL Editor

## Step 3: Verify Environment Variables

Make sure your `backend/.env` file has:
```
GROQ_API_KEY=your_groq_api_key_here
DATABASE_URL=your_postgresql_connection_string
PORT=5000

# Or individual PostgreSQL variables:
PGHOST=localhost
PGUSER=your_user
PGPASSWORD=your_password
PGDATABASE=your_database
PGPORT=5432
```

## Step 4: Create Uploads Directory

The backend will create this automatically, but you can also create it manually:
```powershell
cd backend
mkdir uploads
```

## Step 5: Start Backend Server
```powershell
cd backend
npm run dev
```

## Step 6: Start Frontend
```powershell
cd frontend
npm run dev
```

## Step 7: Test the Feature

### Test Upload (Professor View):
1. Open browser to `http://localhost:5173/notes/upload`
2. Fill in the form:
   - Title: "Test Notes"
   - Subject: "Computer Science"
   - Description: "Test upload"
   - Select a PDF file
3. Click "Upload & Generate Summary"
4. Wait for AI processing
5. Verify success message and summary display

### Test View (Student View):
1. Open browser to `http://localhost:5173/notes`
2. Verify uploaded note appears
3. Test search functionality
4. Test subject filter
5. Click "View Summary" to see modal
6. Click "Download PDF" to download file
7. Test grid/list view toggle

## Troubleshooting

### Backend won't start:
- Check if port 5000 is available
- Verify DATABASE_URL is correct
- Check GROQ_API_KEY is valid

### Upload fails:
- Verify uploads directory exists and is writable
- Check file size (must be < 10MB)
- Ensure file is a valid PDF

### Summary not generated:
- Verify GROQ_API_KEY in .env
- Check Groq API quotas/limits
- Check backend console for errors

### Download fails:
- Verify file exists in uploads directory
- Check file permissions

## API Testing with cURL

### Upload a note:
```bash
curl -X POST http://localhost:5000/api/notes/upload \
  -F "pdf=@/path/to/your/file.pdf" \
  -F "title=Test Note" \
  -F "description=Test Description" \
  -F "subject=Computer Science" \
  -F "uploaded_by=Professor"
```

### Get all notes:
```bash
curl http://localhost:5000/api/notes
```

### Get specific note:
```bash
curl http://localhost:5000/api/notes/1
```

### Download note:
```bash
curl http://localhost:5000/api/notes/1/download -o downloaded.pdf
```

## Database Verification

Check if tables were created:
```sql
-- List all tables
SELECT tablename FROM pg_catalog.pg_tables 
WHERE schemaname = 'public';

-- Check notes table structure
\d notes

-- Check note_summaries table structure
\d note_summaries

-- View notes data
SELECT id, title, subject, uploaded_by, created_at FROM notes;

-- View summaries
SELECT ns.id, n.title, ns.summary_type, ns.model_used 
FROM note_summaries ns 
JOIN notes n ON ns.note_id = n.id;
```

## Frontend Routes

- `/` - Roadmap Generator (existing)
- `/saved` - Saved Roadmaps (existing)
- `/student` - Student Roadmap View (existing)
- `/notes/upload` - **NEW: Upload Notes (Professor)**
- `/notes` - **NEW: View Notes (Student)**

## Success Indicators

✅ Backend starts without errors
✅ Database tables created successfully
✅ Can upload PDF file
✅ AI summary is generated
✅ Notes appear in student view
✅ Can search and filter notes
✅ Can view summaries in modal
✅ Can download original PDFs

## Next Steps

After successful setup:
1. Add navigation links to existing pages
2. Implement user authentication
3. Add role-based access control (Professor vs Student)
4. Set up file backup/cleanup policies
5. Configure production environment variables
6. Deploy to hosting platform

## Support

If you encounter issues:
1. Check backend console logs
2. Check browser console for frontend errors
3. Verify database connection
4. Test API endpoints directly with cURL
5. Review NOTES_FEATURE_README.md for detailed documentation
