## 📌 Objective

Host a static website using cloud object storage services across AWS S3, Azure Blob Storage, and/or GCP Cloud Storage. Configure permissions and enable public access.

---
### What is Static Website Hosting?

A **static website** consists of files (HTML, CSS, JavaScript, images) that are served directly to the browser without server-side processing. There is no backend code executing — what you upload is what the browser receives.

**Static vs Dynamic:**
```
Static Website:
  Browser → CDN/Object Storage → HTML/CSS/JS files returned
  ✅ No server required
  ✅ Extremely fast, cheap, scalable
  ✅ Examples: portfolios, documentation, landing pages

Dynamic Website:
  Browser → Web Server → Application Code → Database → HTML generated
  ✅ Personalized content, user auth, real-time data
  ✅ Examples: e-commerce, social media, SaaS apps
```

### Step 1 — Create an S3 Bucket

1. Go to **AWS Console → S3 → Create bucket**
2. Configure:
   ```
   Bucket name:       new-bucket-2026-erika (must be globally unique)
   AWS Region:        us-east-1
   Object Ownership:  ACLs disabled (recommended)
   
   Block Public Access settings:
     ❌ Uncheck "Block all public access"
     ✅ Acknowledge the warning checkbox
   
   Versioning:   Disabled (for simplicity)
   Encryption:   SSE-S3 (server-side encryption, default)
   ```
   
3. Click **Create bucket**

### Step 2 — Upload Website Files

**Create a sample website locally:**


`index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Cloud Website</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>🌩️ My Cloud-Hosted Website</h1>
        <p>This website is hosted on <strong>AWS S3</strong></p>
        <p>No servers needed — just pure cloud storage!</p>
        <a href="about.html">Learn More</a>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

`styles.css`:
```css
body {
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #667eea, #764ba2);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0;
}
.container {
    background: white;
    padding: 40px;
    border-radius: 15px;
    text-align: center;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
}
h1 { color: #333; }
a { color: #764ba2; text-decoration: none; font-weight: bold; }
```

*Upload via Console:**
```
S3 → Select bucket → Upload → Add files → Select all your files → Upload
```
*Upload via aws cli*
<img width="883" height="192" alt="image" src="https://github.com/user-attachments/assets/fb6239a6-80bb-47f9-9abe-6e040c343c40" />

<img width="1280" height="156" alt="image" src="https://github.com/user-attachments/assets/89bc1d08-bf8e-4427-9e0b-f24356586576" />


<img width="904" height="83" alt="image" src="https://github.com/user-attachments/assets/f301b8fa-e2cc-46d5-8736-e4c8d3118341" />

```
aws configure
AWS Access Key ID: <your-key>
AWS Secret Access Key: <your-secret>
Default region name: ap-south-1
Default output format: json
aws --version
aws s3 ls
cd ~/Downloads
aws s3 cp webserver.html s3://new-bucket-2026-erika/
aws s3 cp webserver.css s3://new-bucket-2026-erika/
aws s3 ls s3://new-bucket-2026-erika
pwd
ls
```
### Step 3 — Enable Static Website Hosting

```
S3 → Select bucket → Properties tab → Static website hosting →
  Edit → Enable
  Hosting type: Host a static website
  Index document: webserver.html

→ Save changes
```
### Step 4 — Configure Permissions (Bucket Policy)

```
S3 → Select bucket → Permissions tab → Bucket policy → Edit
```

Paste this policy (replace `my-static-website-2024` with your bucket name):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-static-website-2024/*"
        }
    ]
}
```
#### Access your website 


