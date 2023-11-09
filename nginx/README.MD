# Nginx Web Server
This is a tutorial to create and run an Nginx web server on mac

### Nginx Installation
Run the following commands in your terminal:

1. Install Nginx:
```
brew install nginx
```

2. Go to the directory of the installation (this is the default directory when you install Nginx):
```
cd /usr/local/etc/nginx
```

3. Open VSCode of this directory* (you can also open it from VSCode):
```
code .
```

4. Start nginx:
```
nginx
```

5. Open the browser to check that the server is running correctly:
```
http://localhost:8080/
```

\* If you an issue running this command, try to create a symbolic link to VSCode:
```
sudo ln -s /Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code /usr/local/bin/
```

### Nginx Terminology

1. We have the key value; pairs, which are also called directives, for example: 
```
worker_processes  1;
```

2. There are blocks that contain a lot of directives, which are called context:
```
events {
    worker_connections  1024;
}
```

### Serving Static Content
\* Don't forget to change the root path to your path
1. The nginx.conf file has to be modified to serve HTML page, here is the code:
```
http {
    server {
        listen 8080;
        root /path-to-your-folder/mysite;
    }
}
event {}
```
Nginx is listening on port 8080 and root server all the files that are in this folder “mysite”. By default, it searches for to index.html file.
*You will have to create a folder on your desktop for example "mysite" that will contain your HTML code.

2. Nginx has to be reloaded after each update, run this command in the terminal to reload Nginx:
```
nginx -s reload
```

3. To accept all the file types in the project we need to include the “mime.types” file in the config file:
```
include mime.types
```

### Location Contexte

1. We can serve different HTML files. To serve another location we add the location in the server block:

```
location /fruits {
    root /another/loction;
}
```

2. To access the same content page but with a different ending url we need to use the “alias” keyword instead of root in our new location block:
```
location /carbs {
    alias /your/path/mysite/fruits;
}
```

3. To use a custom HTML filename (not index), we have to use “try_files” to try to find the HTML file that correspond:
```
location /vegetables {
    alias /your/path/mysite;
    try_files /vegetables/veggies.html /index.html =404;
}
```
##### How it work?

try_files need 3 arguments:
```
try_files arg1 arg2 arg3;
```

* arg1 → Search for the HTML file
* arg2 → If the file doesn’t exist then open the index.html file (the standard one)
* arg3 → If it also failed, throw a 404 error

4. To use regular expression:
```
location ~* /count/[0-9] {
    root /your/path/mysite;
    try_files /index.html =404;
}
```

### Redirect and Rewrites

1. To redirect the user, we use the code 307 followed by the page that we want to display:
```
location /crops {
    return 307 /fruits;
}
```

2. To rewrite a page:
```
rewrite ^/number/(\w+) /count/$1;
```
\w+ → Mean the variable
$1 → It’s the variable

##### The difference between the redirect and the rewrite is:

- Redirect: change the URL to the page displayed
- Rewrite: Keep the same URL but load the redirected page

### Load Balancer

We are gonna use Docker to create a server:
1. Create a server folder in the project

2. Create an index.js file in the server folder

3. Go to the server folder:
```
cd server
```

4. Run the command to initialize a node application:
```
npm init
```

5. Run the command:
```
npm init -y
```

6. Run the command to have the server:
```
npm install express
```

7. Create a basic node application:
```
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
```
"start": "node index"
```

9. To execute the start, run:
```
npm run start
```

10. Create a docker file in the server folder to run the server:
```
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

12. Run the command to start a docker container:
```
docker build . -t myserver
```

13. Run the container of the image that we created and mapped to port 1111:
```
docker run -p 1111:7777 -d myserver
```
p. → for mapping
d → in detached mode
We can see in the  application the container running


14. Running different ports to have multiple servers:
```
docker run -p 2222:7777 -d myserver
docker run -p 3333:7777 -d myserver
docker run -p 4444:7777 -d myserver
```

15. Add these lines in the config file to round robben  the requests to different servers each time:
```
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
