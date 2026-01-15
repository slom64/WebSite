1. XSS as usual
2. Open-redirect to different website
3. Change cookie.
4. Like regular xss you may escape `<script>let x = 'controlled'</script>` inside html file which enable injecting javascript commands.
5. Changing `document.domain` which enable us to bypass `SOP` by putting it as our domain or as vulnerable subdomain on the target domain.
6. Change the websocket URL, which may make the user send sensitive data to us, or sending malicious responses that triggers XSS to the client.
7. If you can change one of those you can bypass Anti-XSS defenses, `element.href`,`element.src`,`element.action` .