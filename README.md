# School Project N°5: TCP sockets programing (Subject: Computer Networking)

## Introduction

### Objective of the Practical Assignment 

The objective of this work is to implement a minimalist web server that enables file transfers between devices connected to the same local network. The server must allow files to be uploaded or downloaded depending on the mode selected prior to execution, and it must display a QR code in the console to facilitate connection to the server from another device. This project explores the application layer of the TCP/IP model and the functioning of TCP sockets. 
The two operation modes of the server are:
- Download: Allows a client to download a specific file. To run the server in this mode, execute the command: `python server_fileTransfer.py download {filename_to_download}`.
- Upload: Allows a client to upload a file to the server. To run the server in this mode, execute the command: `python server_fileTransfer.py upload`.

### Context of the Application Layer in the TCP/IP Model

In the “top-down approach” of the protocol stack proposed by Kurose, the application layer is the highest level. At this layer, applications exchange messages to implement services and provide support using transport layer services. One of the application protocols is HTTP, used for requests and the transfer of web documents.
Processes are programs running on hosts that communicate via sockets. Hosts communicate by exchanging messages (Inter-Process Communication), and this message exchange is determined primarily by application layer protocols.
In a client-server web application architecture (as required in this assignment), there is an always-on host with a known fixed IP address called the server, which responds to requests for objects from other hosts running their respective browsers, known as the clients. In addition to IP addresses, each host distinguishes which process should receive a message using numbered ports. For example, a web server is identified by port 80 (the default port for the HTTP protocol).
A socket is a combination of an IP address and a port, acting as a connection point between two systems and identifying processes. Processes send and receive messages through sockets. There are two sockets involved in a connection, one at each end. In the context of our assignment, we use stream sockets implemented over the TCP transport protocol. These sockets are identified by a 4-tuple containing both the source and destination IP addresses and ports. Additionally, the server has a listening socket through which it receives new connection requests.
The transport layer is responsible for data transfer between processes. One transport layer protocol is TCP (reliable, connection-oriented). In our work, we implement TCP/IP, where IP is a network-level protocol that uses routing algorithms to route and address data packets so they can travel across networks and reach their correct destination.

The application layer in the TCP/IP model is the point where applications interact with the network. The application defines the content and context, deciding what type of data is sent (text, commands, files, forms), how it is interpreted, and how the user or software interacts. Also, some important protocols and security measures are applied at the application layer.

### Purpose of the Server 

The purpose of the implemented server is to exchange files between different hosts on the same network reliably (TCP/IP provides guarantees against loss, duplicates, reordering, corrupted data, flow control, and congestion), while providing a simple web interface (under the HTTP protocol) to facilitate the exchange.

## Implementation

### QR Code Generation

The qrcode library includes a method to print generated QRs to the console using ASCII art: a way of concatenating characters—in this case, squares and empty spaces—to replicate an image. Because of this, implementing the function was quite straightforward.

### Handling Download and Upload Requests

Given our understanding of HTTP response syntax from the course, the first challenge we faced was how to construct it in Python. We learned that data is sent through sockets in bytes, not strings, so we had to use b"..." or .encode() to translate the text. With this in mind, and after researching how to build responses in Python, we constructed the respective error cases (404 Not Found for downloads and 500 Internal Server Error for uploads) and the 200 OK success case for both.
Additionally, we built an HTML file to allow users to upload files again after a successful upload; otherwise, the server would keep running but would be unable to receive new requests. In this file, clicking "Back" requests a new connection and sends an HTTP GET message to request the file upload page.

### Server Initialization

Initializing the server was one of the most labor-intensive parts of the project. To implement this function, we researched various documents to understand socket handling and connections in Python, utilizing a trial-and-error approach.
First, we created the listening socket to receive connection requests. We obtained the local network IP address and assigned a random port number for each new connection. Although we initially considered a fixed port, we decided that assigning a random number (using the random library) was a better option to avoid potential conflicts, as every connection must have a unique IP/port tuple. The port numbers used range between 1024 and 8192 for simplicity.
As a web server using the HTTP protocol, the server-side port should ideally be port 80. However, port 80 is a privileged port (below 1024) and cannot be bound directly in Python without root privileges, which was not the case for our environment.
It is important to clarify that the TCP/IP model used in this project employs non-persistent connections. Therefore, a new connection is established for every HTTP message and closed once the response is sent. When a connection terminates, the server returns to a listening state for the next request.

