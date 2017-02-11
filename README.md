# Docker 101

## Commands

- `docker images` list images
- `docker rmi IMAGE ID` remove image
- `docker pull mhart/alpine-node` download an images
- `docker run --rm nginx` run an image
- `docker ps` list running images
- `docker stop CONTAINER ID` - stop a running image

## nodejs

Create a simple node server

```js
// nodejs/index.js
var http = require('http')

var server = http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'})
  res.end('Welcome to Node.js!\n')
})

server.listen(3000)
console.log('Server running at http://127.0.0.1:3000/')
```

Add a docker file for the node app

```Dockerfile
# nodejs/Dockerfile
FROM mhart/alpine-node
COPY index.js .
EXPOSE 3000
CMD node index.js
```

Build and run

```bash
docker build -t foo/node .
docker run -d -p 3000:3000 --name node-app foo/node
```

Then you should be able to hit http://locahost:3000/

## nginx

Add nginx config

```Nginx
# nginx/default.conf
server {
  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_pass http://app:3000;
  }
}
```

Add a docker file for nginx

```Dockerfile
# nginx/Dockerfile
FROM nginx
COPY default.conf /etc/nginx/conf.d/
```

Build and run

```bash
docker build -t foo/nginx .
docker run -p 8000:80 --link node-app:app --name nginx-proxy foo/nginx
```

Then you should be able to hit nginx at http://locahost:8000/ which proxies through to Node.
