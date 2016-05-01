+++
date = "2016-03-03T00:00:00+00:00"
description = "Complete libuv standard input/output and TCP input/output example in C, quite similar to netcat localhost 12345."
title = "libuv standard input/output and TCP input/output example"

+++

**TL;DR**: Complete `libuv` asynchronous standard input/output and TCP input/output example in C, quite similar to `netcat localhost 12345`.

<!--more-->

No better way to start a blog than with a code example!
This example uses `libuv` to open a TCP connection. Everything read from standard input is written to the TCP socket, and everything read from the TCP socket is written to the standard output. This code behaves similarly to how `netcat localhost 12345` would.

## Prerequisites

It is assumed that the system has a C compiler, C standard libraries and `libuv` installed. I also used `netcat`, `valgrind` and `diff` for testing purposes.

[Get `libuv`](http://libuv.org/).

[Get `netcat`](http://netcat.sourceforge.net/).

[Get `valgrind`](http://valgrind.org/).

[Get `diff`](https://www.gnu.org/software/diffutils/).

## Code

The code is divided into two parts:

 - the main function that sets everything up and initializes it,
 - the callbacks that handle input/output events.
 
 I also defined two macro functions. One to check the return value of `libuv` calls. If an error occurred it would print it out. The second one to print an error message in case of memory allocation problems.
 
    #define check_uv(status) \
	    do { \
            int code = (status); \
            if(code < 0){ \
                fprintf(stderr, "%s: %s\n", uv_err_name(code), uv_strerror(code)); \
                exit(code); \
            } \
        } while(0)

    #define memory_error(fmt, ...) \
	    do { \
            fprintf(stderr, "%s: %s (%d): not enough memory: " fmt "\n", __FILE__, __FUNCTION__, __LINE__, ## __VA_ARGS__); \
        } while (0)

Bodies of the macros are wrapped in `do{}while(0)`, which is one of the tricks used to enable calling the macro functions like regular C functions -- with a semicolon at the end. Though, it does make nesting impossible, but it's a non-issue in this example.
 
### Main function

In the main function, the following steps are taken:

 - main loop is initialized,
 - `SIGTERM` and `SIGINT` handling is initialized (to do some cleanup after receiving those),
 - read and write buffers for the TCP input/output are allocated,
 - IP address and port resolving are started and the TCP handle is initialized,
 - standard input and output pipes are initialized and open,
 - the main event loop is started.
 
After the main event loop is stopped (with a `SIGTERM`, `SIGINT` or a closed TCP connection from the other end), the following actions take place:

 - stop handling the signals (`SIGTERM`, `SIGINT`),
 - stop reading from the TCP socket and the standard input, free buffers,
 - a walk is initialized to close all the handles that we might have missed,
 - the main event loop is started again to actually perform the walk and close all handles,
 - when all handles are closed, the main event loop is finally stopped.
 
For the sake of simplicity and static allocation, I have declared most used variables (handles) global.
Additional comments can be found in the code below:

    uv_loop_t loop;                    // Main event loop
    uv_signal_t sigterm;               // SIGTERM handle
    uv_signal_t sigint;                // SIGINT handle
    uv_getaddrinfo_t getaddrinfo_req;  // getaddrinfo request
    struct sockaddr addr;              // sockaddr with data from getaddrinfo
    uv_connect_t connect_req;          // TCP connection request
    uv_tcp_t tcp;                      // TCP handle
    uv_buf_t read_buffer;              // TCP read buffer
    uv_buf_t write_buffer;             // TCP write buffer
    uv_pipe_t stdin_pipe;              // Standard input handle
    uv_pipe_t stdout_pipe;             // Standard output handle

    int main(void){
	    // Main loop initialization
        check_uv(uv_loop_init(&loop));

        // SIGTERM and SIGINT handling
		// on_signal callback will handle them both
        check_uv(uv_signal_init(&loop, &sigterm));
        check_uv(uv_signal_start(&sigterm, on_signal, SIGTERM));
        check_uv(uv_signal_init(&loop, &sigint));
        check_uv(uv_signal_start(&sigint, on_signal, SIGINT));

		// Buffer allocation for TCP reading and writing
		char *buffer;
		if(!(buffer = malloc(BUFFER_LEN))){
		    memory_error("Unable to allocate buffer of size %d", BUFFER_LEN);
		}
        read_buffer = uv_buf_init(buffer, BUFFER_LEN);
		if(!(buffer = malloc(BUFFER_LEN))){
		    memory_error("Unable to allocate buffer of size %d", BUFFER_LEN);
		}
        write_buffer = uv_buf_init(buffer, BUFFER_LEN);
		
		// Start IP and port resolving and initialize the TCP handle
		// on_getaddrinfo callback will later open the TCP connection and start listening for data
        check_uv(uv_getaddrinfo(&loop, &getaddrinfo_req, on_getaddrinfo, IP, PORT, NULL));
        check_uv(uv_tcp_init(&loop, &tcp));

        // Initialize and open stdin and stdout pipes
        check_uv(uv_pipe_init(&loop, &stdin_pipe, 0));
        check_uv(uv_pipe_open(&stdin_pipe, 0));         // 0 is stdin
        check_uv(uv_pipe_init(&loop, &stdout_pipe, 0));
        check_uv(uv_pipe_open(&stdout_pipe, 1));        // 1 is stdout

        // Run the main loop
        // Execution is within this function until ...
        // ... SIGTERM, SIGINT or EOF are encountered
        check_uv(uv_run(&loop, UV_RUN_DEFAULT));

        // Stop handling the signals
        check_uv(uv_signal_stop(&sigterm));
        check_uv(uv_signal_stop(&sigint));
		
		// Stop reading from inputs
        check_uv(uv_read_stop(connect_req.handle));
        check_uv(uv_read_stop((uv_stream_t*)&stdin_pipe));
		
		// Free buffers
        if(read_buffer.base)
            free(read_buffer.base);
        if(write_buffer.base)
            free(write_buffer.base);

        // Initialize and perform the walk
		// on_walk callback closes every open handle
        uv_walk(&loop, on_walk, NULL);
        check_uv(uv_run(&loop, UV_RUN_DEFAULT));
		
		// Close the main loop
        check_uv(uv_loop_close(&loop));

        return 0;
    }
 
### Callbacks

Overall, eleven callbacks are used, but seven of them are really simple and short.
Following events have callbacks written for them:

 - `SIGTERM` and `SIGINT` received,
 - there's a handle to close during the walk,
 - handle got closed
 - `uv_getaddrinfo` completed
 - TCP connection established
 - buffer allocation for standard input read requested
 - buffer allocation for TCP socket read requested
 - data read from the standard input
 - data written to the standard output
 - data read from the TCP socket
 - data written to the TCP socket
 
`uv_getaddrinfo` is the first event that should happen. If it succeeded, it will try to open a TCP connection. If a TCP connection gets successfully open, listening for data on the standard input and the TCP socket will start.
 
	getaddrinfo	-->	open TCP connection	-->	read stdin		-->	write TCP socket
										-->	read TCP socekt	--> write stdout
										
Additional comments can be found in the code below:

    // Signal handling -- just stop the main loop
    static void on_signal(uv_signal_t *handle, int signum){
        uv_stop(&loop);
    }

    // Callback for each handle that is walked on
	// on_close callback stops the main loop if all handles are closed
    void on_walk(uv_handle_t* handle, void* arg){
        uv_close(handle, on_close);
    }

    // Callback for closing a handle during the walk
	// Stops the main loop when there are no more active handles
	// If a handle is closed beforehand, uv_stop is called multiple times because ...
	// ... the walk will continue until all the handles get walked over
    void on_close(uv_handle_t* handle){
        if(!loop.active_handles){
            uv_stop(&loop);
        }
    }

    // Once the destination IP and port are successfully resolved, ...
    // ... a request is made to open the TCP connection
    void on_getaddrinfo(uv_getaddrinfo_t* req, int status, struct addrinfo* res){
        check_uv(status);
        addr = *res->ai_addr;
        uv_freeaddrinfo(res);
        check_uv(uv_tcp_connect(&connect_req, &tcp, &addr, on_connect));
    }

    // Once the TCP connection is successfully established, ...
    // ... stdin and the TCP socket are being listened on for data
    void on_connect(uv_connect_t *connection, int status){
        check_uv(status);
        char node[INET_ADDRSTRLEN];
        check_uv(uv_ip4_name((const struct sockaddr_in *) &addr, node, INET_ADDRSTRLEN));
        printf("Connected to %s on port %hd!\n", node, ntohs(((struct sockaddr_in*)&addr)->sin_port));
        check_uv(uv_read_start(connection->handle, on_alloc_tcp, on_tcp_read));
        check_uv(uv_read_start((uv_stream_t*)&stdin_pipe, on_alloc_stdin, on_stdin_read));
    }

    // Buffer allocation for stdin reads / TCP writes
    // There's no need to allocate a new buffer every time, so the global one is reused
    void on_alloc_stdin(uv_handle_t* handle, size_t suggested_size, uv_buf_t* buf){
        *buf = write_buffer;
    }

    // Buffer allocation for TCP reads
    // There's no need to allocate a new buffer every time, so the global one is reused
    void on_alloc_tcp(uv_handle_t* handle, size_t suggested_size, uv_buf_t* buf){
        *buf = read_buffer;
    }

	// Callback for a read on stdin
    void on_stdin_read(uv_stream_t* stream, ssize_t nread, const uv_buf_t* buf){
        if (nread > 0){									// Valid data is read
            uv_write_t *req;
            if(!(req = malloc(sizeof(uv_write_t)))){	// New request must be allocated as well as ...
                memory_error();							// ... a buffer of nread size
			}											// Don't want to send BUFFER_LEN bytes
            uv_buf_t buffer = uv_buf_init(malloc(nread), nread);
            memcpy(buffer.base, buf->base, nread);
            req->data = buffer.base;
            check_uv(uv_write(req, connect_req.handle, &buffer, 1, on_tcp_write)); // Write to the TCP socekt
        } else if (nread < 0) {							// EOF is probably read -- the pipe is closed
            if (nread == UV_EOF){
                uv_stop(&loop);
            }
        }
    }

	// Callback for a write on stdout
	// After the write is done, allocated memory for the request is freed
    void on_stdout_write(uv_write_t* req, int status){
        check_uv(status);
        if(req && req->data){
            free(req->data);
		}
        if(req){
            free(req);
		}
    }

	// Callback for a read from the TCP socekt
    void on_tcp_read(uv_stream_t* handle, ssize_t nread, const uv_buf_t* buf){
        if (nread > 0){									// Valid data is read
            uv_write_t *req;
            if(!(req = malloc(sizeof(uv_write_t)))){	// New request must be allocated as well as ...
                memory_error();							// ... a buffer of nread size
			}											// Don't want to print BUFFER_LEN bytes
            uv_buf_t buffer = uv_buf_init(malloc(nread), nread);
            memcpy(buffer.base, buf->base, nread);
            req->data = buffer.base;
            check_uv(uv_write(req, (uv_stream_t*)&stdout_pipe, &buffer, 1, on_stdout_write)); // Write to stdout
        } else if (nread < 0) {							// EOF is probably read -- the socket is closed
            if (nread == UV_EOF){
                uv_stop(&loop);
            }
        }
    }

	// Callback for a write on the TCP socekt
	// After the write is done, allocated memory for the request is freed
    void on_tcp_write(uv_write_t* req, int status){
        check_uv(status);
        if(req && req->data){
            free(req->data);
	    }
        if(req){
            free(req);
	    }
    }

### Complete code

The complete code with the appropriate includes and defines can be found [in my GitHub repo](https://github.com/ChubbyHunteR/blog-code).

## Compilation and testing

Linking against `libuv` is required, so compilation with `gcc main.c -luv` should work.

I've used `valgrind` to test for memory leaks while sending a file to an instance of `netcat`. `diff` confirmed that the files indeed were transferred correctly.

In one terminal run `netcat -ltp 12345 > file2`. In the other run `cat file | valgrind ./a.out`. Finally, compare `file` and `file2` with `diff file file2`.

`valgrind` should say that there are no memory leaks and `diff` should say nothing, because the files should be the same.