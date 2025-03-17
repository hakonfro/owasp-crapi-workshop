# Oppgaver til OWASP Top 10 - OWASP crAPI

## Useful tools

- [Burp Suite](https://portswigger.net/burp)
- [Postman](https://www.postman.com/)
- [Docker](https://www.docker.com/)

## Setup
crAPI consists of several applications that can all be run as docker containers.
The file [docker-compose.yml](docker-compose.yml) is the configurations for running all applications as docker container.
Follow these steps to run the applications

1. Pull the images
```shell
docker compose pull
```
2. Start the containers
```shell
docker compose -f docker-compose.yml --compatibility up
```
3. Go to http://localhost:8888
4. The mail server is available at http://localhost:8025

The containers and the ports they are running on can be viewed by running 
```shell
docker ps -a
```

# OWASP Top 10 API Security Risks and Challenges

## Broken Object-Level Authorization (BOLA)

BOLA, or Broken Object-Level Authorization, is a type of security vulnerability that occurs when an application fails to properly enforce access controls at the object or data level. This can lead to unauthorized users gaining access to sensitive data or performing actions they should not be allowed to perform within the application.

[Read more about BOLA here](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)

### Challenge 1 — Access details of another user’s vehicle

To solve the challenge, you need to leak sensitive information of another user’s vehicle. Since vehicle IDs are not sequential numbers, but GUIDs, you need to find a way to expose the vehicle ID of another user.

Find an API endpoint that receives a vehicle ID and returns information about it.

<details><summary>Hint 💡</summary>
Try adding a vehicle to explore the details that can be disclosed.
</details>

<details><summary>Solution ⚠️</summary>

1. Clicking "Refresh location" sends a GET request to `/identity/api/v2/vehicle/<VEHICLE ID/location>`
2. Find another user's vehicle ID in the `/community` page
3. Make a request to the endpoint found earlier with another user's vehicle ID

</details>

### Challenge 2 — Access mechanic reports of other users

This challenge is about accessing mechanic reports that were submitted by other users.

<details><summary>Hint️ 💡</summary>
crAPI allows vehicle owners to contact their mechanics by submitting a "contact mechanic" form. Submit such a form and analyze the report submission process.
</details>

<details><summary>Solution ⚠️</summary>

1. Go to “Contact Mechanic” page and fill out the form
2. Inspect the request sent when submitting the form. The response includes a response from the mechanic API,
   containing a `report_link` with the format `/workshop/api/mechanic/mechanic_report?report_id=<REPORT ID>`
4. Change the report ID to access reports of others users

</details>

## Broken User Authentication

Broken user authentication is a critical security issue when an application fails to properly authenticate or authorize users, potentially allowing unauthorized individuals to access restricted resources or perform actions they shouldn’t have access to.

Read more about Broken User Authentication: https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/

### Challenge 3 — Reset the password of another user

Try to reset you password, and look for possible exploits in the password reset procedure.

<details><summary>Hint 💡</summary>
The endpoint for OTP-check implements rate-limiting, but the endpoint is in v3. Does all API versions implement this?🤔
</details>

<details><summary>Solution ⚠️</summary>

1. Go to Forgot Password” option on the login page
2. Receive a OTP code via email (the mail server is available at `http://localhost:8025`)
3. Intercept the requests that verifies OTP: `/identity/api/auth/v3/check-otp`
3. Change endpoint from v3 to v2: `/identity/api/auth/v2/check-otp`
4. Brute-force the endpoint and iterate through all possible 4-digit OTPs. Burp Suite's Intruder tool is a good option for this.
3. Reset the password of another user.

</details>

## Excessive data exposure

Excessive data exposure is a critical security issue that occurs when sensitive information is unintentionally or improperly disclosed to unauthorized individuals or systems. It can have serious consequences for individuals and organizations, including data breaches, privacy violations, and legal ramifications.

Read more about Excessive data exposure: https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/

### Challenge 4 — Find an API endpoint that leaks sensitive information of other users

<details><summary>Hint 💡</summary>
Explore the “Community” page.
</details>
<details><summary>Solution ⚠️</summary>

1. Go to "Community" page and intercept the request to `/community/api/v2/community/posts/recent`
2. Inspect the response and find sensitive information about other users.
</details>

### Challenge 5 — Find an API endpoint that leaks an internal property of a video

In this challenge, you need to find an internal property of the video resource that shouldn’t be exposed to the user. This property name and value can help you to exploit other vulnerabilities.

<details><summary>Hint 💡</summary>
Try to upload a video and intercept the request.
</details>

<details><summary>Solution ⚠️</summary>

1. Go to `/video`
2. Try to upload a video
3. Intercepting the request to `/identity/api/v2/user/videos` shows that the response contains internal properties of the video resource
   like **id**, **video_name**, **conversion_params** and **profileVideo**.
</details>

## Rate Limiting

Rate limiting is a technique used in computer systems to control the rate at which requests or actions are allowed. It’s often implemented to prevent abuse, protect resources, and maintain system stability. Rate limiting can be applied in various contexts, such as API rate limiting, login attempts, or even in protecting against DDoS attacks.

Read more about Rate Limiting: https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/

### Challenge 6 — Perform a layer 7 DoS using ‘contact mechanic’ feature

A Layer 7 Denial of Service (DoS) attack, often referred to as an application layer DoS attack, is a type of cyberattack that specifically targets the application layer of the OSI model. In this type of attack, the attacker aims to overwhelm a web server or application by sending a large volume of malicious requests that are designed to consume server resources, exhaust server capacity, or exploit vulnerabilities in the application.

<details><summary>Hint 💡</summary>
Try to explore the "Contact Mechanic" feature.
</details>

<details><summary>Solution ⚠️</summary>

1. Go to the "Contact Mechanic" feature and fill out a form for assistance.
2. Intercept the POST-request
3. The request body indicates that clients can specify whether they want the request to be repeated and, if so, how many times.
2. Notice the parameter “repeat_request_if_failed” was set to “false,” with “number_of_repeats” at a value of 1.
3. Change the parameters `repeat_request_if_failed` and `number_of_repeats` to respectively `true` and `10000`.
4. Sending the requests will return the response “Service unavailable. Seems like you caused a Layer 7 DoS :)”.

</details>

## Broken Function Level Authorization (BFLA)

Broken Function Level Authorization (BFLA) is a security vulnerability that occurs when an application or system does not properly enforce access controls at the function or feature level. In other words, it allows users to perform actions or access features that they should not have permission to use.

Read more about Broken Function Level Authorization: https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/

### Challenge 7 — Delete a video of another user

Leverage the predictable nature of REST APIs to find an admin endpoint to delete videos.

<details><summary>Hint 💡</summary>
Attempt to rename the video and examine the HTTP verb used in the request.
</details>

<details><summary>Solution ⚠️</summary>

1. Try to change the name of a video. This sends a PUT request to `/identity/api/v2/user/videos/<VIDEO ID>`.
2. Send an OPTIONS request to `/identity/api/v2/user/videos/<VIDEO ID>` and observe allowed HTTP verbs in the response.
2. Try to send an empty DELETE request to `/identity/api/v2/user/videos/<VIDEO ID>`, which indicates that there exists an admin API.
3. Send an empty DELETE request to `/identity/api/v2/admin/videos/<VIDEO ID>` with a video ID of your choice.

</details>

## Mass Assignment

Mass assignment is a security vulnerability that occurs when an attacker can manipulate input data to modify an object’s properties, often leading to unauthorized changes in a system. This can happen when developers don’t properly validate and sanitize user inputs or fail to restrict which properties can be modified in an object.

### Challenge 8 — Get an item for free

crAPI allows users to return items they have ordered. You simply click the "return order" button, receive a QR code and show it in a USPS store. To solve this challenge, you need to find a way to get refunded for an item that you haven’t actually returned.

<details><summary>Hint 💡</summary>
Leverage the predictable nature of REST APIs to find a shadow API endpoint that allows you to edit properties of a specific order.
</details>

<details><summary>Solution ⚠️</summary>

1. Explore the ‘shop’ page, where we noticed an available balance of $100 and two items: ‘Seat’ and ‘Wheel’.
2. Place an order and closely examined the request and response from the workshop API.
3. Navigate to past orders, view order details and intercept the request.
4. This exposes a `status` field for each order along with all permitted HTTP verbs.
5. Change the verb from GET to PUT and send the following request body: `{"status": "returned"}`.
   This will "refund" the order and the refund will be added to your account's credit balance.

</details>

### Challenge 9 — Increase your balance by $1,000 or more

<details><summary>Hint 💡</summary>
Leverage the predictable nature of REST APIs to find a shadow API endpoint that allows you to edit properties of a specific order.
</details>

<details><summary>Solution ⚠️</summary>
This challenge exploits the same vulnerable API found in challenge 8.

1. Explore the ‘shop’ page, where we noticed an available balance of $100 and two items: ‘Seat’ and ‘Wheel’.
2. Place an order and closely examined the request and response from the workshop API.
3. Navigate to past orders, view order details and intercept the request.
4. This exposes a `status` and `quantity` field for each order along with all permitted HTTP verbs.
5. Change the verb from GET to PUT and send the following request body: `{"quantity": 100, "status": "returned"}`.
   This will "refund" the order and the refund*100 will be added to your account's credit balance.

</details>

### Challenge 10 — Update internal video properties

<details><summary>Hint 💡</summary>
Try to upload a video and intercept the request.
</details>

<details><summary>Solution ⚠️</summary>

1. Go to `/video`
2. Try to upload a video
3. Intercepting the request to `/identity/api/v2/user/videos` shows that the response contains internal properties of the video resource
   like **id**, **video_name**, **conversion_params** and **profileVideo**.
4. Alter the PUT request to update internal video properties: `{"videoName": <NEW NAME>, "conversion_params": <NEW INTERNAL VIDEO PROPERTY>}`.
</details>

## Server-Side Request Forgery (SSRF)

SSRF is a type of security vulnerability that occurs when an attacker can manipulate the requests made by a web application to access resources on the server or other internal systems that they should not have access to.

Read more about Server-Side Request Forgery: https://owasp.org/API-Security/editions/2023/en/0xa7-server-side-request-forgery/

### Challenge 11 — Make crAPI send an HTTP call to “www.google.com" and return the HTTP response

Use the “Send HTTP Request” option in the "Contact Mechanic" form to send an HTTP request to “www.google.com” and return the HTTP response.

<details><summary>Hint 💡</summary>
Inspect the request body sent when submitting the "Contact Mechanic" form. 
</details>

<details><summary>Solution ⚠️</summary>

1. Incpect the request when submitting the "Contact Mechanic" form.
2. Observe that `mechanic_api` is sent by the client in the request.
3. Modify the request URL. By substituting the URL, we successfully prompted the server to make an HTTP call to “www.google.com."

</details>

## NoSQL Injection

NoSQL Injection is a type of security vulnerability that occurs in applications that use NoSQL databases, such as MongoDB, Cassandra, or Redis when user-supplied data is not properly sanitized or validated before being used in database queries.

### Challenge 12 — Find a way to get free coupons without knowing the coupon code.

Explore the ‘coupon’ page and find a way to get free coupons without knowing the coupon code.

<details><summary>Hint 💡</summary>
Try to modify the request body sent when validating coupon codes. 
</details>

<details><summary>Solution ⚠️</summary>

We initiated the challenge by intercepting the validate-coupon request to `/community/api/v2/coupon/validate-coupon`

1. Intercept the validate-coupon request in Burp Suite
2. Modify the request body to `{"coupon_code": { "$ne": 1 }}` and get a free coupon code.

</details>

## SQL Injection

SQL Injection (SQLi) is a type of security vulnerability that occurs in web applications when user-supplied data is not properly validated or sanitized before being included in SQL queries. This allows malicious users to manipulate these queries to gain unauthorized access to a database or perform unintended actions on the database.

### Challenge 13 — Find a way to retrieve information about database version used

This challenge assumes you already completed Challenge 13 and obtained a free coupon code.

<details><summary>Hint 💡</summary>
Try to use the coupon code and modify the request body with SQLi exploit payloads. 
</details>

<details><summary>Solution ⚠️</summary>

1. Use the coupon code and intercept the POST request sent to `/workshop/api/shop/apply_coupon`
2. Manipulate the `coupon_code` in the request body to `0' or '0' = '0`. The response indicates that the provided `coupon_code` is not sanitized.
3. Manipulate the `coupon_code` in the request body to `0'; select version() --+`, which returns the Postgres version used.

</details>

## Unauthenticated access

Unauthenticated access refers to allowing users or clients to interact with a system or application without requiring them to provide any form of authentication or identification. This means that users can access certain resources or perform certain actions without needing to log in or provide credentials.

### Challenge 14 — Find an endpoint that does not perform authentication checks for a user.

<details><summary>Hint 💡</summary>
Browse the web page and look for endpoints that responds even though no bearer token was supplied. 
</details>

<details><summary>Solution ⚠️</summary>

1. Intercept the "Order Details" request.
2. Resend the GET request to `/workshop/api/shop/orders/<USER ID>` without bearer token and observe that you still get a response from the server.

</details>

## JWT Token

A JWT (JSON Web Token) is a compact, URL-safe means of representing claims to be transferred between two parties. It’s commonly used for authentication and authorization purposes in web applications. JWTs consist of three parts: the header, the payload, and the signature. The header and payload are Base64Url encoded JSON objects, and the signature is used to verify the integrity of the token.

### Challenge 15 — Forging Valid JWT Tokens for Full Access

JWT Authentication in crAPI is vulnerable to various attacks. Find any one way to forge a valid JWT token and get full access to the platform.

<details><summary>Solution ⚠️</summary>

crAPI is vulnerable to the following JWT vulnerabilities:

1. **JWT Algorithm Confusion Vulnerability**
    - crAPI uses the RS256 JWT algorithm by default.
    - The public key to verify JWT is available at `http://localhost:8888/.well-known/jwks.json`.
    - Convert the public key to a Base64-encoded form and use it as a secret to create a JWT in the HS256 algorithm.
    - This JWT will be accepted as a valid JWT token by crAPI.

2. **Invalid Signature Vulnerability**
    - The User Dashboard API is not validating the JWT signature.
    - Create a JWT with the `sub` header set to a different user's email.
    - With the above JWT, you will be able to extract user data from the User Dashboard API endpoint.

3. **JKU Misuse Vulnerability**
    - crAPI will verify a JWT token with any public key that is pointed to by the `jku` JWT header.
    - Create your own public/private key pair and sign a JWT using the RS256 algorithm.
    - Host the public key somewhere in JWK format.
    - Pass the public key URL in the `jku` header of the JWT with an appropriate `kid` header.
    - This JWT will be accepted as a valid JWT token by crAPI.

4. **KID Path Traversal Vulnerability**
    - Set the `kid` header of the JWT to `../../../../../../dev/null`.
    - Create a custom JWT using the HS256 algorithm with the secret as `AA==`.
    - `AA==` is the Base64-encoded form of the hex null byte `00`.
    - This JWT will be accepted as a valid JWT token by crAPI.

</details>
