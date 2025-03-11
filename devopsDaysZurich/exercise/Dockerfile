# Use an official Node.js runtime as the base image  
FROM node:22  

# Set the working directory in the container  
WORKDIR /app  

# Copy package.json and install dependencies  
COPY package.json .  
RUN npm install  

# Copy the rest of the application  
COPY index.js index.js 

# Expose port 3000  
EXPOSE 3000  

# Run the application  
CMD ["node", "index.js"]