This is a README for project Docker

There are 7 tasks in this project as follows:

## Task 0. Create Your First Docker Image
To create a Docker image, you will need to utilize a Dockerfile. Create a Dockerfile that:

-   Is based on the latest ubuntu
-   Update APT using apt-get update
-   Upgrade currently installed software through APT using apt-get upgrade -y
-   Once built, you can run the Docker image in a container and it will echo “Hello, World!” on the terminal.

## Task 1. Back-end
For this task, start by making a copy of your  `task0`  directory and name it  `task1`. Next, we want to change the  `Dockerfile`  to install Python3, pip3, and Flask. You may not have used Flask, yet, but not to worry; for this project, we will give you all of the Flask code you need to get started. We’ll validate that all have been installed correctly by running a Flask server with one endpoint that when called returns “Hello, World!”

-   Install python3, python3-pip, and flask
    -   Note: Make sure to automatically install and skip user input with the “-y” flag for apt-get
    -   Note: flask must be installed with pip3, not through apt-get
-   Locally, create a Python file named  `api.py`  and paste the following Python script - it uses Flask to create one endpoint that returns “Hello, World!” when called
    -   Hosting this Flask app on  `0.0.0.0`  instead of  `127.0.0.1`  means that it is reachable outside of the current machine (the current machine being a Docker container which is running inside of your laptop/desktop)
    -   Host this Flask app on port 5252

## Task 2. Front-end
For this task, start by making a copy of your  `task1`  directory and name it  `task2`. We have created a very simple API server with one route that returns “Hello, World!” and now we want to create a web page to view content from our API server in the context of a more full front-end. Before creating our front end, let’s reorganize this project a bit in our  `task2`  directory. Create a new directory named  `back-end`  inside of your  `task2`  directory and move all of the files currently in  `task2`  inside of the new  `back-end`  directory. That means you should have  `api.py`  and  `Dockerfile`  inside of your new  `task2/back-end`  directory.

Next, let’s create a new  `task2/front-end`  directory, as well. Inside your new  `task2/front-end`, please clone this repository: https://github.com/Holberton-Tulsa/softy-pinko-front-end

