Use macros for session management and to make the session ready before making requests.
https://www.youtube.com/watch?v=J8Ykn4Pa2bc

## Steps
- Add a macro.
	1. Macros
	2. Add :get what request you need to be set before sending the request
	3.  configure item, `use this when you need to handle CSRF or other parameters, BUT session cookies are handled without using this.`
	4. select the value you want
- Make a rule that runs the macro you have created.


> [!NOTE] Unexpected parameter changing
> If there is parameter getting changed that you don't want it get changed, you can specify execulde it from the settings
