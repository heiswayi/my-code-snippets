Simple NAS Server-Client using Python script and `socket` library

`nas-server.py`

```python
import socket
import os

# Server configuration
SERVER_HOST = '127.0.0.1'  # Loopback address (localhost)
SERVER_PORT = 12345          # Port to listen on
BUFFER_SIZE = 1024           # Size of the receive buffer

# Function to handle file uploads
def handle_upload(client_socket, filename):
    with open(filename, 'wb') as f:
        while True:
            data = client_socket.recv(BUFFER_SIZE)
            if not data:
                break
            f.write(data)
    print(f"File '{filename}' uploaded successfully.")

# Function to handle file downloads
def handle_download(client_socket, filename):
    if os.path.exists(filename):
        with open(filename, 'rb') as f:
            data = f.read(BUFFER_SIZE)
            while data:
                client_socket.send(data)
                data = f.read(BUFFER_SIZE)
            print(f"File '{filename}' sent successfully.")
    else:
        print(f"File '{filename}' not found.")

# Main server function
def main():
    # Create socket
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((SERVER_HOST, SERVER_PORT))
    server_socket.listen(1)  # Listen for incoming connections

    print(f"Server listening on {SERVER_HOST}:{SERVER_PORT}...")

    while True:
        client_socket, client_address = server_socket.accept()
        print(f"Connection from {client_address}")

        # Receive command from client
        command = client_socket.recv(BUFFER_SIZE).decode().strip()

        if command.startswith("UPLOAD"):
            _, _, filename = command.partition(' ')
            handle_upload(client_socket, filename)
        elif command.startswith("DOWNLOAD"):
            _, _, filename = command.partition(' ')
            handle_download(client_socket, filename)
        else:
            print("Invalid command.")

        # Close client connection
        client_socket.close()

if __name__ == "__main__":
    main()
```

`nas-client.py`

```python
import socket
import sys
import os

# Server configuration
SERVER_HOST = '127.0.0.1'  # Server IP address
SERVER_PORT = 12345        # Server port
BUFFER_SIZE = 1024         # Size of the receive buffer

# Function to upload a file to the server
def upload_file(filename):
    if not os.path.exists(filename):
        print(f"File '{filename}' does not exist.")
        return

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((SERVER_HOST, SERVER_PORT))
        s.sendall(f"UPLOAD {filename}".encode())
        
        with open(filename, 'rb') as f:
            while True:
                data = f.read(BUFFER_SIZE)
                if not data:
                    break
                s.sendall(data)
        
        print(f"File '{filename}' uploaded successfully.")

# Function to download a file from the server
def download_file(filename):
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((SERVER_HOST, SERVER_PORT))
        s.sendall(f"DOWNLOAD {filename}".encode())
        
        with open(filename, 'wb') as f:
            while True:
                data = s.recv(BUFFER_SIZE)
                if not data:
                    break
                f.write(data)
        
        print(f"File '{filename}' downloaded successfully.")

# Main client function
def main():
    if len(sys.argv) != 3:
        print("Usage: python nas-client.py [upload/download] [filename]")
        return
    
    command = sys.argv[1].lower()
    filename = sys.argv[2]
    
    if command == 'upload':
        upload_file(filename)
    elif command == 'download':
        download_file(filename)
    else:
        print("Invalid command. Use 'upload' or 'download'.")

if __name__ == "__main__":
    main()
```

Example:

```bash
python nas-client.py upload path/to/your/file
```

```bash
python nas-client.py download example.txt
```