
What if you want your processes to be independent, to communicate, and yet not distribute some arcane shell scripts along with them? It turns out that other IPC mechanisms allow you to do all these. They are:

- [Named Pipes](https://goodyduru.github.io/os/2023/09/26/ipc-named-pipes.html)
- [Unix Domain Sockets](https://goodyduru.github.io/os/2023/10/03/ipc-unix-domain-sockets.html)
- [Unix Signals](https://goodyduru.github.io/os/2023/10/05/ipc-unix-signals.html)
- [Message Queues](https://goodyduru.github.io/os/2023/11/13/ipc-message-queues.html)
- Shared Memory
- Memory-Mapped Files

Files and TCP Sockets (which I call networks) are also IPC mechanisms, but I won’t discuss those. They are familiar, and there are lots of articles about them. `eventfd` is another mechanism, but the processes cannot be independent (they must share a parent process in the code).