
```
import socket
import select
import struct
import keyboard
import time

# define global close
close = False

# function to open the connection
def openConnection(host, port):
    try:
        # Create a TCP/IP socket
        client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        
        # Connect the socket to the server
        client_socket.connect((host, port))

        # Return the connected socket
        return client_socket
    
    except socket.error as e:
        print("Connection error", e)
        return None

# function to close the connection
def closeConnection(client_socket):
    # close the connection
    client_socket.close()
    print("Connection destroyed successfully")

# function to read the 1st four bytes and see how long the message is
def parseHeader(client_socket):
    ready_to_read = False
    iterationCount = 0

    # check 10 times to see if ready to read
    while not ready_to_read:
        iterationCount += 1

        # Check if there are bytes available
        ready_to_read, _, _ = select.select([client_socket], [], [], 0.1)

        if iterationCount > 9:
            raise ValueError("Received less than 4 bytes")
            time.sleep(0.001)
        

    # get 4 bytes and cast into a 32bit int
    data = client_socket.recv(4)  # Read the first 4 bytes

    if len(data) < 4:
        raise ValueError("Received less than 4 bytes")

    # Unpack the bytes as a 32-bit integer
    packetLength = int.from_bytes(data, byteorder='big')
    return packetLength

# function to read the data frame
def readFrame(client_socket, packetLength):
    # read the defined number of bytes from the connection
    data = client_socket.recv(packetLength)

    # Decode the byte string
    data = data.decode('utf-8').strip()
    return data

if __name__ == "__main__":
    host = "localhost"
    port = 2055
    client_socket = openConnection(host, port)

    if client_socket:
        try:
            while not close:
                # Obtain the size (in bytes) of the dataframe
                packetLength = parseHeader(client_socket)

                # read the data frame
                data = readFrame(client_socket, packetLength)

                # Process data here
                print(data)

                # Check if escape key has been pressed
                if keyboard.is_pressed('esc'):
                    close = True

                # wait 1 ms
                #time.sleep(0.001)

        except Exception as e:
            print("Error:", e)

        finally:
            # close if close is set to true
            closeConnection(client_socket)

```