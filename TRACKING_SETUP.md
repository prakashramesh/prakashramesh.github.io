# Visit Tracking Setup Guide

The tracking page now uses **JSONBin.io**, a free online JSON storage service. No Node.js server required!

## Quick Setup (5 minutes)

### Step 1: Create a JSONBin.io Account
1. Go to [https://jsonbin.io](https://jsonbin.io)
2. Click "Sign Up" and create a free account
3. Verify your email if required

### Step 2: Create a Bin
1. After logging in, click "Create Bin" or "New Bin"
2. Set the bin name (e.g., "site-visits")
3. Set the bin to **Public** (so it can be accessed without authentication)
4. Initialize it with an empty array: `[]`
5. Click "Create"

### Step 3: Get Your Bin ID
1. After creating the bin, you'll see the bin details
2. Copy the **Bin ID** (it looks like: `507f1f77bcf86cd799439011`)
3. The bin URL will be something like: `https://jsonbin.io/507f1f77bcf86cd799439011`

### Step 4: Configure the Tracking Page
1. Open `_tabs/tracking.md` in your editor
2. Find this line (around line 242):
   ```javascript
   const JSONBIN_BIN_ID = 'YOUR_BIN_ID';
   ```
3. Replace `'YOUR_BIN_ID'` with your actual bin ID:
   ```javascript
   const JSONBIN_BIN_ID = '507f1f77bcf86cd799439011';
   ```

### Step 5: (Optional) Add API Key for Private Bins
If you want to use a private bin for security:
1. Go to your JSONBin.io account settings
2. Copy your API key (X-Master-Key)
3. In `_tabs/tracking.md`, find:
   ```javascript
   const JSONBIN_API_KEY = '';
   ```
4. Add your API key:
   ```javascript
   const JSONBIN_API_KEY = 'your-api-key-here';
   ```

## That's It!

Your tracking page will now:
- ✅ Store all visits in JSONBin.io (free, online storage)
- ✅ Work without any local server
- ✅ Work on GitHub Pages (no backend needed)
- ✅ Persist data across all visitors

## Testing

1. Visit your tracking page: `http://localhost:4000/tracking/`
2. The page should load and record your visit
3. Check your JSONBin.io dashboard to see the data being stored

## Troubleshooting

**Error: "Please configure JSONBin.io bin ID"**
- Make sure you've replaced `YOUR_BIN_ID` with your actual bin ID

**Error: "Failed to fetch visits"**
- Check that your bin is set to **Public** in JSONBin.io
- Verify the bin ID is correct
- Check browser console for detailed error messages

**CORS Errors**
- JSONBin.io supports CORS, but if you see errors, make sure you're using the correct API endpoint format

## Free Tier Limits

JSONBin.io free tier includes:
- Unlimited bins
- 10,000 requests/month
- Public bins (no authentication needed)
- Perfect for personal sites!

For higher limits, check their pricing at [jsonbin.io/pricing](https://jsonbin.io/pricing)

