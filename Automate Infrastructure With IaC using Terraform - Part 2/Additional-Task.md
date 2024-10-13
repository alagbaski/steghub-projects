# Networking Concepts Documentation

## 1. **Networking Concepts**

### IP Address
An **IP Address** (Internet Protocol Address) is a unique identifier assigned to each device connected to a network, allowing devices to communicate with one another. There are two types of IP addresses:
- **IPv4**: A 32-bit address written as four decimal numbers separated by dots (e.g., 192.168.1.1).
- **IPv6**: A 128-bit address written as eight groups of four hexadecimal digits separated by colons (e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334).

### Subnets
A **Subnet** (Subnetwork) is a logical subdivision of an IP network. Subnets help in organizing a large network into smaller segments, improving efficiency and security. Subnetting is often used to isolate networks from each other while reducing traffic. The size of a subnet is defined using the **subnet mask**, which determines how many bits of the IP address represent the network and how many represent the host.

### CIDR Notation
**CIDR (Classless Inter-Domain Routing)** is a method for assigning IP addresses and routing IP packets. CIDR allows more efficient allocation of IP addresses by removing the fixed class-based addressing system. CIDR notation specifies an IP address followed by a slash and a number (e.g., 192.168.1.0/24). The number after the slash represents how many bits are used for the network portion of the address.

### IP Routing
**IP Routing** is the process of determining the path that data packets take from the source device to the destination device across a network. Routers are network devices responsible for forwarding these data packets between different networks. The routing decision is based on the destination IP address and the routing table, which stores possible routes.

### Internet Gateways
An **Internet Gateway** is a networking device that connects a local network to the broader internet. It acts as the entry and exit point for data traffic between the local network and the internet, providing NAT (Network Address Translation) services and routing data to the correct destination.

### NAT (Network Address Translation)
**NAT** is a method that allows multiple devices on a private network to share a single public IP address for accessing external networks like the internet. NAT modifies the source IP address in outgoing packets and translates it back for incoming traffic, ensuring that data reaches the correct internal device.

---

## 2. **OSI Model and TCP/IP Suite**

### OSI Model
The **OSI Model** (Open Systems Interconnection Model) is a conceptual framework used to understand and standardize network communication across different systems. It divides networking into seven layers, each responsible for specific tasks:
1. **Physical Layer**: Handles the physical transmission of data (e.g., cables, radio frequencies).
2. **Data Link Layer**: Provides node-to-node data transfer and handles error detection and correction.
3. **Network Layer**: Responsible for routing, forwarding, and logical addressing (e.g., IP).
4. **Transport Layer**: Ensures reliable data transfer with flow control, error detection, and correction (e.g., TCP, UDP).
5. **Session Layer**: Manages sessions or connections between applications.
6. **Presentation Layer**: Translates data formats between the application and network layers (e.g., encryption, compression).
7. **Application Layer**: Interfaces with end-user applications (e.g., HTTP, FTP).

### TCP/IP Suite
The **TCP/IP Suite** (Transmission Control Protocol/Internet Protocol) is a set of networking protocols that define how data is transmitted over the internet. It has four layers, which roughly correspond to layers in the OSI model:
1. **Network Interface (Link) Layer**: Corresponds to the OSI's Physical and Data Link layers, responsible for hardware and data transmission.
2. **Internet Layer**: Maps to the Network Layer, handling logical addressing and routing (e.g., IP).
3. **Transport Layer**: Corresponds to the OSI's Transport Layer, ensuring reliable communication (e.g., TCP, UDP).
4. **Application Layer**: Encompasses the OSI's Session, Presentation, and Application layers, managing higher-level protocols (e.g., HTTP, SMTP).

### Connection between OSI and TCP/IP
While the OSI model is more of a theoretical framework, the TCP/IP suite is a practical implementation used in modern networking. TCP/IP's layers are more condensed compared to the OSI model. Both models serve to explain how data moves through a network, with OSI offering a broader concept and TCP/IP being the actual protocol stack used for internet communication.

---

## 3. **Assume Role Policy vs. Role Policy**

### Assume Role Policy
An **Assume Role Policy** is a policy that defines **who (which principal)** can assume (i.e., take on) a specific IAM (Identity and Access Management) role. It specifies which entities (users, applications, services) are allowed to assume the role. This is often used in cross-account scenarios or when delegating access between different AWS services. The assume role policy is attached to the **trusted entity** that grants the permissions to assume the role.

- Example:
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }
    ```

### Role Policy
A **Role Policy** (or inline policy) defines **what actions** the role itself is permitted to perform after it has been assumed. It specifies the permissions granted to the role when it interacts with AWS resources. The role policy is attached to the **IAM role** itself and controls what the role can do (e.g., read data from S3, launch EC2 instances).

- Example:
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:ListBucket",
          "Resource": "arn:aws:s3:::example-bucket"
        }
      ]
    }
    ```

### Key Differences
- **Assume Role Policy**: Specifies **who** can assume the role.
- **Role Policy**: Specifies **what actions** the role can perform once assumed.

---