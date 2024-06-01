# Sprawozdanie_Zadanie1 Część Obowiązkowa

# 1. Program serwera w Node.js

```
const express = require('express');
const fs = require('fs');
const moment = require('moment-timezone');
const requestIp = require('request-ip');

const app = express();
const PORT = process.env.PORT || 8080;
const AUTHOR = 'Maksim Rymasheuski'; // Imię i nazwisko studenta

// Logowanie informacji o uruchomieniu serwera
const logServerStart = () => {
    const logMessage = `Server started on ${new Date().toISOString()} by ${AUTHOR} on port ${PORT}`;
    console.log(logMessage);
    fs.appendFileSync('/var/log/server.log', logMessage + '\n');
};

// Middleware do uzyskiwania IP klienta
app.use(requestIp.mw());

// Middleware do logowania informacji o klientach
app.use((req, res, next) => {
    const clientIp = req.clientIp;
    const clientTime = moment().tz(clientIp).format('YYYY-MM-DD HH:mm:ss');
    console.log(`Client IP: ${clientIp}, Client Time: ${clientTime}`);
    next();
});

// Obsługa żądania głównego
app.get('/', (req, res) => {
    const clientIp = req.clientIp;
    const clientTime = moment().tz(clientIp).format('YYYY-MM-DD HH:mm:ss');
    res.send(`<h1>Adres IP klienta: ${clientIp}</h1><p>Data i godzina w jego strefie czasowej: ${clientTime}</p>`);
});

// Uruchomienie serwera
app.listen(PORT, () => {
    logServerStart();
});
```

# 2. Plik Dockerfile

```
# Etap 1: Budowanie aplikacji
FROM node:14-alpine AS build

LABEL maintainer="Maksim Rymasheuski"

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Etap 2: Tworzenie ostatecznego obrazu
FROM node:14-alpine
LABEL maintainer="Maksim Rymasheuski"

WORKDIR /app
COPY --from=build /app /app

# Informacje o uruchomieniu serwera
RUN echo "Server built by Maksim Rymasheuski" > /app/author.txt

# Healthcheck
HEALTHCHECK CMD curl --fail http://localhost:8080 || exit 1

# Uruchomienie serwera
CMD ["node", "index.js"]
```

# 3. Polecenia niezbędne do:

  a. Zbudowanie obrazu kontenera
  ```
  docker build -t my-node-server .
  ```
  ```
    [+] Building 10.7s (13/13) FINISHED                                                                      docker:default
   => [internal] load build definition from Dockerfile                                                               0.2s
   => => transferring dockerfile: 585B                                                                               0.1s
   => [internal] load metadata for docker.io/library/node:14-alpine                                                  1.7s
   => [auth] library/node:pull token for registry-1.docker.io                                                        0.0s
   => [internal] load .dockerignore                                                                                  0.2s
   => => transferring context: 2B                                                                                    0.0s
   => [internal] load build context                                                                                  0.1s
   => => transferring context: 645B                                                                                  0.0s
   => [build 1/5] FROM docker.io/library/node:14-alpine@sha256:434215b487a329c9e867202ff89e704d3a75e554822e07f3e0c0  0.0s
   => CACHED [build 2/5] WORKDIR /app                                                                                0.0s
   => CACHED [build 3/5] COPY package*.json ./                                                                       0.0s
   => CACHED [build 4/5] RUN npm install                                                                             0.0s
   => [build 5/5] COPY . .                                                                                           0.5s
   => [stage-1 3/4] COPY --from=build /app /app                                                                      0.9s
   => [stage-1 4/4] RUN echo "Server built by Maksim Rymasheuski" > /app/author.txt                                  2.2s
   => exporting to image                                                                                             1.1s
   => => exporting layers                                                                                            0.9s
   => => writing image sha256:e93657c37d52bf7f2fe1b6e80fa2ac7e3c1ab59cec4907a70c084f6fa5b24b44                       0.0s
   => => naming to docker.io/library/my-node-server                                                                  0.2s
  
  View build details: docker-desktop://dashboard/build/default/default/u1jg7uhl8edoicjwlhs4oh76c
  
  What's Next?
    View a summary of image vulnerabilities and recommendations → docker scout quickview
  ```

  b. Uruchomienie kontenera na podstawie zbudowanego obrazu
  ```
  docker run -d -p 8080:8080 --name my-node-server my-node-server
  ```

  ```
  1dadfcff93234defca96f341e80839b2cf9402172318334de7b5e431138d8dc2
  ```

  c. Sposób uzyskania informacji, które wygenerował serwer w trakcie uruchamiana kontenera
  ```
  docker exec my-node-server cat /app/author.txter
  ```
  
  ```
  Server built by Maksim Rymasheusk
  ```

  ```
  docker exec my-node-server cat /var/log/server.log
  ```

  ```
  Server started on 2024-06-01T14:33:37.967Z by Maksim Rymasheuski on port 8080
  ```

  d. Sprawdzenie, ile warstw posiada zbudowany obraz
  ```
  docker history my-node-server
  ```

  ```
    IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
  410f3c59f80f   12 minutes ago   CMD ["node" "index.js"]                         0B        buildkit.dockerfile.v0
  <missing>      12 minutes ago   HEALTHCHECK &{["CMD-SHELL" "curl --fail http…   0B        buildkit.dockerfile.v0
  <missing>      12 minutes ago   RUN /bin/sh -c echo "Server built by Maksim …   35B       buildkit.dockerfile.v0
  <missing>      12 minutes ago   COPY /app /app # buildkit                       9.67MB    buildkit.dockerfile.v0
  <missing>      6 days ago       WORKDIR /app                                    0B        buildkit.dockerfile.v0
  <missing>      6 days ago       LABEL maintainer=Maksim Rymasheuski             0B        buildkit.dockerfile.v0
  <missing>      14 months ago    /bin/sh -c #(nop)  CMD ["node"]                 0B
  <missing>      14 months ago    /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
  <missing>      14 months ago    /bin/sh -c #(nop) COPY file:4d192565a7220e13…   388B
  <missing>      14 months ago    /bin/sh -c apk add --no-cache --virtual .bui…   7.85MB
  <missing>      14 months ago    /bin/sh -c #(nop)  ENV YARN_VERSION=1.22.19     0B
  <missing>      14 months ago    /bin/sh -c addgroup -g 1000 node     && addu…   104MB
  <missing>      14 months ago    /bin/sh -c #(nop)  ENV NODE_VERSION=14.21.3     0B
  <missing>      14 months ago    /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
  <missing>      14 months ago    /bin/sh -c #(nop) ADD file:9a4f77dfaba7fd2aa…   7.05MB
  ```
  
