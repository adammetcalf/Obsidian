
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


A better class based version:
```
import socket
import select
import struct
import keyboard
import time

class SocketClient:

    # Constructor
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.client_socket = None
        self.close = False

    # Open TCP connection
    def open_connection(self):
        try:
            self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.client_socket.connect((self.host, self.port))
        except socket.error as e:
            print("Connection error:", e)
            self.client_socket = None

    # Close connection 
    def close_connection(self):
        if self.client_socket:
            self.client_socket.close()
            print("Connection destroyed successfully")

    # Read first 4 bytes to get frame length
    def parse_header(self):
        ready_to_read = False
        iteration_count = 0

        while not ready_to_read:
            iteration_count += 1
            ready_to_read, _, _ = select.select([self.client_socket], [], [], 0.1)
            if iteration_count > 9:
                raise ValueError("Received less than 4 bytes")
            time.sleep(0.001)

        data = self.client_socket.recv(4)
        if len(data) < 4:
            raise ValueError("Received less than 4 bytes")

        packet_length = int.from_bytes(data, byteorder='big')
        return packet_length

    # Read data frame
    def read_frame(self, packet_length):
        data = self.client_socket.recv(packet_length)
        data = data.decode('utf-8').strip()
        return data

    # Main running code
    def run(self):

        #Open Connection
        self.open_connection()

        # if valid connection continue, else quit
        if self.client_socket:
            try:
                while not self.close:

                    # Get size (in bytes) of dataframe
                    packet_length = self.parse_header()

                    # Read data frame
                    data = self.read_frame(packet_length)
                    print(data)

                    # Check whether to continue
                    if keyboard.is_pressed('esc'):
                        self.close = True

            except Exception as e:
                print("Error:", e)

            finally:

                # Destroy connection
                self.close_connection()
        else:
            print("Connection fail")       


if __name__ == "__main__":
    client = SocketClient(host="localhost", port=2055)
    client.run()

```


A better version with visualisation, error handling and a UI:

