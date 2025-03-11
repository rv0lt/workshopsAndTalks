# use ubuntu as base image
FROM alpine:3.20 AS build-env

# install package to compile the source code
RUN apk add --no-cache build-base

# copy the source code
COPY hello.c hello.c

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# FROM alpine for an even greater size reduce
FROM alpine:3.20
    
# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]

