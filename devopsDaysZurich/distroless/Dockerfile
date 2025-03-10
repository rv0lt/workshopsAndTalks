# use ubuntu as base image
FROM ubuntu as build-env

# install build-essential package to compile the source code
RUN apt update && apt install -y build-essential

# copy the source code
COPY hello.c hello.c

# Compile and generate binary
RUN gcc -o helloWorld hello.c

# use distroless container to run the program
FROM scratch

# copy the required libraries to run the program
COPY --from=build-env  /lib/aarch64-linux-gnu/libc.so.6 /lib/aarch64-linux-gnu/libc.so.6
COPY --from=build-env  /lib/ld-linux-aarch64.so.1 /lib/ld-linux-aarch64.so.1

# copy the /etc/passwd 
COPY --from=build-env  /etc/passwd /etc/passwd

# tmp directory
COPY --from=build-env  /tmp /tmp

# copy binary executable to new layer
COPY --from=build-env ./helloWorld ./helloWorld 

# Run the program
ENTRYPOINT ["/helloWorld"]

