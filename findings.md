# Quickbet poc

## Desired outcome

Allow users to store their username/passwords in their browsers password manager on a per-bookie basis to leverage browsers username/password autofill capabilities. This will ultimately increase user experience by decreasing the time it takes for a user to log into their chosen bookie.

### Standard autofill behaviour

The standard browser autofill behaviour for username/password fields is as follows:

- User lands on login page with no saved credentials
- User fills in username and password and submits form
- Browser prompts the user (some kind of popup) if they would like to save their details for this site
- If user accepts, the browser will automatically populate the username and password fields on subsequent visits to the login page 

## Option 1 - Using autocomplete tokens

This option involved using [autocomplete section tokens](https://wiki.whatwg.org/wiki/Autocomplete_Types#Section_tokens) so that the browser would be able to differentiate login forms at the same url. 

Example:

```html
<form autocomplete="on">
  <p>Bet365 Login</p>
  <label for="uname-bet365"><b>Username</b></label>
  <input type="text" placeholder="Enter Username" name="uname-bet365" autocomplete="uname-bet365">
  
  <label for="pword-bet365"><b>Password</b></label>
  <input type="password" placeholder="Enter Password" name="pword-bet365" autocomplete="pword-bet365">
  <button type="submit">Login</button>
</form>  
<form autocomplete="on">
  <p>Sportsbet Login</p>
  <label for="uname-sb"><b>Username</b></label>
  <input type="text" placeholder="Enter Username" name="uname-sb" autocomplete="uname-sb">
  
  <label for="pword-sb"><b>Password</b></label>
  <input type="password" placeholder="Enter Password" name="pword-sb" autocomplete="pword-sb">
  <button type="submit">Login</button>
</form>
```

### Verdict

This approach is **not feasible**. When tested in both Chrome and Firefox, the above approach did not work to spec. The browser didn't differentiate between the two forms (e.g. If the user had a Sportsbet username/password saved, the Bet365 login form would also be autofilled with the Sportsbet credentials)

## Option 2 - Serving login forms on different subdomains via iframe

This option involves hosting the login forms for quickbet on multiple subdomains (e.g. `bet365.quickbet.com.au`, `sportsbet.quickbet.com.au`) and serving the form to the user via an `<iframe>`. As the browser saves password on a domain basis, this would allow user's login experience to be less cumbersome as they can use browsers autofill features to speed up the process.

### Verdict

This option is **feasible** (tested in Chrome, Firefox and Safari), however there are a few caveats:

- When the page is served over `http` (as apposed to `https`), both browsers deviate from the [standard behaviour](#standard-autofill-behaviour) and will not automatically autofill the inputs for the user. The user will still be prompted to save their password when submitting it for the first time and can still access autocomplete functionality by either clicking or typing in the fields and selecting their credentials from a dropdown. This should not be a problem as we should be using `https` anyway.
- Even when served via `https`, Chrome still behaves as described in the above point when the content is displayed in an `<iframe>` (as is the proposed solution).
- Chrome requires a page redirect after form submission before it will prompt the user to save their credentials. This shouldn't be a problem but nice to know. More information on this can be found [here](https://stackoverflow.com/questions/2382329/how-can-i-get-browser-to-prompt-to-save-password) and [here](https://stackoverflow.com/questions/21191336/getting-chrome-to-prompt-to-save-password-when-using-ajax-to-login).
- Safari behaves in a similar way to Chrome however it won't prompt if `preventDefault` is called on form submit.
- This approach requires punters to configure a subdomain for each bookie.

## Implementing Option 2

- Vue.js app build and stored in `quickbet-ui` Amazon S3 Bucket.
- Establish multiple subdomains (Amazon Route 53) in the format `[BOOKIENAME].quickbet.puntapi.com`
    - Ideally will use `punters.com.au` domain but it's SSL certificate is not handled via AWS which makes it easier to use `puntapi.com` for the poc
- There is a restriction around Route 53 aliases pointing to S3 Buckets in that they must have an identical name (e.g. both called `sportsbet.quickbet.punters.com.au`). To get around this we need to put Cloudfront on top of the S3 bucket.
- Set up Cloudfront to point to S3 bucket's `index.html` with a 404 redirection to `index.html` (so an SPA works)
- Enter all required subdomains as Alternate Domain Names (CNAMEs) in Cloudfront.