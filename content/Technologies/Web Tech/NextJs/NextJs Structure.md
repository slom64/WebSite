```
my-nextjs-app/
├── app/                  # App Router
│   ├── layout.js         # Root layout (wraps all pages)
│   ├── page.js           # Homepage (/)
│   ├── blog/             # Nested route (/blog)
│   │   ├── page.js       # Blog list
│   │   └── [slug]/       # Dynamic post (/blog/my-post)
│   │       └── page.js
│   ├── api/              # Not standard here; use route.js for APIs
│   ├── _components/      # Private components (not routable)
│   └── (auth)/           # Route group (e.g., for auth pages without URL prefix)
│       └── login/
│           └── page.js   # /login
├── pages/                # Pages Router (alternative or hybrid)
│   ├── index.js          # Homepage (/)
│   ├── about.js          # /about
│   └── api/
│       └── hello.js      # /api/hello
├── public/
│   ├── images/
│   │   └── logo.png      # /images/logo.png
│   └── favicon.ico       # /favicon.ico
├── components/
│   └── Header.js         # Reusable component
├── styles/
│   └── globals.css
├── next.config.js
├── package.json
├── .env.local
├── .env
└── README.md
```