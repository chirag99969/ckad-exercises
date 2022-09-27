FROM node:12
WORKDIR /usr/shared/app
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
