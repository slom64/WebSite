```
my-node-app/
├── node_modules/         # Dependencies (auto-generated)
├── public/               # Static files
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
│       └── logo.png      # Served as /images/logo.png
├── routes/
│   ├── index.js          # Home route (/)
│   └── users.js          # /users endpoints
├── controllers/
│   └── userController.js # Logic for users
├── models/
│   └── user.js           # DB schema
├── views/
│   └── index.ejs         # Template for /
├── config/
│   └── db.js             # Database connection
├── .env
├── .gitignore
├── app.js                # Main app setup
├── package.json
└── README.md
```