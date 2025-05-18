# os-porject


This repository contains the code and data for an academic client-server project implemented in C using pthreads.

## Structure

- `client/` - Source and binary for client
- `server/` - Source and binary for server
- `academia/` - Common code (e.g., file operations)
- `data/` - `.dat` files for students, faculty, etc.
- `report/` - Project report PDF

## How to Compile

```bash
gcc -o client/client client/client.c academia/file_ops.c -lpthread
gcc -o server/server server/server.c academia/file_ops.c -lpthread
