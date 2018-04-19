## Quickbet poc

Autocomplete option


This no work:

```html
<form autocomplete="on">
  <label for="uname"><b>Username</b></label>
  <input type="text" placeholder="Enter Username" name="uname" autocomplete="username-one">
  
  <label for="psw"><b>Password</b></label>
  <input type="password" placeholder="Enter Password" name="psw" autocomplete="password-one">
  <button type="submit">Login</button>
</form>  
<form autocomplete="on">
  <label for="uname2"><b>Username</b></label>
  <input type="text" placeholder="Enter Username" name="uname2" autocomplete="username-two">
  
  <label for="psw2"><b>Password</b></label>
  <input type="password" placeholder="Enter Password" name="psw2" autocomplete="password-two">
  <button type="submit">Login</button>
</form>
```

subdomain works fine

when served up via an iframe:
- both chrome and firefox remembers passwords correctly for separate subdomains, but doesn't fully auto complete. User needs to click on user/pass inputs (or start typing) and select their account from drop down