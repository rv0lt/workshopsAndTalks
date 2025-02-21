# use ubuntu as base image
FROM ubuntu 

# install build-essential package to compile the source code
RUN apt update
RUN apt install -y build-essential

# copy the source code
COPY hello.c hello.c

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# Run the program
ENTRYPOINT ["/helloWorld"]

