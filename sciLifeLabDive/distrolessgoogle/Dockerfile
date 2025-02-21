# use ubuntu as base image
FROM ubuntu as build-env

# install build-essential package to compile the source code
RUN apt update && apt install -y build-essential
#RUN apk update && apk --update --no-cache add build-base

# copy the source code
COPY hello.c hello.c

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# use distroless container to run the program
FROM gcr.io/distroless/cc
    
# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]

