FROM node:20-alpine

# Install audiowaveform (BBC tool)
RUN apk add --no-cache audiowaveform

# Create app directory
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy rest of the project
COPY . .

# Expose port
EXPOSE 3000

# Start server
CMD ["node", "server.js"]