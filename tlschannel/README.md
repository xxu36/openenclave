TLS-Channel
===========

This sample shows how to create a client-server application that uses an
attested TLS channel. The steps are as follows.

(1) Create attested credentials by running the gencreds tool, which
    creates two files: a certificate (DER) and a private key (PEM).

(2) Run the server program which waits for client connections. This program
    is an enclave application.

(3) Run the client program, whose command-line arguments include the files
    generated by the gencreds tool above. This program is an ordinary Linux
    application (which will eventually run within SGX-LKL).

Writing credential files to the untrusted system of course is not secure, but
remember that the gencreds tool and the client will eventually run inside
SGX-LKL where the credential files will be written to an enclave-resident file
system (under /var/run).

To build and run this sample type:

```
    make run
```

The following source files comprise a simple client-server SDK.

```
    cli/tlscli.h
    cli/tlscli.c
    srv/tlssrv.h
    srv/tlssrv.c
```

This SDK simplifies the creation of client-server applications, where
the client is an ordinary Linux application and the server is an enclave
application. See cli/cli.c and srv/srv.c for samples that use this SDK.

The following illustrates a simple client skeleton that uses this SDK.

```
    /* Initialize the SDK */
    tlscli_startup(&err);

    /* Connect to the server */
    tlscli_connect(host, port, cert, private_key, &cli, &err);

    /* Write message to the server */
    tlscli_write(cli, message, sizeof(message), &err)

    /* Read a message from the server */
    tlscli_read(cli, buf, sizeof(buf), &err);

    /* Destroy client object */
    tlscli_destroy(cli, &err);

    /* Shutdown the SDK */
    tlscli_shutdown(&err);
```

The server skeleton is shown here.

```
    /* Initialize the SDK */
    tlssrv_startup(&err);

    /* Create the server object */
    tlssrv_create(ip, port, _verify_identity, NULL, &srv, &err);

    /* Handle connections */
    for (;;)
    {
        /* Accept the next client connection */
        tlssrv_accept(srv, &conn, &err);

        /* Read a message from the client */
        tlssrv_read(srv, buf, sizeof(buf), &err);

        /* Write a message to the client */
        tlssrv_write(srv, buf, nbytes, &err));

        /* Terminate the client */
        mbedtls_net_free(&conn);
    }

    /* Destroy the server object */
    tlssrv_destroy(srv, &err);

    /* Shutdown the SDK */
    tlssrv_shutdown(&err);
```

The user-defined _verify_identity function verifies the identity of the peer.
This signature is shown below.

```
static oe_result_t _verify_identity(
    void* arg,
    const uint8_t* mrenclave,
    size_t mrenclave_size,
    const uint8_t* mrsigner,
    size_t mrsigner_size,
    const uint8_t* isvprodid,
    size_t isvprodid_size,
    uint64_t isvsvn)
```

This function should either verify the MRENCLAVE or the MRSIGINER of the client
application.

Note that the SDK is non-opaque; it exposes the underlying mbed TLS types. This
allows the developer to use the mbed TLS types directly when necessary.