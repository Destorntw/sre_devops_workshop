# SRE + Devops Workshop

## In this lab we will create a NodeJS application and deploy it to the Openshift cluster, and understand what does it mean to change the replicaCount

### Part 1 lets build our application

#### 1. create a new folder named Application (this will be our root application folder) in this folder create another folder named "src"

```Bash
mkdir Application
cd Application
mkdir src
```

#### 2. create a new file called app.js

```Bash
cd src
touch app.js
npm init
```

Fill the fields:

- package-name = hello-world
- version = 1.0.0
- description = application for workshop
- entrypoint = app.js
- all the rest leave empty

#### 3. open the file in VScode(codespaces) and create a basic web application

```js
var image = process.env.IMAGE
var tag = process.env.TAG
var host = process.env.HOSTNAME
var port = process.env.PORT || 8080;
var express = require('express');
const Prometheus = require('prom-client');
const register = new Prometheus.Registry();

app = express();

register.setDefaultLabels({
  app: 'hello-world Nodejs application'
})
Prometheus.collectDefaultMetrics({register})


const http_request_counter = new Prometheus.Counter({
  name: 'myapp_http_request_count',
  help: 'Count of HTTP requests made to my app',
  labelNames: ['method', 'route', 'statusCode'],
});
register.registerMetric(http_request_counter);


   
   
// Health Probe - Application Liveliness
app.get('/health/liveliness',function(req,res){
  console.log(`I am Alive`)
  res.status(200)
  res.send('Healty')
});
    
// Health Probe - Application Readiness
app.get('/health/readiness',function(req,res){
  console.log(`I am Ready`)
  res.status(200);
  res.send('Ready');
  });  

app.get('/', function (req, res) {

  var clientHostname = req.headers['x-forwarded-for'] || req.connection.remoteAddress;

  res.send(`Hello Gitea-New-Demo!!,New Version, My Image is ${image}:${tag} , the Server is ${host} accessed from ${clientHostname} `);

  console.log(`Someone accessed me! --> from ${clientHostname}`)
});

app.get('/test1', function (req, res) {

  res.send(`This is Test1, All Good`);

  console.log(`Someone accessed Test1 Path!`)
});

app.get('/test2', function (req, res) {

  res.send(`This is Test2, All Good`);

  console.log(`Someone accessed Test2 Path!`)
});

app.get('/metrics', function(req, res)
{
    res.setHeader('Content-Type',register.contentType)

    register.metrics().then(data => res.status(200).send(data))
});

app.use(function(req, res, next)
{
    // Increment the HTTP request counter
    http_request_counter.labels({method: req.method, route: req.originalUrl, statusCode: res.statusCode}).inc();

    next();
});

app.listen(port, function () {
  console.log(`Example app listening on port ${port}!`);
});
```

#### 4. test the application localy to see if it works

```Bash
npm install express prom-client
node app.js
```

the codespaces enviourment will notify you with a link to the Application access or you can:

```Bash
curl http://localhost:8080
```

add a .gitignore to the root folder to not sync the Node Modules folder.

```Bash
cd ..
touch .gitignore
echo "src/node_modules" >> .gitignore
```

Commit and push the new file to the Repo

```Bash
git add .
git commit -m "hello-world app"
git push
```

#### 5. Now lets build a Contianer for our app and push it to our quay.io image registry

i. create a Dockerfile in our Home folder

```Bash
touch Dockerfile
```

ii. Open the Dockerfile with VScode and copy the following snippet to the Dockerfile

```Dockerfile
FROM registry.access.redhat.com/ubi9/nodejs-18

# Create app directory
WORKDIR /tmp

USER root
# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY src/package*.json ./


# update the base image
RUN npm install && npm audit fix --force
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY src .

USER 1001

EXPOSE 8080
CMD [ "node", "app.js" ]
```

iii. Build the continer image

```Bash
docker build . -t quay.io/<quay-userName>/<imageName>:v1
```

Then wait for it to finish

Now, navigate to www.quay.io, and login with your qauy username and password

- Click on "+ Create New Repository".
- enter the image name you enter in the docker build step.
- Select public
- Select (Empty repository)
- Click "Create Public Repositoy"

iiii. Push the image to quay.io registry

```Bash
$ docker login -u <userName> -p <Password> quay.io

'Login Successful'

$ docker push quay.io/<userName>/<imageName>:v1
...

pushed successfuly!
```

Add,commit and push our changes to our Git Repo (don't forget a comit message)

```Bash
git add .
git commit -m "added Dockerfile"
git push
```

## Great Jog You have Finished Part 1

### Now you can start part 2 [Here](https://github.com/rhilconsultants/HELM-ArgoCD-Lab/blob/main/Lab1/Lab1_Part_2.md)
