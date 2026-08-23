# KidShield Analysis & Alerts System - Setup & Testing Guide

## System Overview

This system allows parents to analyze text, videos, and games for safety, with all results being automatically saved to the database and displayed in an alerts list.

## Key Features

✅ **Save Only** - Saves content without analysis  
✅ **Analyze & Save** - Analyzes content and saves result  
✅ **Automatic Alerts** - All saved/analyzed content creates an alert  
✅ **Alert Management** - Mark alerts as reviewed or ignored  
✅ **Rich Display** - Shows timestamps, child names, source apps, risk levels  
✅ **Refresh Support** - Manual refresh button + pull-to-refresh  

## Database Flow

```
User clicks "تحليل وحفظ" (Analyze & Save)
    ↓
Backend analyzes text/media
    ↓
Inserts into content_logs table
    ↓
Creates alert entry (status='new')
    ↓
Parent sees success message
    ↓
Alert appears in alerts list
```

## API Endpoints

### POST /analyze_text
- **Request:**
  ```json
  {
    "text": "محتوى النص",
    "user_id": 1,
    "child_id": null,
    "child_name": "اسم الطفل (اختياري)",
    "save_only": false,
    "source_app": "واتساب"
  }
  ```
- **Response:**
  ```json
  {
    "success": true,
    "result": {
      "label": "toxic/safe",
      "risk_level": "high/medium/low",
      "confidence": 0.95,
      "reason": "سبب التصنيف",
      "explanation": "التفسير"
    },
    "log_id": 123
  }
  ```

### POST /analyze_media
- Similar to /analyze_text but with:
  - `url`: Media URL
  - `media_type`: 'video' or 'game'
  - Returns `source_url` and `extracted_text` in response

### GET /alerts?user_id=1
- **Response:**
  ```json
  {
    "success": true,
    "alerts": [
      {
        "id": 1,
        "text": "المحتوى المحلل",
        "risk_level": "high",
        "risk_score": 0.87,
        "reason": "سبب التنبيه",
        "status": "new",
        "is_read": false,
        "date": "2024-05-02T10:30:00",
        "child_name": "أحمد",
        "source_app": "واتساب",
        "explanation": "التفسير"
      }
    ]
  }
  ```

### PUT /alerts/<id>/status
- **Request:** `{"status": "reviewed|ignored"}`
- **Response:** `{"success": true, "message": "..."}`

## Testing Checklist

### 1. Backend Setup
- [ ] Start Flask server: `python app.py`
- [ ] Server running on http://192.168.8.11:5000
- [ ] Database initialized with tables

### 2. Text Analysis
- [ ] Open app and login as parent
- [ ] Go to "التحليل" (Analysis) screen
- [ ] Enter test text in Arabic
- [ ] Click "حفظ فقط" (Save Only)
  - Expected: ✓ Text saved, alert created with status 'new'
  - Message: "تم الحفظ - سيظهر في قائمة التنبيهات"
- [ ] Enter toxic text
- [ ] Click "تحليل وحفظ" (Analyze & Save)
  - Expected: ✓ Text analyzed, alert created if risky
  - Message: Shows risk level and reason

### 3. Media Analysis
- [ ] Go to "فيديو" (Video) tab
- [ ] Enter YouTube URL
- [ ] Click "تحليل وحفظ"
  - Expected: ✓ Video analyzed, alert created
- [ ] Go to "ألعاب" (Games) tab
- [ ] Enter game store URL
- [ ] Click "تحليل وحفظ"
  - Expected: ✓ Game analyzed, alert created

### 4. Alerts List
- [ ] Open "التنبيهات" (Alerts) screen
- [ ] Verify all analyzed content appears in list
- [ ] Check for:
  - [ ] Risk level badge (red/orange/green)
  - [ ] Status badge (جديد/تمت المراجعة/تم التجاهل)
  - [ ] Child name (if available)
  - [ ] Source app (if available)
  - [ ] Relative timestamp (الآن/منذ دقيقة/منذ ساعة/etc)
- [ ] Click refresh button
  - Expected: ✓ List updates
- [ ] Pull to refresh
  - Expected: ✓ List updates
- [ ] Expand an alert
  - Expected: ✓ Shows full text and explanation
- [ ] Click "تحديد كمقروء" (Mark as Read)
  - Expected: ✓ Status changes to "تمت المراجعة"
  - Message: "تم تحديد التنبيه كمقروء"
- [ ] Click "تجاهل" (Ignore)
  - Expected: ✓ Status changes to "تم التجاهل"
  - Message: "تم تجاهل التنبيه"

### 5. Child Mode
- [ ] Login as child user
- [ ] Go to "التحليل" (Analysis) screen
- [ ] Select source app from dropdown
- [ ] Enter text and click "تحليل وحفظ"
- [ ] Login as parent
- [ ] Open alerts
  - Expected: ✓ See child's analysis with child name and source app

## Database Queries for Testing

### Check if alerts were created
```sql
SELECT a.id, cl.text_content, cl.risk_level, a.reason, a.status, a.created_at
FROM alerts a
JOIN content_logs cl ON a.content_log_id = cl.id
WHERE a.user_id = 1
ORDER BY a.created_at DESC;
```

### Check content logs
```sql
SELECT * FROM content_logs WHERE user_id = 1 ORDER BY created_at DESC;
```

### Update alert status
```sql
UPDATE alerts SET status = 'reviewed' WHERE id = 1;
```

## Troubleshooting

### Alerts not appearing in list
1. Check database connection: `sqlite3 backend/database.db`
2. Verify alerts table exists: `.tables`
3. Check if alerts were created: See SQL queries above
4. Restart backend server

### Analysis not working
1. Check if text classifier model is trained
2. Verify backend is running on correct port
3. Check error logs in backend output

### Timestamps showing wrong format
- Verify datetime parsing in _formatDate method
- Check if dates are being stored correctly in database

## Files Modified

- `backend/app.py` - analyze_text & analyze_media endpoints
- `lib/screens/analyze_text_screen.dart` - Enhanced feedback
- `lib/screens/alerts_screen.dart` - Added refresh, better display

## Next Steps

1. Run the Flutter app: `flutter run`
2. Test the complete flow as per Testing Checklist
3. Monitor backend logs for any errors
4. Report any issues found during testing
