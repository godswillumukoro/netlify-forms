Netlify is awesome! You can deploy static websites, single-page applications, serverless functions, and more.

They have also made it easy to collect form submissions from your website. The [documentation page](https://docs.netlify.com/forms/setup) contains all you need to get started.

At the end of this article you will be able to:
1. Submit forms to your Netlify account
2. Setup spam protection
3. Get an HTTP response from the Netlify API using AJAX
4. Display custom success and error notification messages on the current web page

If any of these sound exciting to you, let's get started.

## Basic form Setup
```html
 <form name="Getting Started" method="POST" data-netlify="true" data-netlify-honeypot="bot-field">
    <input name="bot-field" class="dnone">
    <input type="text" name="firstname" id="fn" placeholder="First name" required>
    <input type="email" name="email" id="em" placeholder="Email address" required>
    <button type="submit">Get started</button>
</form>
```

`data-netlify="true"` This informs Netlify that you wish to receive submissions in your Netlify site admin panel.

`data-netlify-honeypot="bot-field"` This is a hidden spam protection form field that lures bot users into completing a field that human users can’t detect. Be sure to hide the corresponding `input` field using CSS. I hid mine in this example by declaring "display: none" within the `dnone` class.

At this point, you can submit forms to Netlify and view them in your site admin panel under the "Forms" tab

![Screenshot from 2022-12-04 18-43-33.png](https://res.cloudinary.com/dlwivxwf8/image/upload/v1670240500/Blog/netlify-form-blog/Screenshot_from_2022-12-04_18-43-33.png)

## Display Success or Error Messages
Next, we need a way to notify users about the status of their form submission. 
Was the form sent successfully or not? This is the question almost every person asks after filling out an online form. 

To achieve this, we will be using AJAX

```javascript
const form = document.querySelector('form')
const successElement = document.querySelector('#success');
const errorElement = document.querySelector('#error');

form.addEventListener('submit', event => {
    event.preventDefault();

    const myForm = event.target;
    const formData = new FormData(myForm);

    successElement.classList.add('dnone');
    errorElement.classList.add('dnone');

    fetch("/", {
        method: "POST",
        headers: { "Content-Type": "application/x-www-form-urlencoded" },
        body: new URLSearchParams(formData).toString(),
    })
        .then((response) => response.ok ? successElement.classList.remove('dnone') : errorElement.classList.remove('dnone'))
        .catch(() => errorElement.classList.remove('dnone'));
})
```
Using Javascript, we select the `form`, success, and error message HTML elements.

Next, we target the form to override the `form` submission event with JavaScript. To do this, we add a submit event listener to the `form` and prevent its default behavior with `event.preventDefault()`

Using the [FormData Interface](https://developer.mozilla.org/en-US/docs/Web/API/FormData), we were able to derive the values of every input in the `form`. We could skip this step if we were sending the HTML form directly to the server. However, in this case, we would be sending the form with JavaScript, so we build the body of the request using `formData`

Now we make a `POST` request to submit the form using the [global fetch method](https://developer.mozilla.org/en-US/docs/Web/API/fetch). It takes the resource URL we are submitting to. In the case of Netlify, that will be the current root page which can be simply represented by `/`.

The fetch method takes a second parameter to specify the `request method`, `headers`, and `body` of the request. We can pass `POST` as the request method in this case since it is a POST request. Netlify needs the content-type headers so we specify that as:
```
headers: { "Content-Type": "application/x-www-form-urlencoded" }
```

And finally, we can pass our body as the query string which was built with `FormData`
```
body: new URLSearchParams(formData).toString(),
```

The `Fetch API` will asynchronously process our request and return a response. 
We use the `.catch` to handle failures on the request, and the `.then` to handle a successful response. There's a possibility of getting an unsuccessful server response like a `4xx` or `5xx` error within the `.then` method, so we make sure to check for a `2xx` success response by using `response.ok`.
```javascript
.then((response) => response.ok ? successElement.classList.remove('dnone') : errorElement.classList.remove('dnone'))
```

## What it looks like
![Screenshot from 2022-12-05 11-55-26.png](https://res.cloudinary.com/dlwivxwf8/image/upload/v1670239945/Blog/netlify-form-blog/Screenshot_from_2022-12-05_11-55-26.png)

## Final Code

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Getting started with Netlify Forms</title>

    <style>
        #success {
            color: green;
        }

        #error {
            color: red;
        }

        .dnone {
            display: none;
        }
    </style>
</head>

<body>
    <form name="Getting Started" method="POST" data-netlify="true" data-netlify-honeypot="bot-field">
        <input name="bot-field" class="dnone">
        <input type="text" name="firstname" id="fn" placeholder="First name" required>
        <input type="email" name="email" id="em" placeholder="Email address" required>
        <button type="submit">Get started</button>

        <!-- sign up notification message -->
        <p id="success" class="dnone">Thanks! You’ve been registered successfully.</p>
        <p id="error" class="dnone">Sorry, we had an issue connecting to the server.</p>
    </form>

    <script>
        const form = document.querySelector('form')
        const successElement = document.querySelector('#success');
        const errorElement = document.querySelector('#error');

        form.addEventListener('submit', event => {
            event.preventDefault();

            const myForm = event.target;
            const formData = new FormData(myForm);

            successElement.classList.add('dnone');
            errorElement.classList.add('dnone');

            fetch("/", {
                method: "POST",
                headers: { "Content-Type": "application/x-www-form-urlencoded" },
                body: new URLSearchParams(formData).toString(),
            })
                .then((response) => response.ok ? successElement.classList.remove('dnone') :
                    errorElement.classList.remove('dnone'))
                .catch(() => errorElement.classList.remove('dnone'));
        })
    </script>
</body>

</html>
```

## Resources
Find the [Github Link](https://github.com/godswillumukoro/netlify-forms)

View the [Live Demo](https://netlify-forms-article.netlify.app/)

> Did you find this article helpful? 
Help spread the word so more people can get started with Netlify forms... and hey, check out my [Youtube videos](https://www.youtube.com/channel/UC2vJBc9AzljnTXVxnSK4qEw)
