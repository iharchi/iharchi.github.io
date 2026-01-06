# n8n Workflow Setup Guide
## Real Estate Lead Capture & Welcome Email Automation

This guide will help you set up the n8n workflow to automatically send welcome emails and subscribe leads from your website.

---

## 📋 Prerequisites

1. **n8n Instance** (choose one):
   - Self-hosted: https://docs.n8n.io/hosting/
   - n8n Cloud: https://n8n.io/cloud/

2. **Gmail Account** (for sending emails)

3. **Mailchimp Account** (for subscriber management)

4. **Google Sheets** (optional, for logging leads)

---

## 🚀 Step-by-Step Setup

### Step 1: Import the Workflow into n8n

1. Open your n8n instance
2. Click on **Workflows** in the left sidebar
3. Click the **"+"** button to create a new workflow
4. Click the **three dots (⋮)** in the top right
5. Select **"Import from File"**
6. Upload the `n8n-workflow-lead-capture.json` file
7. The workflow will be imported with all nodes

### Step 2: Configure Credentials

#### A. Gmail SMTP (for sending emails)

1. Click on the **"Send Welcome Email"** node
2. Click **"Create New Credential"** under SMTP credentials
3. Fill in the following:
   ```
   User: harchi.isaak@gmail.com
   Password: [Your App Password]
   Host: smtp.gmail.com
   Port: 587
   SSL/TLS: Enable
   ```
4. **Important:** For Gmail, you need to create an App Password:
   - Go to: https://myaccount.google.com/apppasswords
   - Select "Mail" and your device
   - Copy the 16-character password
   - Use this instead of your regular password

#### B. Mailchimp API

1. Click on the **"Subscribe to Mailchimp"** node
2. Click **"Create New Credential"**
3. Get your Mailchimp API Key:
   - Log into Mailchimp
   - Go to Account → Extras → API Keys
   - Create a new key or copy existing one
4. Enter your API key in n8n
5. In the node settings, update:
   - **List ID**: Your Mailchimp audience/list ID
     - Find it: Audience → Settings → Audience name and defaults

#### C. Google Sheets (Optional)

1. Click on the **"Log to Google Sheets"** node
2. Click **"Create New Credential"**
3. Follow n8n's OAuth flow to connect your Google account
4. Create a new Google Sheet with these column headers:
   ```
   Timestamp | First Name | Last Name | Email | Phone | Guide Type | Status
   ```
5. In the node, select your spreadsheet and sheet name

### Step 3: Activate the Webhook

1. Click on the **"Webhook - Form Submission"** node
2. Click **"Execute Node"** to start the webhook
3. Copy the **Production Webhook URL**
   - It will look like: `https://your-n8n-instance.com/webhook/real-estate-lead`

### Step 4: Update Your Website

1. Open `index.html`
2. Find line 735 where it says:
   ```javascript
   const N8N_WEBHOOK_URL = 'YOUR_N8N_WEBHOOK_URL_HERE';
   ```
3. Replace with your actual webhook URL:
   ```javascript
   const N8N_WEBHOOK_URL = 'https://your-n8n-instance.com/webhook/real-estate-lead';
   ```
4. Save the file

### Step 5: Activate the Workflow

1. In n8n, toggle the workflow to **"Active"** (switch in top right)
2. Save the workflow (Ctrl+S or Cmd+S)

---

## 🧪 Testing the Workflow

### Test from n8n (Manual Test)

1. Click on the **"Webhook - Form Submission"** node
2. Click **"Listen for Test Event"**
3. Use this test data in Postman or curl:

```bash
curl -X POST https://your-n8n-instance.com/webhook/real-estate-lead \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "test@example.com",
    "phone": "(555) 123-4567",
    "guideType": "buying",
    "timestamp": "2026-01-06T12:00:00Z",
    "source": "website"
  }'
```

4. Check that:
   - ✅ Email is sent to the test address
   - ✅ Lead is added to Mailchimp
   - ✅ Entry appears in Google Sheets
   - ✅ Webhook returns success response

### Test from Your Website

1. Deploy your updated `index.html` to your website
2. Fill out one of the forms (Selling, Buying, or Inspection)
3. Submit the form
4. Verify:
   - ✅ Success message appears on the website
   - ✅ Welcome email is received
   - ✅ Subscriber appears in Mailchimp
   - ✅ Lead is logged in Google Sheets

---

## 📧 Customizing the Welcome Email

The welcome email is defined in the **"Send Welcome Email"** node.

To customize it:
1. Click on the "Send Welcome Email" node
2. Edit the HTML in the "Message" field
3. Available variables you can use:
   - `{{$json.firstName}}` - First name
   - `{{$json.lastName}}` - Last name
   - `{{$json.email}}` - Email address
   - `{{$json.phone}}` - Phone number
   - `{{$json.guideType}}` - Guide type (selling/buying/inspection)

