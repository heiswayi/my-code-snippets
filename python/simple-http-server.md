Script to create a simple HTTP server

```python
import argparse
import http.server
import socketserver
import logging
import os

class CustomHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
    def log_message(self, format, *args):
        logging.info("%s - - [%s] %s\n" %
                     (self.client_address[0],
                      self.log_date_time_string(),
                      format % args))

def start_server(directory, port, mode):
    os.chdir(directory)
    handler_class = CustomHTTPRequestHandler
    
    if mode == 'file':
        handler_class = http.server.SimpleHTTPRequestHandler
    
    with socketserver.TCPServer(("", port), handler_class) as httpd:
        logging.info("Serving HTTP on port %d (directory: %s) in %s mode", port, directory, mode)
        try:
            httpd.serve_forever()
        except KeyboardInterrupt:
            logging.info("Keyboard interrupt received, stopping server")
            httpd.server_close()
            logging.info("Server stopped")
        httpd.server_close()
        logging.info("Stopping server")

def main():
    parser = argparse.ArgumentParser(description="Simple Python web server")
    parser.add_argument("-d", "--directory", type=str, default=os.getcwd(),
                        help="Directory to serve files from")
    parser.add_argument("-p", "--port", type=int, default=8000,
                        help="Port to listen on")
    parser.add_argument("-m", "--mode", type=str, choices=['web', 'file'], default='web',
                        help="Server mode: 'web' for web server, 'file' for file server")
    
    args = parser.parse_args()
    
    logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')
    
    start_server(args.directory, args.port, args.mode)

if __name__ == "__main__":
    main()
```