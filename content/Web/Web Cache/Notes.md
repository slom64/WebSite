The following entries might all be cached separately but treated as equivalent to `GET /` on the back-end:
- Apache: `GET //   `
- Nginx: `GET /%2F   `
- PHP: `GET /index.php/xyz   `
- .NET `GET /(A(xyz)/   `