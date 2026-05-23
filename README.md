# HTTP File Transfer Server using TCP Sockets

Minimalist HTTP file transfer server implemented in Python using TCP sockets. The project supports file uploads and downloads between devices connected to the same local network, while exploring low-level networking concepts such as HTTP request handling, TCP communication and client-server architecture.

---

## Overview

This project was developed as part of a networking and distributed systems assignment focused on the application layer of the TCP/IP model.

The server allows users to:
- upload files to the host machine,
- download files from the server,
- access the service through a browser,
- connect using a generated QR code,
- and optionally enable Gzip compression for file transfers.

The implementation was built from scratch using Python sockets and manual HTTP response handling.

---

## Features

- File upload and download support
- HTTP request and response handling
- TCP socket programming in Python
- QR code generation for local access
- Optional Gzip compression
- Basic password-based authentication
- Wireshark traffic analysis
- Transfer-time experimentation and benchmarking

---

## Tech Stack

- Python
- TCP Sockets
- HTTP
- Wireshark
- R
- Gzip

---

## Running the Server

### Download Mode

```
python server_fileTransfer.py download <filename>
```
### Upload Mode
```
python server_fileTransfer.py upload
```
## How It Works

The server creates a listening TCP socket and waits for incoming HTTP requests from clients connected to the same local network.

Depending on the selected mode, the server:

- serves files to connected clients,
- receives uploaded files,
- or generates the corresponding HTML interface.

The project implements:

- manual HTTP request parsing,
- response generation,
- socket communication,
- header validation,
- and Content-Length handling for large file uploads.

Connections are non-persistent: a new TCP connection is created for each request and closed once the response is sent.

---

## Compression Experimentation

The project includes an experimental analysis of Gzip compression during file transfers.

Using Wireshark packet captures and transfer-time measurements, we compared compressed and uncompressed transfers across different file types and sizes.

### Main Findings
- Small files benefited from compression.
- Larger files often performed worse when compressed due to compression overhead.
- Already compressed formats such as .jpg showed minimal improvement.

The experimentation dataset was analyzed and visualized using R.

---

## Security Considerations

The server intentionally exposes several limitations and vulnerabilities commonly associated with basic HTTP services.

Some identified issues include:

- unencrypted HTTP traffic,
- lack of input sanitization,
- sequential request handling,
- absence of upload limits,
- and vulnerability to denial-of-service attacks.

The project includes an analysis of:

- traffic interception,
- file overwrite vulnerabilities,
- and Slowloris-style attacks.

A basic password authentication mechanism was also implemented as an additional extension.

---

## Key Learnings
- TCP socket programming
- HTTP protocol internals
- Client-server communication
- File transfer mechanisms
- Packet inspection with Wireshark
- Compression tradeoffs
- Network security fundamentals
- Debugging low-level networking systems

---

## Repository Structure
```
.
├── analisis_HTTP_wireshark.pdf
├── server_fileTransfer.py
├── server_fileTransfer_contraseña.py
├── archivos_servidor/
    ├── utdt_grande.jpg
    ├── utdt_mediana.jpg
    ├── utdt_pequeña.jpg
├── docs/
│   ├── report.pdf
└── README.md
```

---

## Future Improvements

Potential future extensions include:

- concurrent connection handling,
- chunked file transfers,
- HTTPS/TLS support,
- upload size limits,
- filename sanitization,
- and improved error handling.

---

## Author

Developed as a group project for a university networking course at Universidad Torcuato Di Tella. Forked from [taimouteo's tp_td4](https://github.com/taimouteo/tp_td4).