The next step was to provide the QR code and URL to the client. When the client opens the URL and requests a connection, the server accepts it and inspects the HTTP request to verify the requested mode. After decoding the message and filtering for errors, we generated the response.
The response generation was based on the HTTP method:
- GET: If the client requests the home page (path = /), we send the corresponding HTML file. If the client requests a file download (path = /download), the response body contains the file. Both are sent with a 200 OK status.
- POST: This indicates a file upload. We decode the message, save the file, and send a success response.
- Errors: If an invalid method is used, we respond with a 405 Method Not Allowed message.

We encountered an issue where the \r\n\r\n sequence was missing from the HTTP message because the browser would request favicon.ico, crashing the server. We decided to reject these requests by mandating that the message must contain \r\n\r\n.
Initially, our server only read the first 4096 bytes of the requests and continued reading only for POST methods. However, with large files, the server would hang because it failed to read the full headers. We solved this by reading the entire message as bytes and, for POST requests, verifying completion using the Content-Length header.

### Protocol and HTTP Header Analysis (Wireshark)

The HTTP packet capture using Wireshark can be found in the following document: analisis_HTTP_wireshark.pdf.
In both upload and download cases, we observed two messages and two responses (ignoring rejected favicon requests).
- Download: A GET is used to request the initial HTML page and another to download the specific file. The payloads contain the requested files.
- Upload: A GET is used for the initial page, followed by a POST to send the file data to the server.
- For GET messages, headers included: Host, Connection, Upgrade-Insecure-Requests, User-Agent, Accept, Accept-Encoding, and Accept-Language.
- POST messages included the same, plus Content-Length, Cache-Control, Origin, Content-Type, and Referer.
- All responses included Content-Type, Content-Length, and Connection.

### Gzip Compression Implementation

Gzip is a compression algorithm that works on byte sequences to reduce file size. Integrating it was simple:
We added a command-line option to enable Gzip when starting the server.
We checked the Accept-Encoding header to see if the client supports it.
We modified the handle_download() function accordingly.

Here is the translation of the Experimentation section into English:

## Experimentation
To carry out the experimentation, we started the web server in Download mode, passing various files as parameters. We then connected from another device and took a sample of 10 downloads. We captured the packets for each of these downloads using Wireshark and measured the transfer time as the arrival time of the response minus the arrival time of the request. Once the data was collected and classified, we built a dataset, generated a boxplot in R, and compared the transfer times of each file with and without Gzip.

### Images
(Please note that for this part of the experimentation, a virtual machine was used)

For this part of the project, we will use the .jpg format, which already applies lossy compression to images. Therefore, Gzip compression will not significantly reduce the size of the sent files, and we expect that transfer times with and without compression will not differ greatly. We used the following small, medium, and large images:

<img width="427" height="145" alt="image" src="https://github.com/user-attachments/assets/6e0f1cc4-164e-48b9-8680-42bc2c993f04" />

### Transfer Time (measured in milliseconds):

<img width="427" height="142" alt="image" src="https://github.com/user-attachments/assets/84b978ce-f92f-4396-8a82-e317e72b8148" />
<img width="427" height="142" alt="image" src="https://github.com/user-attachments/assets/f2f7d150-4b51-454d-9fa8-86f12f08765d" />
<img width="427" height="216" alt="image" src="https://github.com/user-attachments/assets/18cd767c-3d0c-466b-8197-fd3cf2a76a31" />

We can observe that when the file is small, transfer times are significantly lower when the file is compressed. However, as the file size increases, transfers without compression take less time. Therefore, we conclude that when the content to be compressed is larger, the time required for the compression process itself ends up negatively impacting the total transfer time, as the reduction in file size does not compensate for the time invested in compression.

<img width="427" height="142" alt="image" src="https://github.com/user-attachments/assets/8255f93c-bf60-4c9d-bec5-9b973f154172" />
<img width="427" height="81" alt="image" src="https://github.com/user-attachments/assets/87ff8960-c767-47b9-8857-393fce5f7c49" />

### Documents
(Please note that for this part of the experimentation, a virtual machine was not used)

On the other hand, we will continue our experimentation using various books (in .html format to allow for greater compression potential) obtained from Project Gutenberg: “The Raven” by Edgar Allan Poe (small), “Dracula” by Bram Stoker (medium), and “The Complete Works of William Shakespeare” (large).

