# Use a specific version of the Node.js base image
FROM node:16

# Create and set the working directory
RUN mkdir /backend
WORKDIR /backend

# Copy package.json and package-lock.json separately to leverage Docker's caching mechanism
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code
COPY . .

# Run Prisma commands (assuming you need to push schema changes and generate Prisma client)
RUN npx prisma db push && npx prisma generate

# Build the application
RUN npm run build

# Expose the port the app runs on
EXPOSE 8080

# Command to run the application
CMD ["node", "build/index.js"]