### Email Personalization Examples

```html
<!-- Personalized greeting -->
<h1>Welcome, {{$json.firstName}}!</h1>

<!-- Conditional content based on guide type -->
{{#if $json.guideType === 'buying'}}
  <p>Excited to help you find your dream home!</p>
{{else if $json.guideType === 'selling'}}
  <p>Let's get your home sold for top dollar!</p>
{{else}}
  <p>Home inspections are crucial - let me guide you!</p>
{{/if}}
```

---

## 🔄 Workflow Features

### What Happens When a Form is Submitted:

1. **Webhook Receives Data** → Captures form submission
2. **Data Validation** → Checks all required fields are present
3. **Format Data** → Splits name, formats fields
4. **Three Parallel Actions:**
   - 📧 **Send Welcome Email** (personalized HTML email)
   - 📬 **Subscribe to Mailchimp** (with tags: guide-type, lead)
   - 📊 **Log to Google Sheets** (for tracking/backup)
5. **Return Success** → Confirms submission to website

### Fallback Mechanism

If the n8n webhook fails, the website will automatically fall back to direct Mailchimp submission (legacy method).

---

## 🛠 Troubleshooting

### Problem: Email not being sent

**Solutions:**
- Verify Gmail App Password is correct
- Check spam/junk folder
- Verify SMTP settings: smtp.gmail.com:587 with TLS
- Check n8n execution logs for errors

### Problem: Mailchimp subscription failing

**Solutions:**
- Verify API key is valid
- Check List ID is correct (go to Audience → Settings)
- Ensure email doesn't already exist with "pending" status
- Enable "Update Existing" option in Mailchimp node

### Problem: Webhook not receiving data

**Solutions:**
- Ensure workflow is **Active** (toggle in top right)
- Verify webhook URL is correct in index.html
- Check browser console for CORS errors
- Test with curl command first

### Problem: Google Sheets not updating

**Solutions:**
- Verify OAuth connection is active
- Check spreadsheet sharing permissions
- Ensure column names match exactly
- Re-authenticate Google credential if needed

---

## 📊 Monitoring & Analytics

### View Execution History

1. In n8n, go to **Executions** tab
2. See all workflow runs with timestamps
3. Click any execution to see detailed logs
4. Check for errors in failed executions

### Mailchimp Analytics

- Log into Mailchimp → Audience → View contacts
- Filter by tags: "selling-guide", "buying-guide", "inspection-guide"
- Track engagement rates

### Google Sheets Reports

Create formulas to track:
- Total leads per day: `=COUNTIF(A:A, TODAY())`
- Leads by guide type: `=COUNTIF(F:F, "buying")`
- Conversion tracking (update Status column)

---

## 🔐 Security Best Practices

1. **Never commit credentials** to GitHub
2. **Use environment variables** in n8n for sensitive data
3. **Enable HTTPS** on your n8n instance
4. **Rotate API keys** regularly
5. **Monitor execution logs** for suspicious activity
6. **Add rate limiting** to prevent abuse

---

## 🎯 Next Steps & Enhancements

### Optional Improvements:

1. **Add SMS Notifications**
   - Use Twilio node to send text messages
   - Notify you when new leads come in

2. **CRM Integration**
   - Add Salesforce, HubSpot, or Pipedrive nodes
   - Automatically create deals/contacts

3. **Drip Campaign**
   - Set up follow-up emails (Day 3, 7, 14)
   - Use n8n Schedule trigger + Mailchimp

4. **Slack Notifications**
   - Get instant notifications in Slack
   - Add Slack node after webhook

5. **Lead Scoring**
   - Use Function node to score leads
   - Prioritize based on guide type, time, etc.

6. **PDF Generation**
   - Dynamically generate personalized PDFs
   - Attach to welcome email

---

## 📞 Support

If you need help:
- n8n Documentation: https://docs.n8n.io/
- n8n Community: https://community.n8n.io/
- Mailchimp API Docs: https://mailchimp.com/developer/

---

## ✅ Deployment Checklist

Before going live:

- [ ] n8n workflow imported and saved
- [ ] All credentials configured (Gmail, Mailchimp, Google Sheets)
- [ ] Webhook URL copied from n8n
- [ ] index.html updated with webhook URL
- [ ] Workflow activated in n8n
- [ ] Test submission completed successfully
- [ ] Welcome email received
- [ ] Mailchimp subscription confirmed
- [ ] Google Sheets entry created
- [ ] Website deployed with updated index.html
- [ ] Fallback to Mailchimp tested

---

**You're all set!** 🎉

Your automated lead capture system is now ready to send welcome emails and subscribe leads automatically.