### Transfer Time (measured in milliseconds):

<img width="427" height="302" alt="image" src="https://github.com/user-attachments/assets/332fcbbe-0061-496b-b8da-e0ea05fe217d" />
<img width="308" height="273" alt="image" src="https://github.com/user-attachments/assets/c47d1264-7118-4cd7-8806-45c841f054f3" />

Compression significantly reduces transfer time in the case of the small document. However, the transfer time increases considerably for both medium and large files. We attribute this to the fact that the decrease in transmission time from the server to the client fails to compensate for the computational time required to reduce the file size.

<img width="426" height="219" alt="image" src="https://github.com/user-attachments/assets/ad788c1b-f3e4-490e-9a9c-c9bf2946d004" />

## Security

The server has several significant vulnerabilities as it uses unencrypted HTTP, lacks input validation, and handles connections sequentially. Given that the Internet was originally designed to be open rather than secure, these weaknesses allow an attacker (Trudy) to compromise the confidentiality, integrity, and availability of the service. Below, we highlight some of the ways these objectives could be achieved.

### Traffic Interception (Sniffing)

This type of attack affects confidentiality. The server uses HTTP without TLS (HTTPS is the extension of HTTP that encrypts using TLS); therefore, all data travels in plain text, including files sent via POST and content requested via GET. Trudy can capture the traffic with tools like Wireshark (just as we did) and read files, usernames, passwords, or any transmitted information. This threat stems from the total absence of encryption. To fix this, the socket should be replaced with a secure one using TLS, allowing the server to operate over HTTPS, rendering the traffic unreadable to attackers.

### Data Modification and Corruption

This issue affects the integrity of the server. During file uploads, the server does not validate the filename or the content of the file. This allows Trudy to send filenames that could navigate through the computer's directories (Directory Traversal) and overwrite legitimate files. This vulnerability arises from the lack of controls over user input. It could be solved by sanitizing the filename, verifying the file size and type, and rejecting requests with inconsistent headers.

### Denial of Service (DoS) Attack

This attack makes the server unavailable. The server handles one client at a time and has no size limits or timeouts on connections. Trudy can send an extremely large POST request or transmit data very slowly (Slowloris style), causing the server to block so no one else can use it. This exploits the lack of limits on body reading and the absence of maximum wait times. To mitigate this, a maximum allowed size for uploads should be imposed, slow connections should be closed, and the architecture should prevent a single connection from blocking the entire service.

### Extra: (Attempted) Implementation of a Basic Security Method

We decided to follow the example proposed in the assignment and build a password-based authentication method.
Front-end: We modified the generated HTML file to include a password input field for the client.
Mechanism: The client sends the password as part of the GET or POST path (e.g., GET /download?pass=1234 HTTP/1.1). While not entirely secure, it fulfills the requirement.
Server Logic: The server extracts the "1234" string, validates it, and decides the outcome. If the password is incorrect, it responds with a 403 Forbidden status; otherwise, it proceeds with the request.
We modified the server_start() function to account for this in both server modes. The chosen password was td2025. The file containing these modifications is titled server_fileTransfer_contraseña.py.

## Conclusion

Throughout this project, we explored the practical side of applications that we had previously studied theoretically in the course, yet were unfamiliar with in terms of real-world operation. This experience, combined with the other concepts we learned, has significantly increased the value of the course for us. The primary takeaways were the practical application of protocols in device interaction, an understanding of how a web server is structured and functions (something we use daily while ignoring the underlying mechanics), and the management of sockets and HTTP messages in Python. Furthermore, we previously did not know how file uploading and downloading functioned on the Internet, and this assignment helped us understand these processes much more clearly.

We encountered many technical challenges during implementation, leading us to spend considerable time debugging and troubleshooting failures. This process forced us to restructure the server code multiple times until we found a solution that was both convincing and efficient. We identified several potential improvements that we ultimately did not implement, as they would have increased the complexity of the code and the scope of the project beyond the available time. These include:
- Handling concurrent connections (more than one at a time).
- Filename validation (to avoid collisions) and content verification for uploaded files.
- Upload size limits.
- Sending files in chunks (instead of reading the entire file into memory).
- Proper favicon handling (which we chose to ignore).
