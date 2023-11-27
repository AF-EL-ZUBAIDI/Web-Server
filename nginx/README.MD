# Nginx Web Server Tutorial

This tutorial provides step-by-step instructions for setting up and running an Nginx web server on macOS.

## Introduction

Nginx is a high-performance web server that is widely used for serving web content. In this tutorial, we will guide you through the process of installing and configuring Nginx on your Mac.

## Nginx Installation

To get started with Nginx, follow these steps:

1. **Install Nginx** using Homebrew:
```bash
brew install nginx
```

2. Navigate to the Nginx installation directory (the default location):
```bash
cd /usr/local/etc/nginx
```

3. Open the Nginx configuration directory in Visual Studio Code:
```bash
code .
```

4. Start Nginx:
```bash
nginx
```

5. Open your web browser and visit the following URL to check if the server is running correctly:

```bash
http://localhost:8080/
```

If you encounter issues running code ., you can create a symbolic link to Visual Studio Code:

```bash
sudo ln -s /Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code /usr/local/bin/
```

### Nginx Terminology

Before we dive into configuring Nginx, let's briefly explain some key terminology:

1. Directives: These are key-value pairs used in the Nginx configuration, such as:
```bash
worker_processes  1;
```

2. Contexts: Blocks that contain multiple directives are referred to as contexts. For example:
```bash
events {
    worker_connections  1024;
}
```

### Serving Static Content
To serve static content with Nginx, follow these steps:

1. Modify the nginx.conf file to serve HTML pages. Here is the code snippet to add:
```bash
http {
    server {
        listen 8080;
        root /path-to-your-folder/mysite;
    }
}
event {}
```
Make sure to change the root path to your actual project path. Nginx will serve files from the "mysite" folder, and it looks for an "index.html" file by default.


2. Reload Nginx after making changes to the configuration:
```bash
nginx -s reload
```

3. To ensure that Nginx can serve various file types in your project, include the "mime.types" file in the configuration:
```bash
include mime.types
```

### Location Contexte

Location contexts allow you to serve different HTML files for specific URLs. Here's how to use them:

1. To serve content at a specific URL, add a location block:

```bash
location /fruits {
    root /another/loction;
}
```

2. To serve content with a different URL ending, use the "alias" keyword in the location block:
```
location /carbs {
    alias /your/path/mysite/fruits;
}
```

3. To use a custom HTML filename (not "index.html"), use "try_files" to search for the corresponding file:
```bash
location /vegetables {
    alias /your/path/mysite;
    try_files /vegetables/veggies.html /index.html =404;
}
```
##### How it work?

try_files need 3 arguments:
```bash
try_files arg1 arg2 arg3;
```

* arg1 → Search for the HTML file
* arg2 → If the file doesn’t exist then open the index.html file (the standard one)
* arg3 → If it also failed, throw a 404 error

4. To use regular expression:
```bash
location ~* /count/[0-9] {
    root /your/path/mysite;
    try_files /index.html =404;
}
```

### Redirect and Rewrites (advanced)

Redirects and rewrites are used for URL manipulation:

1. To redirect a user, use the code 307 followed by the target page:
```bash
location /crops {
    return 307 /fruits;
}
```

2. To rewrite a URL:
```bash
rewrite ^/number/(\w+) /count/$1;
```
* \w+ → Mean the variable
* $1 → It’s the variable

##### The difference between the redirect and the rewrite is:

- Redirect: change the URL to the page displayed
- Rewrite: Keep the same URL but load the redirected page

### Load Balancer (advanced)

In this section, we'll use Docker to create a load-balanced server. Follow these steps:

1. Create a server folder in the project

2. Create an index.js file in the server folder

3. Go to the server folder:
```bash
cd server
```

4. Initialize a Node.js application:
```bash
npm init
```

5. Accept all defaults when prompted:
```bash
npm init -y
```

6. Install the Express.js library:
```bash
npm install express
```

7. Create a basic Node.js application in "index.js":
```js
const express = require('express')

const app = express()

app.get("/", (req, res) => {
    res.send("I am an endpoint")
})

app.listen(7777, () => {
    console.log("listening on port 7777")
})
```

8. To run the application from the script, add to “package.json” in the section script:
```json
"start": "node index"
```

9. Run the application using the script:
```bash
npm run start
```

10. Create a Dockerfile in the "server" folder to run the server:
```Dockerfile
FROM node:16

# Setting up the work directory
WORKDIR /usr/src/app

COPY package*.json ./

# Installing dependencies
RUN npm install

# Copying all the files in our project
COPY . .

# Exposing server port
EXPOSE 7777

# Starting our application
CMD [ "npm", "run", "start"]
```

11. Open the docker application in mac

12. Build a Docker image for your server:
```bash
docker build . -t myserver
```

13. Run a container from the image and map it to port 1111:
```bash 
docker run -p 1111:7777 -d myserver
```
* p. → for mapping
* d → in detached mode

14. Run different ports to have multiple servers:
```bash 
docker run -p 2222:7777 -d myserver
docker run -p 3333:7777 -d myserver
docker run -p 4444:7777 -d myserver
```

15. Modify the Nginx configuration file to enable load balancing:
```bash
upstream backendserver {
        server 127.0.0.1:1111;
        server 127.0.0.1:2222;
        server 127.0.0.1:3333;
        server 127.0.0.1:4444;
    }
```
before the server block

```
location / {
    proxy_pass http://backendserver/;
}
```
\* Don’t forget to reload NGINX
