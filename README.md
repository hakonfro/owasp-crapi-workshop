# Oppgaver til OWASP Top 10 - OWASP crAPI

## Useful tools

- [Burp Suite](https://portswigger.net/burp)
- [Postman](https://www.postman.com/)
- [Docker](https://www.docker.com/)

## Setup

1. Download crAPI docker-compose file: `curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml`
2. Pull the images: `docker compose pull`
3. Start the containers: `docker compose -f docker-compose.yml --compatibility up -d`
4. Go to `http://localhost:8888`
5. The mail server is available at `http://localhost:8025`

### OWASP Top 10 API Security Risks and challenges

## BOLA

BOLA, or Broken Object-Level Authorization, is a type of security vulnerability that occurs when an application fails to properly enforce access controls at the object or data level. This can lead to unauthorized users gaining access to sensitive data or performing actions they should not be allowed to perform within the application.

[Read more about BOLA here](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)

### Challenge 1 — Access details of another user’s vehicle

To solve the challenge, you need to leak sensitive information of another user’s vehicle. Since vehicle IDs are not sequential numbers, but GUIDs, you need to find a way to expose the vehicle ID of another user.

Find an API endpoint that receives a vehicle ID and returns information about it.

<details><summary>Solution ⚠️</summary>
  
1. Find the endpoint that receives a vehicle ID and returns information about it.
2. Find another user's vehicle ID in the /community page
3. Make a request to the endpoint with the other user's vehicle ID

</details>

### Challenge 2 — Access mechanic reports of other users

crAPI allows vehicle owners to contact their mechanics by submitting a "contact mechanic" form. This challenge is about accessing mechanic reports that were submitted by other users.

- Analyze the report submission process
- Find an hidden API endpoint that exposes details of a mechanic report
- Change the report ID to access other reports

<details><summary>Solution ⚠️</summary>

1. Go to “Contact Mechanic” page and fill out the form
2. Inspect the request in the network tab
3. Find an hidden API endpoint that exposes details of a mechanic report
4. Change the report ID to access other reports

</details>

## Broken User Authentication

Broken user authentication is a critical security issue when an application fails to properly authenticate or authorize users, potentially allowing unauthorized individuals to access restricted resources or perform actions they shouldn’t have access to.

Read more about Broken User Authentication: https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/

#### Challenge 3 — Reset the password of a different user

Find an email address of another user on crAPI

Brute forcing might be the answer. If you face any protection mechanisms, remember to leverage the predictable nature of REST APIs to find more similar API endpoints.

<details><summary>Solution ⚠️</summary>
  
1. Go to Forgot Password” option on the login page
2. Receive a OTP code via email
3. change endpoint from v3 to v2. 
3. Reset the password of another user

</details>

## Excessive data exposure

Excessive data exposure is a critical security issue that occurs when sensitive information is unintentionally or improperly disclosed to unauthorized individuals or systems. It can have serious consequences for individuals and organizations, including data breaches, privacy violations, and legal ramifications.

Read more about Excessive data exposure: https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/

### Challenge 4 — Find an API endpoint that leaks sensitive information of other users

- explore the “Community” page and find information about other users

<details><summary>Solution ⚠️</summary>
  
1.Go to `/community`
2.Find an API endpoint that leaks sensitive information of other users in the response body
</details>

### Challenge 5 — Find an API endpoint that leaks an internal property of a video

- In this challenge, you need to find an internal property of the video resource that shouldn’t be exposed to the user. This property name and value can help you to exploit other vulnerabilities.
- Try to upload a video

<details><summary>Solution ⚠️</summary>

1. Go to `/video`
2. Try to upload a video
3. Inspect the response body and find an internal property of the video resource that shouldn’t be exposed to the user: `video_id`, `name`, `profileVideo`
</details>

## Rate Limiting

Rate limiting is a technique used in computer systems to control the rate at which requests or actions are allowed. It’s often implemented to prevent abuse, protect resources, and maintain system stability. Rate limiting can be applied in various contexts, such as API rate limiting, login attempts, or even in protecting against DDoS attacks.

Read more about Rate Limiting: https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/

### Challenge 6 — Perform a layer 7 DoS using ‘contact mechanic’ feature

- Try to contact a mechanic
- noticed the parameter “repeat_request_if_failed” was set to “false,” with “number_of_repeats” at a value of 1.

<details><summary>Solution ⚠️</summary>
  
1. Go to the Contact Mechanic’ feature, inspect the request in the network tab
2. Notice the parameter “repeat_request_if_failed” was set to “false,” with “number_of_repeats” at a value of 1.
3. Change the parameter to `true` and `1000`
4. Send the request
5. You will get a “Service unavailable. Seems like you caused a Layer 7 DoS :)”

</details>

## BROKEN FUNCTION LEVEL AUTHORIZATION (BFLA)

Broken Function Level Authorization (BFLA) is a security vulnerability that occurs when an application or system does not properly enforce access controls at the function or feature level. In other words, it allows users to perform actions or access features that they should not have permission to use.

Read more about Broken Function Level Authorization: https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/

### Challenge 7 — Delete a video of another user

Leverage the predictable nature of REST APIs to find an admin endpoint to delete videos

<details><summary>Solution ⚠️</summary>

1. Go to `/video`
2. Use the “Change Video Name” option to modify the name of the video.
3. Try to delete a video of another user
4. Notice that the method used was PUT. DELETE method was permitted.
5. Change the user to “admin”.

</details>

## Mass Assignment

Mass assignment is a security vulnerability that occurs when an attacker can manipulate input data to modify an object’s properties, often leading to unauthorized changes in a system. This can happen when developers don’t properly validate and sanitize user inputs or fail to restrict which properties can be modified in an object.

### Challenge 8 — Get an item for free

crAPI allows users to return items they have ordered. You simply click the "return order" button, receive a QR code and show it in a USPS store. To solve this challenge, you need to find a way to get refunded for an item that you haven’t actually returned.

Leverage the predictable nature of REST APIs to find a shadow API endpoint that allows you to edit properties of a specific order.


<details><summary>Solution ⚠️</summary>
- Explore the ‘shop’ page, where we noticed an available balance of $100 and two items: ‘Seat’ and ‘Wheel’.
- Placed an order and closely examined the request and response from the workshop API.
- Find the endpoint that allows you to edit properties of a specific order.
- Change the request method from GET to PUT.
- Modify the ‘status’ to ‘returned’ in the request body.

</details>

## Server-Side Request Forgery (SSRF)

SSRF is a type of security vulnerability that occurs when an attacker can manipulate the requests made by a web application to access resources on the server or other internal systems that they should not have access to.

Read more about Server-Side Request Forgery: https://owasp.org/API-Security/editions/2023/en/0xa7-server-side-request-forgery/

### Challenge 11 — Make crAPI send an HTTP call to “www.google.com" and return the HTTP response

- Use the “Send HTTP Request” option to send an HTTP request to “www.google.com” and return the HTTP response.

<details><summary>Solution ⚠️</summary>
1. Incpect the request from the contact mechanic form in Burp Suite.
2. Discover a URL that our request was using.
3. Modify the request URL. By substituting the URL, we successfully prompted the server to make an HTTP call to “www.google.com."

</details>

## NoSQL Injection

NoSQL Injection is a type of security vulnerability that occurs in applications that use NoSQL databases, such as MongoDB, Cassandra, or Redis when user-supplied data is not properly sanitized or validated before being used in database queries.

### Challenge 12 — Find a way to get free coupons without knowing the coupon code.

- Explore the ‘coupon’ page and find a way to get free coupons without knowing the coupon code.

<details><summary>Solution ⚠️</summary>
  
We initiated the challenge by intercepting the validate-coupon request in Burp Suite

1. Intercept the validate-coupon request in Burp Suite
2. Modify the request body to `{ "$ne": 1 }`

</details>

## SQL Injection

SQL Injection (SQLi) is a type of security vulnerability that occurs in web applications when user-supplied data is not properly validated or sanitized before being included in SQL queries. This allows malicious users to manipulate these queries to gain unauthorized access to a database or perform unintended actions on the database.

### Challenge 13 — Find a way to redeem a coupon that you have already claimed by modifying the database

- Explore the ‘coupon’ page and find a way to get free coupons without knowing the coupon code.

<details><summary>Solution ⚠️</summary>
  
1. Go to `/coupon`
2. Enter `admin` in the coupon code field
3. Inspect the request in the network tab
4. Notice the request is sent to `http://localhost:8080/WebGoat/start.mvc#test/webgoat.customjs.phoneHome()`

</details>

## Unauthenticated access

Unauthenticated access refers to allowing users or clients to interact with a system or application without requiring them to provide any form of authentication or identification. This means that users can access certain resources or perform certain actions without needing to log in or provide credentials.

### Challenge 14 — Find an endpoint that does not perform authentication checks for a user.

- Explore the ‘coupon’ page and find a way to get free coupons without knowing the coupon code.

<details><summary>Solution ⚠️</summary>
  
1. Go to `/coupon`
2. Enter `admin` in the coupon code field
3. Inspect the request in the network tab
4. Notice the request is sent to `http://localhost:8080/WebGoat/start.mvc#test/webgoat.customjs.phoneHome()`

</details>

## JWT Token

A JWT (JSON Web Token) is a compact, URL-safe means of representing claims to be transferred between two parties. It’s commonly used for authentication and authorization purposes in web applications. JWTs consist of three parts: the header, the payload, and the signature. The header and payload are Base64Url encoded JSON objects, and the signature is used to verify the integrity of the token.

### Challenge 15 — Forging Valid JWT Tokens for Full Access

JWT Authentication in crAPI is vulnerable to various attacks. Find any one way to forge a valid JWT token and get full access to the platform.

<details><summary>Solution ⚠️</summary>
  
1. Go to `/coupon`
2. Enter `admin` in the coupon code field
3. Inspect the request in the network tab
4. Notice the request is sent to `http://localhost:8080/WebGoat/start.mvc#test/webgoat.customjs.phoneHome()`

</details>
