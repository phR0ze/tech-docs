# Ngrok <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Ngrok is a networking tunneling tool that allows you to securely expose a local server to the 
internet through a public URL. It is expecially useful for development, testing, and demonstration 
purposes.

### Quick links
* [.. up dir](../README.md)
* [Overview](#overview)

## Overview
Ngrok tunnels your local server e.g. [localhost:3000](http://localhost:3000) to a publically 
accessible URL e.g. https://1234abcd.ngrok.io. Ngrok supports HTTP, HTTPS and TCP tunnels with 
end-to-end encryption. This is useful for:
* testing webhooks from services like Stripe, Twilio or Github that need a public endpoint to hit
* sharing a development site without deploying it
* running a temporary demo for a client or team
* debugging API traffic

### How it Works
1. Run your local server `localhost:3000`
2. Start ngrok: `ngrok http 3000`
3. Ngrok gives you a public URL `https://abs123.ngrok.io`
4. This endpoint is now available publically

## Getting started

### Hello World
1. In one shell launch a simple python HTTP server to serve up `Hello World`
   1. Create a simple python HTTP server
      ```python
      from http.server import BaseHTTPRequestHandler, HTTPServer

      class HelloHandler(BaseHTTPRequestHandler):
          def do_GET(self):
              self.send_response(200)  # HTTP status code 200 OK
              self.send_header("Content-type", "text/plain")
              self.end_headers()
              self.wfile.write(b"Hello World")  # Send response body

      def run(server_class=HTTPServer, handler_class=HelloHandler, port=3000):
          server_address = ('', port)
          httpd = server_class(server_address, handler_class)
          print(f"Serving on port {port}...")
          httpd.serve_forever()

      if __name__ == "__main__":
          run()
      ```
  2. Run python server
     ```bash
     $ python3 server.py
     ```
  3. Check the server is accessible `localhost:3000`
2. Now launch Ngrok with port 3000
   1. TBD
