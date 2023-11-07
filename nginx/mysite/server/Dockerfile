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