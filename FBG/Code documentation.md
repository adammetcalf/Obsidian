Reading and converting the 1st 4 bytes to get the frame length:

![[LabVIEW_4_Bytes_FP.png]]
![[LabVIEW_4_Bytes_BD.png]]

The implementation in python:
```
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
```

The message in python:
b'2024/08/02\t09:39:38\t162304\t1\t1\t18\t0\t0\t0\t0\t1514.8491\t1518.7310\t1522.6240\t1526.5408\t1530.4740\t1534.3638\t1538.2476\t1542.1299\t1546.0118\t1549.9127\t1553.8159\t1557.7068\t1561.6129\t1565.5099\t1569.4160\t1573.2914\t1577.1964\t1581.0872\t30\t29\t43\t39\t45\t58\t68\t70\t64\t67\t69\t74\t60\t65\t54\t48\t57\t58\t18\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\tNaN\r\n'