```
import socket
import select
import keyboard
import time
import numpy as np
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import tkinter as tk
from threading import Thread

class SocketClient:

    # Constructor
    def __init__(self, host, port):
        self.host = host
        self.port = port
        self.client_socket = None
        self.close = False
        self.shape = []

        # Initialize Tkinter
        self.root = tk.Tk()
        self.root.title("3D Shape Viewer")

        # Create matplotlib figure and axis
        self.fig = plt.figure()
        self.ax = self.fig.add_subplot(111, projection='3d')

        # Create a canvas to display the matplotlib figure
        self.canvas = FigureCanvasTkAgg(self.fig, master=self.root)
        self.canvas.draw()
        self.canvas.get_tk_widget().pack(side=tk.TOP, fill=tk.BOTH, expand=1)

        # Add 'quit' button
        self.quit_button = tk.Button(master=self.root, text="Quit", command=self.quit)
        self.quit_button.pack(side=tk.BOTTOM)

        # Add status indicator
        self.status_indicator = tk.Label(master=self.root, text="Status", bg="green", width=10)
        self.status_indicator.pack(side=tk.BOTTOM)

    # Open TCP connection
    def open_connection(self):
        try:
            self.client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.client_socket.connect((self.host, self.port))
            self.client_socket.setblocking(0)
            self.flush_socket()
        except socket.error as e:
            print("Connection error:", e)
            self.client_socket = None

    # Flush any remaining data in the socket
    def flush_socket(self):
        while True:
            ready_to_read, _, _ = select.select([self.client_socket], [], [], 0.1)
            if ready_to_read:
                try:
                    data = self.client_socket.recv(1024)
                    if not data:
                        break
                except socket.error:
                    break
            else:
                break
        self.client_socket.setblocking(1)

    # Close connection 
    def close_connection(self):
        if self.client_socket:
            self.client_socket.close()
            print("Connection destroyed successfully")

    # Read first 4 bytes to get frame length
    def parse_header(self):
        ready_to_read = False
        iteration_count = 0

        while not ready_to_read:
            iteration_count += 1
            ready_to_read, _, _ = select.select([self.client_socket], [], [], 0.1)
            if iteration_count > 9:
                raise ValueError("Received less than 4 bytes")
            time.sleep(0.001)

        data = self.client_socket.recv(4)
        if len(data) < 4:
            raise ValueError("Received less than 4 bytes")

        packet_length = int.from_bytes(data, byteorder='big')
        return packet_length

    # Read data frame
    def read_frame(self, packet_length):
        data = self.client_socket.recv(packet_length)
        return data.decode('utf-8').strip()

    # Parse the received data to extract shape arrays
    def parse_data(self, data):
        
        # data frame is tab delimited. God knows why there isnt a json definition or something equally searchable
        lines = data.split('\t')

        try:
            shape_x_start = lines.index('Shape x [cm]') + 1
            shape_y_start = lines.index('Shape y [cm]') + 1
            shape_z_start = lines.index('Shape z [cm]') + 1
        except ValueError as e:
            print(f"Error finding indices: {e}")
            return

        def find_errors(label):
            indices = [(i, i+1) for i in range(len(lines) - 1) if lines[i] == label[0] and lines[i+1] == label[1]]
            starts = [index + 2 for index, _ in indices]
            values = []
            for start in starts:
                # Handle potential missing values or extraneous data
                value_group = []
                for j in range(4):
                    try:
                        value_group.append(int(lines[start + j]))
                    except (ValueError, IndexError):
                        value_group.append(0)  # Default to 0 if there is an issue
                values.extend(value_group)
            return values

        try:
            error_labels = ['1 18', '2 18', '3 18', '4 18']
            all_errors = []

            for label in error_labels:
                all_errors.extend(find_errors(label.split()))

            # Check the condition for the indicator
            if all_errors.count(0) == 16:
                self.status_indicator.config(bg="green")
            else:
                self.status_indicator.config(bg="red")

        except ValueError as e:
            print(f"Error finding error indices: {e}")
            return

        shape_x = [float(x) for x in lines[shape_x_start:shape_y_start - 1]]
        shape_y = [float(y) for y in lines[shape_y_start:shape_z_start - 1]]
        shape_z = [float(z) for z in lines[shape_z_start:]]

        # Remove the first value from each list
        shape_x = shape_x[1:]
        shape_y = shape_y[1:]
        shape_z = shape_z[1:]

        self.ax.clear()

        # Set axis limits
        self.ax.set_xlim([-25, 25])
        self.ax.set_ylim([-25, 25])
        self.ax.set_zlim([-25, 25])

        self.ax.plot(shape_x, shape_y, shape_z)

        self.ax.set_xlabel('x (cm)')
        self.ax.set_ylabel('y (cm)')
        self.ax.set_zlabel('z (cm)')
        self.canvas.draw()

        # Combine shape_x, shape_y, and shape_z into shape
        self.shape = [[x, y, z] for x, y, z in zip(shape_x, shape_y, shape_z)]

    # Main running code
    def run(self):
        # Open Connection
        self.open_connection()

        frame_counter = 0  # Initialize frame counter

        # if valid connection continue, else quit
        if self.client_socket:
            try:
                while not self.close:
                    # Get size (in bytes) of dataframe
                    packet_length = self.parse_header()

                    # Read data frame
                    data = self.read_frame(packet_length)

                    # Check if the current frame is the 10th one
                    if frame_counter % 1 == 0:
                        # Parse data to extract shape arrays
                        self.parse_data(data)
                        # print("Combined shape:", self.shape)

                    frame_counter += 1  # Increment frame counter

                    # Check whether to continue
                    if keyboard.is_pressed('esc'):
                        self.close = True

            except Exception as e:
                print("Error:", e)

            finally:
                # Destroy connection
                self.close_connection()
        else:
            print("Connection fail")       

    def quit(self):
        self.close = True
        self.root.quit()

if __name__ == "__main__":
    client = SocketClient(host="localhost", port=5000)
    # Run the client in a separate thread to avoid blocking the UI
    client_thread = Thread(target=client.run)
    client_thread.start()
    client.root.mainloop()
```