# Questbee Website

This folder contains the **questbee.io** website—a professional, multi-purpose platform designed to serve:
* The **Community** (open-source developers)
* **Enterprise/Licensing** information
* **Partner** engagement and pilot programs

## 📁 Structure

```
web-page/
├── index.html          # Main website (all-in-one single page)
├── styles.css          # Responsive styling
├── README.md           # This file
└── docs/              # (Optional) Documentation pages
    ├── getting-started.md
    ├── architecture.md
    ├── api-reference.md
    └── deployment.md
```

## 🚀 Quick Start

1. **Local Development**
   ```bash
   # Simply open the file in a browser
   open index.html
   
   # Or serve locally (Python):
   python -m http.server 8000
   # Then visit: http://localhost:8000
   ```

2. **Deploy to Web**
   * Upload `index.html` and `styles.css` to any web host.
   * Update links in the HTML to point to your actual GitHub repo, documentation, and contact email.
   * Replace logo paths if needed (currently points to `../images/` folder).

## 🎨 Customization

### Update Links
* **GitHub:** Change all `href="https://github.com"` to your actual repo URL.
* **Email:** Update `mailto:hello@questbee.io` and `mailto:partnerships@questbee.io`.
* **Docs:** Point to your documentation hosting (GitHub Pages, ReadTheDocs, etc.).

### Change Colors
Edit the CSS variables in `styles.css`:
```css
:root {
    --color-primary: #2563eb;      /* Main brand color */
    --color-primary-dark: #1e40af;
    --color-primary-light: #dbeafe;
    /* ... other colors ... */
}
```

### Add Images
* Replace or enhance the placeholder mockups (`community-image`, `partners-image`).
* Current images are sourced from `../images/` folder.

## 📄 Page Sections

### Hero Section
* Value proposition
* Call-to-action buttons (Get Started, Join Community)

### Features
* 6 key differentiators with icons

### Community Edition
* Free, open-source option
* Quick setup instructions
* GitHub link

### Licensing & Enterprise Plans
* 3 pricing tiers: Community, Enterprise, Implementation Services
* Feature matrix

### Partner Program
* Benefits of partnership
* Ideal partner profiles (foundations, consultancies, integrators)
* Revenue share information

### Use Cases
* Real-world applications (Government, NGO, Enterprise, Data Science)

### Documentation
* Links to guides (Getting Started, Architecture, Mobile, API, Deployment)
* Partner guides

### Call-to-Action
* Final push for conversion

### Footer
* Navigation links
* Social / company links
* Legal links (to be added)

## 🔗 Navigation Structure

The site uses a single-page design with smooth scrolling:
- **#features** → Features section
- **#community** → Community Edition
- **#licensing** → Pricing plans
- **#partners** → Partner program
- **#docs** → Documentation

## 📱 Responsive Design

The website is fully responsive and mobile-friendly, adapting to:
* Desktop (1200px+)
* Tablet (768px – 1200px)
* Mobile (<768px)

## 🔐 Security & Privacy

* No external tracking (no GA, no Mixpanel).
* No form submissions (recommend integrating with a service like Formspree, Netlify Forms, or AWS SES).
* Static HTML (can be served from a CDN for high performance).

## 📞 Contact / Conversion

Current contact points:
* **General:** `hello@questbee.io`
* **Partnerships:** `partnerships@questbee.io`

Consider integrating:
* Email capture for newsletter
* Live chat for quick questions
* Scheduled demo booking

## 🚀 Hosting Options

### Free / Low-Cost
* **Netlify** — Drag-and-drop deployment, free HTTPS, CDN
* **GitHub Pages** — Host directly from your GitHub repo
* **Vercel** — Optimized for static sites, very fast

### Self-Hosted
* Any Linux server with a simple `python -m http.server` or Nginx
* Docker: `docker run -d -p 80:8000 -v $(pwd):/app python:3 -m http.server -d /app`

## ✏️ To-Do / Future Enhancements

- [ ] Add blog/case studies section
- [ ] Integration with email capture (newsletter)
- [ ] Dark mode toggle
- [ ] Multi-language support
- [ ] Team member bios / About page
- [ ] Live pricing calculator
- [ ] Testimonials section
- [ ] FAQ section
- [ ] Demo video embed

---

**Last Updated:** 2026-03-16  
**Maintained by:** Questbee Team
