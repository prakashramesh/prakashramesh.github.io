# Hidden Visit Tracker Setup

This is a **hidden** visit tracking page accessible only via the special URL: `/visit-tracker/`

The page is not listed in the navigation menu and can only be accessed by directly visiting the URL.

## Setup Instructions

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

### Step 4: Configure the Tracking Page
1. Open `visit-tracker.md` in your editor
2. Find this line (around line 204):
   ```javascript
   const JSONBIN_BIN_ID = '6916a610d0ea881f40e7573d';
   ```
3. Replace the bin ID with your actual bin ID:
   ```javascript
   const JSONBIN_BIN_ID = 'your-actual-bin-id-here';
   ```

### Step 5: (Optional) Add API Key for Private Bins
If you want to use a private bin for security:
1. Go to your JSONBin.io account settings
2. Copy your API key (X-Master-Key)
3. In `visit-tracker.md`, find:
   ```javascript
   const JSONBIN_API_KEY = '';
   ```
4. Add your API key:
   ```javascript
   const JSONBIN_API_KEY = 'your-api-key-here';
   ```

## Accessing the Page

The tracking page is accessible at:
- **Local**: `http://localhost:4000/visit-tracker/`
- **Production**: `https://prakashramesh.github.io/visit-tracker/`

The page will **NOT** appear in the navigation menu - it's completely hidden and can only be accessed via the direct URL.

## Features

- ✅ Hidden from navigation (not in menu)
- ✅ Accessible only via special URL path
- ✅ Geo map visualization of visit locations
- ✅ Visit history table
- ✅ Statistics dashboard
- ✅ Automatic visit tracking
- ✅ Free online storage (JSONBin.io)

## Free Tier Limits

JSONBin.io free tier includes:
- Unlimited bins
- 10,000 requests/month
- Public bins (no authentication needed)
- Perfect for personal sites!

