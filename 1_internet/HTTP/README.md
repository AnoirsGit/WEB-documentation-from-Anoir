# What is HTTP?
The Hypertext Transfer Protocol (HTTP) is the foundation of the World Wide Web and is used to load web pages using hypertext links. HTTP is an application layer protocol designed to transfer information between networked deviced and runs on top of other layers of the network protocol stack. A typical flow over HTTP involves a client machine making a request to a server, which then sends sa reslponse message.

## What is in an HTTP request?
An HTTP request is the way internet communications platforms such as web browsers ask for the information they need to load a website.

Each HTTP request made across the Internet carries with it a series of encoded data that carries different types of information. A typical HTTP request contains:  
  1. HTTP version type
  2. a URL
  3. an HTTP method
  4. Headers
  5. Optional HTTP body

## HTTP request Header
HTTP headers contain text information stored in key-value pairs, and they are included in every HTTP request. These Headers communicae core information, such as what browser the clit is using and info about data 

![image](https://user-images.githubusercontent.com/49281851/176595768-13f736fa-040d-47b9-bc31-72701ee5a808.png)


## What’s in an HTTP request body?
The body of a request is the part that contains the ‘body’ of information the request is transferring. The body of an HTTP request contains any information being submitted to the web server, such as a username and password, or any other data entered into a form.

## What’s in an HTTP response?
An HTTP response is what web clients (often browsers) receive from an Internet server in answer to an HTTP request. These responses communicate valuable information based on what was asked for in the HTTP request.

A typical HTTP response contains:
1. an HTTP status code
2. HTTP response headers
3. optional HTTP body
4. Let's break these down:

## What’s an HTTP status code?
HTTP status codes are 3-digit codes most often used to indicate whether an HTTP request has been successfully completed. Status codes are broken into the following 5 blocks:

 - 1xx Informational
 - 2xx Success
 - 3xx Redirection
 - 4xx Client Error
 - 5xx Server Error

## What are HTTP response headers?
HTTP response comes with headers that convey important information such as the languageand format of the data being sent in the response body.

<img width="550" alt="image" src="https://user-images.githubusercontent.com/49281851/176595835-2bc9a0d0-ab2b-402e-ba0e-c57db954250d.png">
