# Different ideas
- Set UID. [[Cap]]
	- when we did `print(os.popen("whoami").read())`, we got the normal user even we have setuid on python. Because almost all **modern interpreters disable honoring SUID/SGID bits** for security reasons. so you need to enable it using `os.setuid(0)`
