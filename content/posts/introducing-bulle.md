---
title: "Introducing Bulle"
date: 2020-05-16T12:06:50+03:00
draft: true
---

_/bull-E/_

> Bulle is a modern solution for mocking HTTP API's.
>
> Bulle can be utilized to create mock API endpoints for development purposes or for general testing.

### Preface

I found myself developing applications where we had decided on the front-end technology, but were still contemplating on the backend technology.
In these situtations, we started work on the front-end nontheless, and as with almost any modern web application, the front-end
utilizes data acquired from the API. However we did not have a API built yet, nor had we decided what it should be written in.

In this situation we had 2 choices:

- Fill the front-end code with mocked JSON objects, which would act as they were the value acquired from the API
- Mocking an API, and fetching the data from it, like we would in the finalized product

I Dislike having my code filled with temporary javascript objects, which will have to be replaced with AJAX calls down the line,
and so decided to go with the latter approach.

However, I was not able to find many good tools for quickly mocking an API for the use cases I needed. Most of them usually required a
setup process, or a full on project strucured around them, and I found that as a huge waste of time and resources.

> In comes Bulle

After not finding a solution fitting for me, I started work on [Bulle](https://github.com/Matsuuu/Bulle).

The aim with Bulle was to create a tool that could be started in the matter of seconds from the command line,
but could also be expanded for larger testing.

---

## Introduction to Bulle

The main focus on Bulle was ease of use, and that is noticeable from the start.

To get started with Bulle, you just need to install it through `npm`

```bash
npm install -g bulle
```

Bulle being a cli tool written in Javascript, it requires [NodeJS](https://nodejs.org/en/) to be installed to be able to run.

#### Hello Bulle

We can now test out that Bulle functions properly setting it up with a single route.

```bash
bulle -r ping '{"message": "pong"}'
```

After running the command, we are greeted with the GUI for Bulle, displaying all of the endpoints mocked, port number of Bulle, and the
amount of requests each of the endpoints has received.

![Bulle Startup screen](/bulle-startup.png)

Now if we check out our localhost port 3000 (default address for Bulle), we should see the endpoint active.

```bash
# -v for more verbose example
curl -v http://localhost:3000/ping
```

We get a return value of

```bash
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 3000 (#0)
> GET /ping HTTP/1.1
> Host: localhost:3000
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: text/plain; charset=utf-8
< content-length: 19
< Date: Sat, 16 May 2020 11:04:22 GMT
< Connection: keep-alive
<
* Connection #0 to host localhost left intact
{"message": "pong"}
```

And just like that, we have a mocked API which returns JSON data.

Next let's look at some real world applications

---

## Bulle as the Developers' best friend

Let's create a common developer scenario:

> You are developing the user profile page, but do not have the backend API for the user profiles built yet.
>
> You, however want to create the data retrieval as a part of the page from the start, instead of just mocking the data in the code
> and worrying about the AJAX later on.

Let's say we have our normal user profile page. It will contain the following user information:

- User Name
- User email address
- User phone number
- Has the user subscribed to the email letter

Meaning that our data model would look something like

```javascript
{
  id: 1,
  name: "Foo Bar",
  email: "foobar@foo.bar",
  phone: "+123 555 555",
  subscribedToLetter: true
}
```

Let's see how we could go about creating the necessary API's for this page.

We would need a:

- Route for the user data (GET)
- Route for updating the user data (PUT)
- Route for deleting the user (DELETE)

We can create a mock API for this page extremely easily with Bulle.

```bash
bulle -p 1234 \
-r user/1 '{"id": 1, "name": "Foo Bar", "email": "foobar@foo.bar", "phone": "+123 555 555", "subscribedToLetter": true}' \
-r user PUT 204 'id=number;name=string;email=string;phone=string;subscribedToLetter=boolean' \
-r user/1 DELETE 202 '{"success": true}'
```

Let's go through the command line by line

## The Initialization

Here we just call Bulle, and specify the port to be 1234 instead of the default 3000

```bash
bulle -p 1234
```

## The GET route

First we specify the GET route. Since GET is the default method, and 200 is the default return code, those can be omitted.

We specify the return value of the request as the same JSON object we came up with earlier. This will be sent back to the
requester on a successful GET request.

```bash
-r user/1 '{"id": 1, "name": "Foo Bar", "email": "foobar@foo.bar", "phone": "+123 555 555", "subscribedToLetter": true}'
```

## The PUT route

Next we specify the PUT route, which is the route we use for updating the user's data.

When updating entities, we usually want to check that the entity field types match what we want. This is where we apply
the Bulle validation pattern.

```bash
-r user PUT 204 'id=number;name=string;email=string;phone=string;subscribedToLetter=boolean'
```

The pattern is a semicolon-seperated pattern of key-value pairs, where the key is the key of the entity we want to check,
and the value is the variable type we want the sent data to be.

For example here we want to make sure that the sent `id` is a number, `name`, `email` and `phone`
fields are strings and that the `subscribedToLetter` variable is a boolean.

On a invalid data type, Bulle will return with a response code `422 Unprocessable Entity`

If some of the specified fields are missing from the sent data, Bulle will return with a `500 Internal Server Error`

## The DELETE route

Last we specify the DELETE route. This one is a simple task of specifying a response code and a simple response body.

```bash
-r user/1 DELETE 202 '{"success": true}'
```

## Testing

Now if we test the API, on a GET request, we should get the user profile information.

![Bulle GET request result](/bulle-get-result.png)

Next up we can try out the update (PUT) route.

We can try out if our type checking works, but wrapping our boolean `subscribedToLetter` in quotes, making it a string
instead of a boolean.

![Bulle PUT getting a validation error](/bulle-put-error.png)

Seems that the validation works.

If we fix the error we should get a successful request through:

![Bulle PUT successful request](/bulle-put-success.png)

And we received a 204, which was the return value we wanted to see.

Last we can test out our delete path:

![Bulle DELETE succesful request](/bulle-delete-success.png)

And as we could have guessed, the DELETE path works just like it should.

## Verdict

By using Bulle, we were able to mock a simple API with different methods with just a single small command from the command line.

When used in real development, we would save a lot of time in refactoring, since we wouldn't need to rewrite our data
acquisition methods all over again after transferring from test data to a real API.

This was just a simple example of how to use Bulle in front-end development. Bulle is still a really early in development and
will be updated and expanded upon as new needs rise.

You can start using Bulle today by getting it from [npm](https://www.npmjs.com/package/bulle).

Contributions to the project are also welcome in the [Bulle Github Repository](https://github.com/Matsuuu/Bulle)
