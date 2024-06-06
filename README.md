# Vulnerable Web Application

The following code snipped, which represents a basic authentication form, has several vulnerabilities that need to be addressed as soon as possible.

## Vulnerabilities

### Local Storage - Credentials disclosure

Looking at the code, we see that the application relies on the browser's local storage to save the user credentials. This is bad practice because event after closing the server, the client's browser still remembers the credentials, which then can be used by treat actors to perform malicious activities.

```js
function login() {
    //const username = document.getElementById('username').value;
    //const password = document.getElementById('password').value;
    localStorage.setItem('username', username); 
    localStorage.setItem('password', password);
    //if (username === 'admin' && password === 'admin123') {
    //    alert('Login successful!');
    //} else {
    //    alert('Invalid credentials!');
    //}
    //document.body.innerHTML += `<p>Welcome, ${username}!</p>`;
}
```
!["Local Storage Vulnerability"](./captures/local_storage_vuln.png "Local Storage Vulnerability")

**Solution**: We can directly remove these two lines of code, because we usually don't want to store sensitive data, but if there is a particular reason to do so, we can rely on session storage instead, which mantains the information only during that particular session. When the user close the browser, all the information will be deleted.

```js
function login() {
    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;
    sessionStorage.setItem('username', username); 
    sessionStorage.setItem('password', password);
    if (username === 'admin' && password === 'admin123') {
       alert('Login successful!');
    } else {
       alert('Invalid credentials!');
    }
    document.body.innerHTML += `<p>Welcome, ${username}!</p>`;
}
```

### InnerHTML - Cross-Site Scripting

Another important vulnerability in this application is the use of the `.innerHTML` method. This function not only renders but also executes HTML in the browser, including JavaScript blocks.
In a real application this can be used by a malicious actor to inject more than just basic HTML or code. There are certain pentesting tools like BeEF that will assist a treat actor to mantain access in a user session using this technique.

```js
    document.body.innerHTML += `<p>Welcome, ${username}!</p>`;
```

To solve this issue we can do several things.
1. Use `.textContent` instead of `.innerHTML` is our intend is just to render text. If the user still tries to inject HTML it won't be executed, but it will be rendered anyway.
2. Parse and verify the user input before render it. This can be done with several Javascript libraries.
3. Use other DOM manipulation methods.

```js
// DOM manipulation
const newParagraph = document.createElement('p');
const sanitizedUserInput = sanitize(username)
newParagraph.appendChild(document.createTextNode(`Welcome, ${sanitizedUserInput}!`));

// Function to sanitize user input
function sanitize(string) {
  const map = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#x27;',
      "/": '&#x2F;',
  };
  const reg = /[&<>"'/]/ig;
  return string.replace(reg, (match)=>(map[match]));
}
```

