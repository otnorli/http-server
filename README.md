# http-server

=begin
Introduction:
Code written by Ole Norli 

Program description:
HTTP server.

Current progress:
Program currently supports GET requests
=end

# include socket
require 'socket'

# include uri, lib for parsing request-uri into path to a file on server
require 'uri'

# Files served from this directory:
ROOT_DIR = 'C:\Users\Ole\Desktop\ruby'

# Extensions. Allow html or plain text, png or jpg images
# (Not in use jet)
#CONTENT_TYPE_MAPPING = {
#  'html' => 'text/html',
#  'txt' => 'text/plain',
#  'png' => 'image/png',
#  'jpg' => 'image/jpeg'
#}
#DEFAULT_CONTENT_TYPE = 'application/octet-stream'

# Function to generate path to file on our server
def requested_file(request_line)
   # Simple split up of the request
   request_uri  = request_line.split(" ")[1] # GET request, request will be [1]
   path = URI.unescape(URI(request_uri).path)

   # Split up string in chars
   sec_path = []
   sec_path = path.split(//)

   # Convert / to \ 
   i = 0
   for element in sec_path
      if element == "/"
         sec_path[i] = "\\"
      end
      i = i+1
   end

   # Join chars together in string
   path = sec_path.join

   # Combine with root directory
   File.join(ROOT_DIR, path) # return path
end

# Open a socket localhost in port 2345. PS: 80 is standard for web
s = TCPServer.new('localhost', 2345)

# while (1)
loop do 

   # Accept incomming client request, one at a time
   socket = s.accept 
   
   # Read request from client. Recieves up to 1000 bytes, which is more than enough
   request = socket.recv(1000) 

   # Logs request
   STDERR.puts request 

   # Get adress of requested file, and add index.html if its not present
   adress = requested_file(request)

   # add index.html to request file if only folder requested
   adress = File.join(adress, 'index.html') if File.directory?(adress)

   # If this file exist, return it
   if File.exist?(adress) && !File.directory?(adress)
      # Output success if we found file
      File.open(adress, "rb") do |file|
         socket.write("HTTP/1.1 200 OK\r\n" +
                "Content-Type: text/html; charset=utf-8\r\n" +
                "Content-Length: #{file.size}\r\n" +
                "Connection: close\r\n\r\n")
         IO.copy_stream(file, socket)
      end

   # if it doesnt exist, return error
   else
      # Define response
      respons = String.new()
      respons.concat("File not found. Requested file: ")
      respons = respons + adress

      # Output error if we didnt find file
      socket.write("HTTP/1.1 404 Not Found\r\n" +
                 "Content-Type: text/plain\r\n" +
                 "Content-Length: #{respons.size}\r\n" +
                 "Connection: close\r\n\r\n" + respons)
   end

   # Close client connection
   socket.close
end

# Finish program
s.close
