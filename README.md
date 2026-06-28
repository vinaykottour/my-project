# My AWS Project

## 👤 About
This is a personal portfolio website for **Vinay Kumar**, Site Reliability Engineer.
Hosted on AWS EC2 using Nginx web server.

## 🌐 Live Website
Access the site at: http://YOUR-EC2-PUBLIC-IP

## 📁 Project Structure
## 📄 index.html Details
- **File**: `index.html`
- **Location on EC2**: `/var/www/html/index.html`
- **Purpose**: Personal portfolio page served by Nginx
- **Contains**:
  - Navigation bar
  - Hero section with name and title
  - About Me section
  - Skills section (AWS Cloud, SRE, DevOps, Cloud Security)
  - Contact section with email
  - Footer

## 🛠️ Tech Stack
| What | Tool |
|------|------|
| Cloud Provider | AWS EC2 |
| Web Server | Nginx |
| Frontend | HTML, CSS |
| Code Storage | GitHub |
| OS | Ubuntu (t3.micro) |

## 🚀 How to Deploy
1. SSH into EC2 instance
2. Copy index.html to Nginx web root:
```bash
   sudo cp index.html /var/www/html/index.html
```
3. Restart Nginx:
```bash
   sudo systemctl restart nginx
```
4. Open browser and visit your EC2 public IP

## 📬 Contact
- **Email**: vinaykottour@gmail.com
- **Role**: Site Reliability Engineer
