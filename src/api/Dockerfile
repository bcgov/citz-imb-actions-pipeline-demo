FROM node:18.5.0

# Create an app directory
WORKDIR /usr/src/app

# Install Project Specific Dependencies
COPY src/api/package*.json ./
RUN npm i --only=production

# Bundle project artifact
COPY src/api/. .

EXPOSE 3000
CMD ["node",  "index.js"]
