FROM node:alpine3.21

WORKDIR /usr/app

COPY package*.json .
COPY *.js .

RUN npm install

EXPOSE 3002

ENTRYPOINT [ "node" ]
CMD ["index.js"]