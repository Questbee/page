# Questbee — Marketing Site

Static landing page and documentation for the Questbee project.

---

## Contents

```
web-page/
├── index.html          # Landing page
├── styles.css          # Site styles
├── images/             # Logo and icons
└── docs/               # HTML documentation
    ├── index.html          # Docs hub
    ├── docs.css            # Shared docs styles
    ├── getting-started.html
    ├── deployment.html
    ├── mobile-app.html
    ├── architecture.html
    └── api-reference.html
```

## Local preview

```bash
cd web-page
python3 -m http.server 8080
# Open http://localhost:8080
```

## Related repos

| Repo | Purpose |
|---|---|
| [Questbee/community](https://github.com/Questbee/community) | Server platform (backend + dashboard) — MIT open source |
| [Questbee/app](https://github.com/Questbee/app) | Android APK releases |
