# 🚀 Google Ads Weekly Performance to LarkBase - Setup Guide

## 📋 Overview

This workflow automatically:
- ✅ Pulls Google Ads data via **SQL queries (GAQL)**
- ✅ Separates data by **campaign type** (Search, Display, Performance Max)
- ✅ Calculates **week-over-week comparison**
- ✅ Saves structured reports to **LarkBase**
- ✅ Runs **every Monday at 8 AM**

---

## 🔧 Prerequisites

### 1. Google Ads API Access
- **Google Ads Developer Token** (apply at: https://ads.google.com/home/tools/manager-accounts/)
- **OAuth 2.0 Client ID & Secret** (from Google Cloud Console)
- **Customer ID** (format: XXX-XXX-XXXX)

### 2. Lark/Feishu Access
- **App ID & App Secret** (from Lark Developer Console)
- **App Token** (your Lark Base app token)
- **Table ID** (the specific table in your Lark Base)

---

## 📊 LarkBase Table Structure

Create a table in LarkBase with these columns:

| Column Name | Type | Description |
|-------------|------|-------------|
| Client Name | Text | Client identifier |
| Campaign Type | Single Select | Search Campaign, Display Campaign, Performance Max |
| Week Start | Date | Start date of reporting week |
| Week End | Date | End date of reporting week |
| **This Week Metrics** | | |
| Impressions (This Week) | Number | Total impressions |
| Clicks (This Week) | Number | Total clicks |
| Cost (This Week) | Number | Total cost in currency |
| Conversions (This Week) | Number | Total conversions |
| Conversion Value (This Week) | Number | Total conversion value |
| CTR (This Week) | Number | Click-through rate (%) |
| Avg CPC (This Week) | Number | Average cost per click |
| Cost/Conv (This Week) | Number | Cost per conversion |
| Conv Rate (This Week) | Number | Conversion rate (%) |
| ROAS (This Week) | Number | Return on ad spend |
| **Last Week Metrics** | | |
| Impressions (Last Week) | Number | Previous week impressions |
| Clicks (Last Week) | Number | Previous week clicks |
| Cost (Last Week) | Number | Previous week cost |
| Conversions (Last Week) | Number | Previous week conversions |
| Conversion Value (Last Week) | Number | Previous week conversion value |
| CTR (Last Week) | Number | Previous week CTR (%) |
| Avg CPC (Last Week) | Number | Previous week avg CPC |
| Cost/Conv (Last Week) | Number | Previous week cost/conv |
| Conv Rate (Last Week) | Number | Previous week conv rate (%) |
| ROAS (Last Week) | Number | Previous week ROAS |
| **Week-over-Week Changes** | | |
| Cost Change % | Number | Percentage change in cost |
| Clicks Change % | Number | Percentage change in clicks |
| Impressions Change % | Number | Percentage change in impressions |
| Conversions Change % | Number | Percentage change in conversions |
| **Metadata** | | |
| Currency | Text | MYR, USD, etc. |
| Report Generated | DateTime | Timestamp of report generation |

---

## ⚙️ Configuration Steps

### Step 1: Import the Workflow
```bash
# In n8n UI:
1. Go to Workflows → Import from File
2. Select: google-ads-weekly-larkbase-report.json
3. Click "Import"
```

### Step 2: Configure Google Ads Credentials
```
1. In n8n: Settings → Credentials → Add Credential
2. Select: Google Ads OAuth2 API
3. Enter:
   - Developer Token: YOUR_DEVELOPER_TOKEN
   - Client ID: YOUR_GOOGLE_CLIENT_ID
   - Client Secret: YOUR_GOOGLE_CLIENT_SECRET
4. Complete OAuth flow
5. Note the Credential ID
```

### Step 3: Update Workflow Configuration

#### Node: "Client Configuration" (Line 22-46)
```json
{
  "customer_id": "123-456-7890",      // Your Google Ads Customer ID
  "client_name": "Acme Corporation",  // Your client name
  "currency": "MYR"                   // Currency code
}
```

#### Nodes: "Google Ads - This Week Data" & "Google Ads - Last Week Data" (Lines 48-87)
```json
{
  "credentials": {
    "googleAdsOAuth2Api": {
      "id": "YOUR_ACTUAL_CREDENTIAL_ID",  // Replace with your Google Ads credential ID
      "name": "Google Ads account"
    }
  }
}
```

#### Node: "Lark Configuration" (Lines 98-119)
```json
{
  "app_token": "VsfLbG5NtaBXs0sDEi6l7yuigMg",  // Your Lark Base app token
  "table_id": "tblzK0a6l4bhySJE"              // Your Lark Base table ID
}
```

#### Node: "Lark Secrets" (Lines 120-141)
```json
{
  "app_id": "cli_a1b2c3d4e5f6g7h8",      // Your Feishu App ID
  "app_secret": "YOUR_APP_SECRET_HERE"  // Your Feishu App Secret
}
```

---

## 🔍 Understanding the SQL Queries

### This Week Data (Line 53)
```sql
SELECT 
  campaign.id, 
  campaign.name, 
  campaign.status, 
  campaign.advertising_channel_type,  -- This identifies campaign type
  metrics.impressions, 
  metrics.clicks, 
  metrics.cost_micros, 
  metrics.conversions, 
  metrics.conversions_value, 
  metrics.ctr, 
  metrics.average_cpc, 
  metrics.cost_per_conversion, 
  segments.date 
FROM campaign 
WHERE segments.date DURING LAST_7_DAYS 
  AND campaign.status = 'ENABLED'
```

### Last Week Data (Line 73)
```sql
-- Same query but with different date range:
WHERE segments.date DURING LAST_14_DAYS_EXCLUDING_LAST_7_DAYS
```

---

## 📈 Data Flow

```
1. Weekly Trigger (Monday 8 AM)
   ↓
2. Client Configuration (set customer ID, name, currency)
   ↓
3. [PARALLEL] Pull Google Ads Data
   ├─ This Week (last 7 days)
   └─ Last Week (days 8-14)
   ↓
4. Process & Compare Campaign Data
   - Aggregates by campaign type (Search, Display, Performance Max)
   - Calculates metrics (CTR, CPC, ROAS, etc.)
   - Computes week-over-week changes
   ↓
5. Lark Authentication
   ├─ Get tenant access token
   └─ Build authorization header
   ↓
6. Transform to Lark Format
   - Maps to LarkBase field structure
   - Formats numbers and dates
   ↓
7. Save to LarkBase
   - Batch creates 3 records (one per campaign type)
   ↓
8. Log Results
```

---

## 🎯 Campaign Type Mapping

The workflow automatically identifies and labels campaigns:

| Google Ads Type | Display Name in LarkBase |
|----------------|--------------------------|
| `SEARCH` | Search Campaign |
| `DISPLAY` | Display Campaign |
| `PERFORMANCE_MAX` | Performance Max |
| `SHOPPING` | Shopping Campaign |
| `VIDEO` | Video Campaign |

---

## 🧪 Testing the Workflow

### Manual Test Run
```
1. In n8n workflow editor
2. Click "Execute Workflow" button
3. Check each node's output
4. Verify data in LarkBase
```

### Debug Node Outputs

**Node: "Google Ads - This Week Data"**
- Should return JSON with `results` array
- Each result has `campaign` and `metrics` objects

**Node: "Process & Compare Campaign Data"**
- Should output 3 records (Search, Display, Performance Max)
- Each record has `_this_week`, `_last_week`, and `_change` fields

**Node: "Save to LarkBase"**
- Response should have `code: 0` (success)
- `data.records` array shows created records

---

## 📅 Schedule Configuration

**Default: Every Monday at 8:00 AM**

To change schedule, edit node "Weekly Trigger (Monday 8AM)" (Lines 4-21):

```json
{
  "rule": {
    "interval": [
      {
        "field": "weeks",
        "triggerAtDay": [1],    // 0=Sunday, 1=Monday, 2=Tuesday, etc.
        "triggerAtHour": 8,     // Hour (0-23)
        "triggerAtMinute": 0    // Optional: minute (0-59)
      }
    ]
  }
}
```

**Examples:**
- Every Friday at 5 PM: `triggerAtDay: [5], triggerAtHour: 17`
- Every Sunday at 10 AM: `triggerAtDay: [0], triggerAtHour: 10`

---

## 🔧 Troubleshooting

### Issue: "Google Ads API Error: Invalid customer ID"
**Solution:** Check customer ID format (XXX-XXX-XXXX) in "Client Configuration" node

### Issue: "Lark API Error: 99991663"
**Solution:** App token expired. Get new tenant access token

### Issue: No data in LarkBase
**Solution:** 
1. Check LarkBase column names match exactly
2. Verify table_id is correct
3. Check app has write permissions

### Issue: "No campaigns found"
**Solution:**
1. Verify campaigns are ENABLED
2. Check date range in SQL query
3. Ensure customer ID has active campaigns

---

## 📊 Expected Output (Example)

After successful run, LarkBase will have 3 new records:

| Campaign Type | This Week Cost | Last Week Cost | Cost Change % |
|--------------|----------------|----------------|---------------|
| Search Campaign | 1,234.56 MYR | 1,100.00 MYR | +12.23% |
| Display Campaign | 456.78 MYR | 500.00 MYR | -8.64% |
| Performance Max | 789.01 MYR | 750.00 MYR | +5.20% |

---

## 🚀 Next Steps

1. ✅ **Test with your Google Ads account**
2. ✅ **Verify data accuracy in LarkBase**
3. ✅ **Add more clients** (duplicate workflow per client)
4. ✅ **Create LarkBase dashboard** with charts
5. ✅ **Set up alerts** for performance drops

---

## 💡 Pro Tips

### Multiple Clients
To track multiple clients, either:
1. **Option A**: Duplicate this workflow for each client
2. **Option B**: Modify "Client Configuration" to loop through multiple customers

### Custom Metrics
To add more metrics, modify the SQL query (line 53):
```sql
-- Add more metrics like:
metrics.bounce_rate,
metrics.engagement_rate,
metrics.video_views
```

### Longer Time Ranges
Change date range in SQL:
- `LAST_30_DAYS` for monthly comparison
- `THIS_MONTH` vs `LAST_MONTH` for month-over-month

---

## 📞 Support

If you encounter issues:
1. Check n8n execution logs
2. Verify Google Ads API quota
3. Test Lark API access with manual HTTP request
4. Review LarkBase field types match workflow output

---

**Ready to automate? Import the workflow and start tracking! 🎯**