With the  `soft-pinko-front-end`  directory and files in place, we need to create a new  `Dockerfile`  in your  `task2/front-end`  directory. In order to host and distribute our front-end content we will use a static web server named  [Nginx](https://intranet.hbtn.io/rltoken/MxDjmBN1aYx8Km72LtAkuQ "Nginx"); there are many others that can be used, but this one is rather simple to get started with and conveniently has a Docker image that we can use which is pre-installed. In the new  `task2/front-end/Dockerfile`, instead of using the latest ubuntu version, use the latest version of  `nginx`. Next in your  `Dockerfile`, you’ll need to copy the  `softy-pinko-front-end`  to the Docker image. Then, you need to configure Nginx to serve your static content. Your  `softy-pinko-front-end files`  must be copied to  `/var/www/html/softy-pinko-front-end`  on the Docker image. Finally, you’ll need to create an Nginx config file named  `softy-pinko-front-end.conf`  inside of the  `task2/front-end`  directory. This file must be copied to the Docker image to  `/etc/nginx/conf.d/default.conf`  and must include all of the configuration settings required to get your site to show up when visiting the URL. When researching Nginx config files, the only section you’ll need in the  `softy-pinko-front-end.conf`  file is the “server” section. Pay attention to the syntax used to set up a port to listen to (recommendation: port 9000), the name of the server, the location, and the index file to use.

**softy-pinko-front-end.conf**

```
server {
// Replace with your Nginx server configuration
}
```

## Task 3. Connecting the Front-end and Back-end
To start this task, make a copy of your  `task2`  directory and name it  `task3`.

This task will have you connect your front-end to the back-end allowing you to have dynamic data on your front-end. This means that communication will occur between your two Docker images (each of which will be running in their own Docker container). To facilitate this, be sure to have multiple terminal instances open so you can have one Docker container running on each.

First thing we should do is prepare the front-end. We need to add a couple of things:

**New HTML to add**

```
<h1 id="dynamic-content"></h1>
```

This needs to be added before the  `<h1>`  that includes the text “We provide the best strategy to grow up your business”.

**index.html**

```
// . . . SOME HTML BEFORE . . . 
<div class="offset-xl-3 col-xl-6 offset-lg-2 col-lg-8 col-md-12 col-sm-12">
    <h1 id="dynamic-content"></h1>
    <h1>We provide the best <strong>strategy</strong><br>to grow up your<strong>business</strong></h1>
    <a href="#features" class="main-button-slider">Discover More</a>
</div>
// . . . SOME HTML AFTER . . . 
```

This new  `<h1>`  tag with the ID of  `dynamic-content`  is the place we’re going to insert dynamic data from our API server.

Now we need to actually use some JavaScript code to make the request. Place the following script near the bottom of the HTML as the last  `<script>`  tag before the closing  `</body>`  tag.

**Add this script to your index.html**

```
<script>
    // Load dynamic data from the back-end on port 5252
    $(function() {
        $.ajax({
            type: "GET",
            url: "http://localhost:5252/api/hello",
            success: function(data) {
                console.log(data);
                $('#dynamic-content').text(data);
            }
        });
    });
</script>
```

That is all the front-end needs to be able to connect to the back-end, however there is one more thing we must do to our back-end so that it can accept cross-origin requests from our front-end. We must use the CORS plugin for Flask.

In your back-end’s  `back-end/Dockerfile`, use  `pip3`  to install  `flask-cors`  after you have installed  `flask`  through  `pip3`. Next, use this updated python script as your  `api.py`.

**api.py**

```
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

@app.route('/api/hello')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5252)
```

Now, your back-end will allow your front-end to communicate with it even though they are not on the same server (they are on two different Docker containers).

## Task 4. Making it Simpler with Docker Compose
Having separate Docker images/containers per component of your application can reduce the complexity of your web apps. That said, having more than a single Docker image that you need to spin up in containers can present challenges; what if you need to spin up 3 distinct Docker images, or 7, or 50? Opening each one, one at a time would quickly become an issue. That’s where Docker Compose comes into play. With a  `docker-compose.yml`  file, we can specify the different components of your entire application, set up some basic configurations, and allow Docker to handle spinning up the entire application all at once, no matter how many containers there are.

Before going further, be sure to copy the  `task3`  directory and name it  `task4`.

Inside the  `task4`  directory, create a  `docker-compose.yml`  file

## Task 5. Proxy Server
So far we’ve created a static server and one API server, each of which has its own Docker images and runs in different Docker containers. This is okay, but this means that we have to keep track of two separate locations. If we change the port that the API server is using, then we’d have to change the client to hit that new port; in this case ports on our computer, but it would be IP addresses if these were external servers. This is not ideal, especially if you have many different clients that you’d need to update and deploy.

Instead, let’s put a server in front of our static server and API server that acts as a proxy between the client and our full application. Clients (in this case the web browser) will only have to know about the proxy server’s location and our front-end can also simply hit the same proxy server, rather than going to the API directly, to then be routed to the API server in order to get dynamic data from it.

Before going further, copy the  `task4`  directory and name it  `task5`. In the  `task5`  directory, create a new folder named  `proxy`.

Just like with our static-content server, we’re going to use Nginx for this proxy server. In your  `proxy`  directory, create a  `Dockerfile`  that uses the latest version of nginx. Also, create a file named  `proxy.conf`, which will contain all of the configuration for your proxy server. In the  `Dockerfile`, make sure to copy the  `proxy.conf`  file to this specific location in your Docker image for the proxy server:  `/etc/nginx/conf.d/default.conf`.

For the  `proxy.conf`  file, the only section you’ll need in the  `proxy.conf`  file is the “server” section. You want this proxy server to be listening to port 80. Set up a  `location`  section for  `/`  and use  `proxy_pass`  to route any request coming in on that route to  `http://INSERT-YOUR-FRONT-END-DOCKER-COMPOSE-SERVICE-NAME-HERE:9000`. Next, set up a  `location`  section for  `/api`  and use  `proxy_pass`  to route any request coming in on that route to  `http://INSERT-YOUR-BACK-END-DOCKER-COMPOSE-SERVICE-NAME-HERE:5252`.

**softy-pinko-front-end.conf**

```
server {
    // Replace with your Nginx server configuration
}
```

Note that you are using the same service name that you used in your  `docker-compose.yml`  file for the front and back ends. This is because Docker has an internal DNS that will setup routes to internal IP addresses for those names. This is a bit of “magic” that Docker does, but it makes it super convenient to not have to know the exact IP addresses of these services.

Another thing that needs to be modified is the JavaScript that is used. It was previously calling the back-end server directly; instead, we want to have the JavaScript make an API call through the proxy server as well. Replace your JavaScript at the bottom of the  `index.html`  file with the following:

**JavaScript At Bottom Of index.html**

```
<script>
    // Load dynamic data from the back-end on port 5252
    $(function() {
        $.ajax({
            type: "GET",
            url: "/api/hello",
            success: function (data) {
                console.log(data);
                $('#dynamic-content').text(data);
            }
        });
    });
</script>
```
Next, we need to update the  `docker-compose.yml`  file to add a new service named  `proxy`. Be sure to include the same sections as previously used: * build, context, dockerfile * image * ports * Map the container’s port 80 to the host machine’s port 80. * Note: You should remove the mapping for the front-end and back-end service to the host machine ports. You will want to have port 5252 and port 9000 open internally on the containers so that other Docker containers can reach them, but you do not want external clients reaching out directly to the front-end or the back-end; everything should go through the proxy server. If you do not map any ports, you will not be able to get past the firewall that Docker has put up. Docker uses something similar to Linux’s Uncomplicated Firewall ufw; you do not directly work with this firewall, but instead, map allowed ports in Dockerfiles or docker-compose files to allow/disallow access through certain ports. depends_on


## Task 6. Scale Horizontally
Congratulations! You’ve got a proxy server that routes to either a static-content server or to an API server for dynamic data. This works well, for now…but what would happen if traffic to your site spiked up dramatically to where your API server becomes too flooded with requests to be able to respond quickly? In this case, we would like to add a second API server and load-balance all requests between those two servers. Luckily, with Nginx and Docker, this is a simple flag when running  `docker-compose up`!

For this task, make a copy of the  `task5`  directory and name it  `task6`. Your goal is to launch your Docker containers such that you have 2 (or more) API servers. Nginx will act as a load-balance between them so that every request goes to the next API server using the Round-Robin load-balancing algorithm.

Create a new file in the  `task6`  directory named  `2-api-servers.txt`  and put in the exact docker-compose command used to spin up two API servers, instead of just one. Note: for this task, the checker will assume that you named your back-end server  `back-end`  and your file must end with a newline