# microtar
A lightweight tar library written in ANSI C


## Basic Usage
The library consists of `microtar.c` and `microtar.h`. These two files can be
dropped into an existing project and compiled along with it.


#### File Reading
```c
mtar_t tar;
mtar_header_t h;
char *p;

/* Open archive for reading */
mtar_open(&tar, "test.tar", "r");

/* Print all file names and sizes */
while ( (mtar_read_header(&tar, &h)) != MTAR_ENULLRECORD ) {
  printf("%s (%d bytes)\n", h.name, h.size);
  mtar_next(&tar);
}

/* Load and print contents of file "test.txt" */
mtar_find(&tar, "test.txt", &h);
p = calloc(1, h.size + 1);
mtar_read_data(&tar, p, h.size);
printf("%s", p);
free(p);

/* Close archive */
mtar_close(&tar);
```

#### File Writing
```c
mtar_t tar;
const char *str1 = "Hello world";
const char *str2 = "Goodbye world";

/* Open archive for writing */
mtar_open(&tar, "test.tar", "w");

/* Write strings to files `test1.txt` and `test2.txt` */
mtar_write_file_header(&tar, "test1.txt", strlen(str1));
mtar_write_data(&tar, str1, strlen(str1));
mtar_write_file_header(&tar, "test2.txt", strlen(str2));
mtar_write_data(&tar, str2, strlen(str2));

/* Finalize -- this needs to be the last thing done before closing */
mtar_finalize(&tar);

/* Close archive */
mtar_close(&tar);
```

## No STDIO mode

Defining `NO_STDIO` will remove all the functions that use file IO.   Saves some space when you're only dealing with memory blocks.

## Memory Reading
Reading a tar file from a memory buffer is exactly the same as before except you open the tar using

```c
mtar_open_mem(&tar, &tarBuffer, "r")
```

where tarBuffer is a pointer to the start of the data

## Memory Writing
Writing a tar file in memory is almost the same.   Pass a NULL pointer to mtar_open_mem and it will allocate the memory as needed.
After finalising you must read back the size of the buffer that has been allocated.
mtar_close does not free the allocated buffer.

```c
mtar_t tar;
char* tarBuffer = NULL;
size_t tarBufferSize = 0;

const char *str1 = "Hello world";
const char *str2 = "Goodbye world";

/* Open archive for writing using memory buffer */
mtar_open_mem(&tar, tarBuffer, "w");

/* Write strings to files `test1.txt` and `test2.txt` */
mtar_write_file_header(&tar, "test1.txt", strlen(str1));
mtar_write_data(&tar, str1, strlen(str1));
mtar_write_file_header(&tar, "test2.txt", strlen(str2));
mtar_write_data(&tar, str2, strlen(str2));

/* Finalize -- this needs to be the last thing done before closing */
mtar_finalize(&tar);

/* Get the size of the final memory buffer */
tarBufferSize = mtar_mem_size(&tar);

/* Close archive */
mtar_close(&tar);

/* do something with tarBuffer here */

/* Free the buffer */
free(tarBuffer);

```


## Error handling
All functions which return an `int` will return `MTAR_ESUCCESS` if the operation
is successful. If an error occurs an error value less-than-zero will be
returned; this value can be passed to the function `mtar_strerror()` to get its
corresponding error string.


## Wrapping a stream
If you want to read or write from something other than a file, the `mtar_t`
struct can be manually initialized with your own callback functions and a
`stream` pointer.

All callback functions are passed a pointer to the `mtar_t` struct as their
first argument. They should return `MTAR_ESUCCESS` if the operation succeeds
without an error, or an integer below zero if an error occurs.

After the `stream` field has been set, all required callbacks have been set and
all unused fields have been zeroset the `mtar_t` struct can be safely used with
the microtar functions. `mtar_open` *should not* be called if the `mtar_t`
struct was initialized manually.

#### Reading
The following callbacks should be set for reading an archive from a stream:

Name    | Arguments                                | Description
--------|------------------------------------------|---------------------------
`read`  | `mtar_t *tar, void *data, unsigned size` | Read data from the stream
`seek`  | `mtar_t *tar, unsigned pos`              | Set the position indicator
`close` | `mtar_t *tar`                            | Close the stream

#### Writing
The following callbacks should be set for writing an archive to a stream:

Name    | Arguments                                      | Description
--------|------------------------------------------------|---------------------
`write` | `mtar_t *tar, const void *data, unsigned size` | Write data to the stream


## License
This library is free software; you can redistribute it and/or modify it under
the terms of the MIT license. See [LICENSE](LICENSE) for details.
