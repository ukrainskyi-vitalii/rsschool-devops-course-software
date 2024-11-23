# Use an official Node.js runtime as a parent image
FROM node:18-alpine AS builder

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json for dependency installation
COPY package*.json ./

# Install production dependencies
RUN npm install

# Copy the rest of the application
COPY . .

# Build the application (for Nest.js)
RUN npm run build


# Use a smaller image for the final build
FROM node:18-alpine AS runner

# Set working directory
WORKDIR /app

# Copy only the production build and node_modules from the builder stage
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

# Expose the application port
EXPOSE 3000

# Command to start the application
CMD ["node", "dist/main"]
