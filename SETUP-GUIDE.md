# Getting insertfriction.com Live — Step by Step

**Goal:** Landing page live at insertfriction.com with working email waitlist by Friday evening.  
**Stack:** GitHub (public repo) + Netlify (hosting) + Namecheap (domain) + Kit/ConvertKit (email capture)  
**Time estimate:** 2-3 hours total

---

## Overview of Steps

1. Set up Kit (ConvertKit) for email capture
2. Update the landing page code with your Kit form endpoint
3. Create GitHub repo and push the code
4. Connect Netlify to the repo
5. Point your Namecheap domain to Netlify
6. Test everything

---

## Step 1: Set Up Kit (ConvertKit) for Email Capture

Kit (formerly ConvertKit) has a free tier that supports up to 10,000 subscribers. This is your email backend — when someone enters their email on the landing page, it goes directly into your Kit account where you can manage subscribers, send updates, and eventually segment between students, parents, and educators.

### 1a. Create your Kit account

- Go to **https://app.kit.com/users/signup**
- Sign up with your email
- Select the **free plan** (called "Newsletter" — it's free up to 10,000 subscribers)
- Complete the onboarding (you can skip most of it or fill in basics)
  - Creator name: Friction
  - Website: insertfriction.com

### 1b. Create a Form

Kit uses "forms" to collect emails. You need one form for the waitlist.

- In your Kit dashboard, go to **Grow** > **Landing Pages & Forms**
- Click **+ Create new** > **Form**
- Choose **Inline** form type (we're embedding it in your page, not using a popup)
- You can use any template — we won't actually use Kit's visual form. We just need the form ID for our API integration
- Name it **"Friction Waitlist"**
- Save it

### 1c. Get your Form ID

- After saving the form, look at the URL in your browser. It will look something like:  
  `https://app.kit.com/forms/designers/1234567/edit`
- The number (e.g., **1234567**) is your Form ID. Copy this — you'll need it in Step 2.

**Alternative method to find Form ID:**
- Go to **Grow** > **Landing Pages & Forms**
- Click on your Friction Waitlist form
- Click **Settings** or the share/embed option
- Look for the form ID in the embed code or URL

### 1d. Verify the API endpoint works

Kit's form API endpoint follows this pattern:
```
https://api.kit.com/v4/forms/YOUR_FORM_ID/subscriptions
```

You can test it in your browser's developer console (F12 > Console):
```javascript
fetch('https://api.kit.com/v4/forms/YOUR_FORM_ID/subscriptions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email_address: 'test@test.com' })
}).then(r => r.json()).then(console.log)
```

Replace YOUR_FORM_ID with your actual form ID. If it works, you'll see a success response and "test@test.com" will appear in your Kit subscribers list. Delete the test subscriber afterward.

**Important note:** Kit may require an API key for v4. If the above doesn't work, try the legacy v3 endpoint instead:
```
https://api.convertkit.com/v3/forms/YOUR_FORM_ID/subscribe
```
With this body format:
```javascript
{ api_key: 'YOUR_API_KEY', email: 'test@test.com' }
```

To find your API key: Kit dashboard > **Settings** > **Developer** (or **Advanced**). Copy the API key shown there.

---

## Step 2: Update the Landing Page Code

Open the `index.html` file in VS Code on your Chromebook.

Find this section in the JavaScript at the bottom of the file (around line 250):

```javascript
// TODO: Replace this section with your ConvertKit/Kit form submission
```

**Replace the entire form event listener** with one of these options, depending on which API endpoint worked in Step 1d:

### Option A: Kit v4 API (preferred)

```javascript
document.getElementById('waitlist-form').addEventListener('submit', async function(e) {
  e.preventDefault();
  const email = this.querySelector('input[type="email"]').value;
  const button = this.querySelector('button');
  button.textContent = 'Joining...';
  button.disabled = true;

  try {
    const response = await fetch('https://api.kit.com/v4/forms/YOUR_FORM_ID/subscriptions', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email_address: email })
    });

    if (response.ok || response.status === 200 || response.status === 201) {
      this.style.display = 'none';
      document.getElementById('signup-success').style.display = 'block';
      document.querySelector('.signup-note').style.display = 'none';
    } else {
      button.textContent = 'Try again';
      button.disabled = false;
    }
  } catch (error) {
    button.textContent = 'Try again';
    button.disabled = false;
  }
});
```

### Option B: Kit v3 / ConvertKit legacy API

```javascript
document.getElementById('waitlist-form').addEventListener('submit', async function(e) {
  e.preventDefault();
  const email = this.querySelector('input[type="email"]').value;
  const button = this.querySelector('button');
  button.textContent = 'Joining...';
  button.disabled = true;

  try {
    const response = await fetch('https://api.convertkit.com/v3/forms/YOUR_FORM_ID/subscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        api_key: 'YOUR_API_KEY',
        email: email
      })
    });

    if (response.ok) {
      this.style.display = 'none';
      document.getElementById('signup-success').style.display = 'block';
      document.querySelector('.signup-note').style.display = 'none';
    } else {
      button.textContent = 'Try again';
      button.disabled = false;
    }
  } catch (error) {
    button.textContent = 'Try again';
    button.disabled = false;
  }
});
```

**Remember to replace:**
- `YOUR_FORM_ID` with the form ID from Step 1c
- `YOUR_API_KEY` with your API key from Kit settings (Option B only)

### Optional: Tweak the copy

Now is the time to adjust any wording on the page to match your voice. The key sections to review:
- Hero headline and subtitle
- The problem section
- The teen quote (make sure you're comfortable with this being public)
- The "For Families" section
- The signup section copy

Save the file when done.

---

## Step 3: Create GitHub Repo and Push

### 3a. Create the repo on GitHub

- Go to **https://github.com/new**
- Repository name: `insertfriction-site` (or whatever you prefer)
- Set to **Public** (required for GitHub Pages / Netlify free tier with public repos)
- Do NOT initialize with a README (you'll push from your local machine)
- Click **Create repository**

### 3b. Push from your Chromebook

Open your Linux terminal on the Chromebook. If you haven't configured Git yet:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-github-email@example.com"
```

Then navigate to where you saved the index.html file and initialize the repo:

```bash
# Create project directory if you haven't already
mkdir -p ~/insertfriction-site
# Copy your index.html into it (adjust source path as needed)
cp /path/to/your/index.html ~/insertfriction-site/

cd ~/insertfriction-site
git init
git add index.html
git commit -m "Initial landing page"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/insertfriction-site.git
git push -u origin main
```

GitHub will prompt for authentication. If you haven't set up a personal access token:
- Go to GitHub > **Settings** > **Developer Settings** > **Personal Access Tokens** > **Tokens (classic)**
- Generate a new token with `repo` scope
- Use this token as your password when Git prompts you

### 3c. Verify

Go to your repo on GitHub. You should see `index.html` in the file list.

---

## Step 4: Connect Netlify

### 4a. Log into Netlify

- Go to **https://app.netlify.com**
- Log in with your GitHub account (or existing Netlify account)

### 4b. Create new site from Git

- Click **Add new site** > **Import an existing project**
- Choose **GitHub** as your Git provider
- Authorize Netlify to access your repos if prompted
- Select your `insertfriction-site` repo

### 4c. Configure build settings

- **Branch to deploy:** main
- **Build command:** (leave blank — it's a static HTML file, no build needed)
- **Publish directory:** (leave blank or enter `.` — the index.html is in the root)
- Click **Deploy site**

### 4d. Verify the Netlify deploy

Netlify will give you a random URL like `https://random-name-12345.netlify.app`. Click it. You should see your landing page. Test the email form — make sure submissions appear in your Kit subscriber list.

---

## Step 5: Point Your Domain to Netlify

### 5a. Add custom domain in Netlify

- In your Netlify site dashboard, go to **Domain management** (or **Site settings** > **Domain management**)
- Click **Add custom domain**
- Enter `insertfriction.com`
- Also add `www.insertfriction.com` 
- Netlify will show you the DNS records you need to set

### 5b. Update DNS in Namecheap

- Log into **Namecheap** > **Domain List** > click **Manage** next to insertfriction.com
- Go to the **Advanced DNS** tab

**Option A: Use Netlify DNS (recommended — simplest)**
- In Netlify, it may prompt you to delegate DNS entirely to Netlify
- If so, Netlify gives you nameservers (e.g., `dns1.p01.nsone.net`)
- In Namecheap, go to the **Domain** tab (not Advanced DNS)
- Under **Nameservers**, select **Custom DNS**
- Enter the Netlify nameservers
- Save

**Option B: Keep Namecheap DNS, add records manually**
- In Namecheap Advanced DNS, add these records:
  - **A Record:** Host = `@`, Value = `75.2.60.5` (Netlify's load balancer IP — verify this in Netlify's docs as it may change)
  - **CNAME Record:** Host = `www`, Value = `your-site-name.netlify.app`
- Delete any conflicting A records or parking page records

### 5c. Enable HTTPS

- Back in Netlify > **Domain management** > **HTTPS**
- Click **Verify DNS configuration**
- Once DNS propagates (can take 5 minutes to 48 hours, usually under 30 minutes), click **Provision certificate**
- Netlify handles SSL automatically via Let's Encrypt

### 5d. Wait for DNS propagation

DNS changes can take anywhere from 5 minutes to 48 hours. Typically:
- Netlify DNS delegation: 15-30 minutes
- Manual A/CNAME records: 30 minutes to a few hours

You can check propagation at **https://www.whatsmydns.net** — enter `insertfriction.com` and see which DNS servers have updated.

---

## Step 6: Test Everything

### Final checklist before Saturday:

- [ ] **insertfriction.com loads** in Chrome, Safari, and on your phone
- [ ] **HTTPS works** (lock icon in browser, no security warnings)
- [ ] **Email signup works** — enter a test email, verify it appears in Kit
- [ ] **Mobile looks good** — check on your phone, have Jen or Dakota check on theirs
- [ ] **All links work** — the Family Throughlines link in the footer, the nav CTA scrolls to signup
- [ ] **The teen quote** — make sure you're comfortable with this being public and that it can't be attributed to a specific person
- [ ] **Speed** — the page should load fast (it's lightweight HTML, should be under 1 second)

### Quick fixes if something goes wrong:

**Form submissions not working?**
- Open browser dev tools (F12) > Console tab
- Submit the form and look for error messages
- Common issues: wrong form ID, API key missing, CORS errors
- If CORS is a problem with Kit's API, use their standard HTML form embed instead — Kit provides an embed code you can paste directly into the HTML, replacing the custom JavaScript form entirely

**Kit HTML embed fallback:**
If the JavaScript API approach gives you trouble, Kit provides a simple HTML form you can copy-paste. In Kit:
- Go to your form > **Publish** or **Share**
- Choose **HTML** embed
- Copy the code
- Replace the `<form>` section in your index.html with Kit's embed code
- Style it to match (you may need to override Kit's default styles)

**Domain not resolving?**
- Verify DNS records in Namecheap match what Netlify expects
- Wait longer — DNS propagation can be slow
- Try `nslookup insertfriction.com` in your terminal to check

**Page looks broken on mobile?**
- The CSS is responsive but test on actual devices, not just browser resize
- Common issue: viewport not set correctly (it's already set in the HTML, but verify `<meta name="viewport">` is present)

---

## Post-Launch: What to Do After Saturday

Once the page is live and the summit is done:

1. **Monitor Kit** for signups over the weekend. Note how many come in from the summit.
2. **Tag subscribers** — in Kit, you can tag subscribers. Consider creating tags for "educator-summit-feb2026" so you can segment this audience later.
3. **Send a welcome email** — Kit lets you set up an automated welcome sequence. Even a simple "Thanks for joining the Friction waitlist. Here's what we're building and why." would be valuable. You can set this up after Saturday.
4. **Begin the Substack post sequence** — Post 1 (Unimaginable Empowerment) can reference the waitlist and insertfriction.com at the end.
5. **Start the Chrome extension build** — now that the landing page is live, you can shift to product development.

---

## File Structure

Your repo should contain just one file for now:

```
insertfriction-site/
  index.html      <-- the landing page (everything in one file)
```

That's it. CSS and JavaScript are embedded in the HTML file. No build step, no dependencies, no complexity. As the site grows, you can break things out, but for now, one file is all you need.

---

**You've got this. Get it live, test it, and walk into Saturday with insertfriction.com in your pocket.**
