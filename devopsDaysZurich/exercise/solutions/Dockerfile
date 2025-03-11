# Multi-stage build  
FROM node:22 as builder  

WORKDIR /app  

# Copy package.json and install dependencies  
COPY package.json .  
RUN npm install --only=production  

# Copy application code  
COPY index.js index.js  

# Final minimal runtime image  
FROM node:22

WORKDIR /app  

# Copy built files from builder stage  
COPY --from=builder /app /app  

# Use a non-root user  
USER nonroot  

# Expose port  
EXPOSE 3000  

# Run the application  
CMD ["index.js"]