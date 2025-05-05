
#Build Stage
# Use a stable Node.js base image
FROM node:20.18.0-alpine3.16

# Set the working directory inside the container
WORKDIR /app

# Copy only the package files first (for caching layer benefit)
COPY package*.json ./

# Install dependencies (legacy-peer-deps to avoid peer conflicts)
#RUN npm install --legacy-peer-deps
RUN npm ci

# Copy the rest of your app
COPY . .

# # Expose the port your app listens on
# EXPOSE 3000

# # Run the app
# CMD ["npm", "start"]

RUN npm run build 
#This will run the app in development mode->Dist->/app/dist

# Production Stage
FROM nginx:1.25.2-alpine
# Copy the build output to the Nginx HTML directory
COPY --from=build /app/dist /usr/share/nginx/html
# Expose the port Nginx is running on
EXPOSE 80
# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]
#docker build -t myapp:latest .
#docker run -d -p 8080:80 myapp:latest