# use ubuntu as base image
FROM ubuntu

# copy the source code
COPY hello.c hello.c

# install build-essential package to compile the source code
RUN apt update
RUN apt install -y build-essential

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# Run the program
ENTRYPOINT ["/helloWorld"]

