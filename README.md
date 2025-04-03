# v8Py
## An in-memory Python enterpreter inside v8

```js
const Py = require("v8Py");

Py({x:0,y:0,z:0},(payload)=>{
  const pos = payload.pos;
  updateMap(pos);
});

function updateMap(pos){
  ...
}

call js from py (i don't know python so this is sudo code):

//First the JavaScript setup:
const callbacks = {};
callbacks[UUID()] = (payload)=>{
  callbacks[payload.UUID]("Hi!");
};

Python and JavaScript can comunicate with eachother like in this WebSockets example:

import http from 'http';
import { Server, Client } from './socket.io-cb.js';

const httpServer = http.createServer();
httpServer.listen(3000);

const server = new Server(httpServer, (socket) => {
  socket.on('func', (data, cb) => {
    console.log(data); // 123
    cb('XYZ');
  });
});

const client = new Client('http://localhost:3000');
client.emit('func', '123', (data) => {
  console.log(data); // XYZ
  client.disconnect();
  server.close();
});

//socket.io-cb.js 
import { Server as SocketIOServer } from 'socket.io';
import SocketIOClient from 'socket.io-client';
import { randomUUID } from 'crypto';

class Socket {
  constructor(p) {
    this.callbacks = {};
    this.listeners = {};
    this.socket = typeof p === 'string' ? SocketIOClient.connect(p) : p;

    this.socket.on('msg', (data) => {
      for (const key in this.listeners) {
        if (key === data.func) {
          this.listeners[key].forEach((cb) => {
            cb(data.data, (d) => {
              this.socket.emit('gsm', {
                uuid: data.uuid,
                data: d,
              });
            });
          });
        }
      }
    });
    
    this.socket.on('gsm', (data) => {
      this.callbacks[data.uuid](data.data);
      delete this.callbacks[data.data];
    });
  }

  emit(func, data, cb) {
    const uuid = randomUUID();
    this.callbacks[uuid] = cb;
    this.socket.emit('msg', {
      func,
      data,
      uuid,
    });
  }

  on(func, cb) {
    if (!this.listeners[func]) this.listeners[func] = [];
    this.listeners[func].push(cb);
  }

  disconnect() {
    this.socket.disconnect();
  }
}

class Server {
  constructor(server, onConnection) {
    this.io = new SocketIOServer(server);
    this.io.on('connection', (socket) => {
      onConnection(new Socket(socket));
    });
  }

  close() {
    this.io.close();
  }
}

class Client {
  constructor(url) {
    this.socket = new Socket(url);
  }

  emit(func, data, cb) {
    this.socket.emit(func, data, cb);
  }

  disconnect() {
    this.socket.disconnect();
  }
}

export { Server, Client, Socket };
```


# v8.dev [![Build status](https://github.com/v8/v8.dev/actions/workflows/deploy.yml/badge.svg)](https://github.com/v8/v8.dev/actions/workflows/deploy.yml)

This repository hosts the source code of [v8.dev, the official website of the V8 project](https://v8.dev/).

## Local setup

1. Clone the repository and `cd` into it.
1. Install and use the [expected](https://github.com/v8/v8.dev/blob/main/.nvmrc) Node.js version: `nvm use`
1. Install dependencies: `npm install`

`npm run` shows the full list of supported commands. Highlights:

- `npm run build` builds the site into `dist`.
- `npm run watch` builds the site into `dist` and watches for changes.
- `npm start` kicks off a local HTTP server.
