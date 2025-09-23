---
tags:
  - htb
---
## Requests

Each HTTP request provides a method for accessing a resource. Though there are others, the two big ones are GET and POST. 

**GET** requests resources, and can pass data via `?=`. You can also pass authorization into it and use it as a (rather stupid) login system with the `Authorization` tag. We can specify tags with `-H` in curl. 

**POST** sends data (ex, uploading a file to a server). Instead of putting parameters in the head like GET, POST puts them in the body.  That way, less data has to be encoded as letters, and more data can be sent without clogging up the URL.  We can send POST requests via `-X` in curl, data with `-d`, and cookies with `-b`. 

### CRUD

CRUD APIs are VERY common, and have 4 operations: Create (POST reqs), Read (GET), Update (PUT), and Delete (DELETE). 

## Responses

Here's a nice little table of response codes. 

| Classes | Meanings                                                                                                                          |
| ------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1xx     | Provides information and does not affect the processing of the request                                                            |
| 2xx     | Returned when a request succeeds                                                                                                  |
| 3xx     | Returned when the server redirects the client                                                                                     |
| 4xx     | Signifies improper requests **from the client**. For example, requesting a resource that doesn't exist or requesting a bad format |
| 5xx     | Returned when there is some problem **with the HTTP server** itself                                                               |


