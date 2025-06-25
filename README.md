# Academia Course Registration Portal

A secure, multi-user, role-based course registration system implemented in C. The portal uses a TCP client-server architecture: a central server listens on a network port and handles requests from multiple clients.

There are three user roles – **Admin**, **Faculty**, and **Student** – each with a password-protected login and distinct menu options for managing courses and accounts. The system stores all data persistently on the server and allows concurrent users to interact without conflicts.

---

## Features

* **User Authentication**: Login with a username and password for each role. Only authenticated users can access the system.
* **Role-Based Menus**:

  * Admins can add/remove students and faculty
  * Faculty can add/remove or update courses
  * Students can view courses and enroll/drop them
* **Course & Account Management**:

  * Admins can create or delete student and faculty accounts
  * Faculty can create or delete offered courses
  * Students can view available courses and enroll/drop
* **Concurrency**:

  * The server is multi-threaded and serves multiple clients at once.
  * Each client connection runs in its own thread enabling simultaneous, independent user sessions
* **Persistent Storage**:

  * All student, faculty, and course records are saved in binary files on disk
  * Binary files store data in machine format (0’s and 1’s), which is space-efficient and not human-readable
* **File Locking**:

  * Uses POSIX file locks (`fcntl`) to prevent race conditions
  * Example: a write lock on the enrollment file blocks other writes until the update is complete
* **Reliable Networking**:

  * Clients and server communicate over TCP sockets
  * TCP provides reliable, ordered delivery of a byte stream
  * Each application-level message is framed with a 4-byte length prefix so the receiver knows where each message ends
* **Error Handling**:

  * The portal validates all user inputs and prints clear error messages on invalid operations (e.g., trying to enroll in a non-existent course)

---

## Architecture

* **Client–Server Model**:

  * The server listens on a known TCP port (e.g., 7891)
  * Clients (`client.c`) connect via TCP (BSD sockets) from any host
  * Network I/O and protocol logic are handled in the client and server programs

* **Transport Protocol**:

  * Communication is over TCP (Transmission Control Protocol)
  * Provides reliable, ordered, error-checked data delivery

* **Message Framing**:

  * Every message is sent with a 4-byte big-endian integer header indicating the message length
  * This framing lets the receiver read exactly the right number of bytes for each logical message

* **Multithreading**:

  * When a client connects, the server spawns a new thread to handle that client’s session
  * Thread-per-connection model allows multiple clients (students/faculty) to interact simultaneously

* **Data Files**:

  * The server maintains binary files for students, faculty, and courses
  * Shared file format headers and data structures are defined in `academia.h`
  * File I/O routines in `file_ops.c` perform read/write on these files

---

## File Overview

* `server.c`:

  * Implements the server-side application
  * Sets up a listening socket, spawns threads for clients, authenticates users, and dispatches requests
  * Uses functions from `file_ops.c`

* `client.c`:

  * Implements the client-side application
  * Connects to the server’s socket, handles user interface (command-line menus and prompts), and sends/receives messages according to protocol (4-byte length + payload)

* `file_ops.c`:

  * Contains functions for operating on data files (e.g., loading student records, updating enrollments)
  * Implements persistent storage and uses file locking (`fcntl`) to serialize critical updates on disk

* `academia.h`:

  * Header file declaring shared data structures (`struct Student`, `struct Course`, etc.) and constants (file names, port number)
  * Includes function prototypes for utilities

---

## Setup and Compilation

Use `gcc` to compile the server and client programs:

```bash
gcc -o server server.c file_ops.c -pthread
gcc -o client client.c -pthread
```

This produces two executables: `server` and `client`.

---

## Usage

1. **Start the Server**
   Run in a terminal:

   ```bash
   ./server
   ```

2. **Run Clients**
   In separate terminals:

   ```bash
   ./client
   ```

3. **Login and Operate**

   * At the prompt, enter:

     * `1` for Admin
     * `2` for Faculty
     * `3` for Student
   * Provide the username and password
   * Follow menu instructions (e.g., add a student, enroll in a course)

4. **Concurrency**

   * You can run multiple `./client` sessions from different machines at the same time
   * The server handles them concurrently

---

## Security and Robustness

* **File Locking**:

  * Updates to shared files are bracketed with `F_WRLCK` / `F_RDLCK` using `fcntl`

* **SIGPIPE Handling**:

  * The server ignores `SIGPIPE` using `signal(SIGPIPE, SIG_IGN)`
  * Prevents crashes if a client disconnects unexpectedly

* **Input Validation**:

  * All input (from clients or files) is verified
  * Example: checking if student/faculty ID exists before modifying records

* **Password Protection**:

  * Accounts are password-protected
  * Incorrect password results in login rejection

---

## Screenshots

**Concurrent Sessions**:
Example showing concurrent client sessions:

* Left terminal: Admin performing user-management actions
* Right terminal: Student viewing available courses

The server’s multithreaded design allows these interactions to proceed simultaneously without conflict

---

## License

(Specify your license here, e.g., MIT License.)

---

## Authors

* Mit Narodia
* Prakhar Agrawal

---

## References

1. Handling multiple clients on server with multithreading using Socket Programming in C/C++
   [https://www.geeksforgeeks.org/handling-multiple-clients-on-server-with-multithreading-using-socket-programming-in-c-cpp/](https://www.geeksforgeeks.org/handling-multiple-clients-on-server-with-multithreading-using-socket-programming-in-c-cpp/)

2. Basics of File Handling in C
   [https://www.geeksforgeeks.org/basics-file-handling-c/](https://www.geeksforgeeks.org/basics-file-handling-c/)

3. File Locks (The GNU C Library)
   [https://www.gnu.org/software/libc/manual/html\_node/File-Locks.html](https://www.gnu.org/software/libc/manual/html_node/File-Locks.html)

4. Transmission Control Protocol - Wikipedia
   [https://en.wikipedia.org/wiki/Transmission\_Control\_Protocol](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

5. How to prevent SIGPIPEs (or handle them properly) - Stack Overflow
   [https://stackoverflow.com/questions/108183/how-to-prevent-sigpipes-or-handle-them-properly](https://stackoverflow.com/questions/108183/how-to-prevent-sigpipes-or-handle-them-properly)
