# AI Task Router - CORS Proxy Deployment

## Quick Fix for CORS Issue

The AI Task Router app is fully functional, but browsers block direct calls to the Relay webhook due to CORS preflight requests. Here's the 5-minute solution:

## Option 1: Deploy Cloudflare Worker (Recommended - FREE)

### Step 1: Create Cloudflare Account
1. Go to https://workers.cloudflare.com/
2. Click "Sign Up" (free forever for up to 100,000 requests/day)
3. Verify your email

### Step 2: Create Worker
1. Click "Create Worker"
2. Replace the code with this:

```javascript
export default {
  async fetch(request) {
    // Handle CORS preflight
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        },
      });
    }

    // Proxy POST request to Relay webhook
    if (request.method === 'POST') {
      const RELAY_WEBHOOK = 'https://hook.relay.app/c/cmj6tpc6n722l';
      
      const response = await fetch(RELAY_WEBHOOK, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: await request.text(),
      });

      const html = await response.text();
      
      return new Response(html, {
        headers: {
          'Content-Type': 'text/html',
          'Access-Control-Allow-Origin': '*',
        },
      });
    }

    return new Response('Method not allowed', { status: 405 });
  },
};
```

3. Click "Save and Deploy"
4. Copy your worker URL (e.g., `https://ai-task-router.YOUR-SUBDOMAIN.workers.dev`)

### Step 3: Update index.html
1. Edit `index.html`
2. Find line 228: `const WEBHOOK_URL = ...`
3. Replace the CORS proxy URL with your Cloudflare Worker URL
4. Commit changes

**Done!** Your app will now work perfectly.

---

## Option 2: Use Vercel Serverless Function

If you prefer Vercel:

1. Create `api/proxy.js`:
```javascript
export default async function handler(req, res) {
  if (req.method === 'OPTIONS') {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
    return res.status(200).end();
  }

  if (req.method === 'POST') {
    const response = await fetch('https://hook.relay.app/c/cmj6tpc6n722l', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(req.body),
    });

    const html = await response.text();
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Content-Type', 'text/html');
    return res.status(200).send(html);
  }

  res.status(405).send('Method not allowed');
}
```

2. Deploy to Vercel
3. Update `WEBHOOK_URL` in index.html to your Vercel URL

---

## Testing

Once deployed, test your app at:
https://jain26k.github.io/ai-task-router/

Enter any task (e.g., "Create a professional logo") and click "Find Best AI Tool" - you should get AI-powered recommendations!

## Current Status

✅ Relay.app workflow: Working perfectly
✅ Web interface: Deployed and beautiful  
✅ GitHub Pages: Live
⏳ CORS proxy: Needs deployment (5 minutes)

The entire app is ready - it just needs this final CORS proxy step!
