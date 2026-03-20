FROM node:20-bookworm-slim

# Install audiowaveform
RUN apt-get update \
 && apt-get install -y curl \
 && curl -L -o /tmp/audiowaveform.deb \
 https://github.com/bbc/audiowaveform/releases/download/1.10.2/audiowaveform_1.10.2-1-12_amd64.deb \
 && apt-get install -y /tmp/audiowaveform.deb \
 && rm /tmp/audiowaveform.deb \
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