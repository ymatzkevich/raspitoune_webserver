FROM node:20-bookworm-slim

# Install audiowaveform
RUN apt-get update \
 && dpkg -i /tmp/audiowaveform.deb || true \
 && apt-get -f install -y \
 && rm -rf /var/lib/apt/lists/*

# Verify
RUN audiowaveform --help

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