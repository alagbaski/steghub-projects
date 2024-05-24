# Side Self Study Documentation

## Network-attached storage (NAS) and Storage Area Network (SAN)

### Network-attached storage (NAS)
Network-attached storage (NAS) is a dedicated file storage device that provides local area network (LAN) users with centralized, consolidated disk storage through a standard Ethernet connection. NAS devices are configured with network protocols like NFS, SMB, and FTP.

#### Key Features of NAS:
- **File-level storage**: Stores and retrieves files from a dedicated storage device.
- **Network sharing**: Allows multiple users and client devices to access the files over a network.
- **Ease of setup and management**: Often simpler to set up compared to SAN.

### Storage Area Network (SAN)
A Storage Area Network (SAN) is a specialized, high-speed network that provides block-level storage. SANs connect servers to a pool of storage devices using protocols like iSCSI and Fibre Channel.

#### Key Features of SAN:
- **Block-level storage**: Provides raw storage that can be partitioned and formatted by the server.
- **High performance**: Designed for high-speed data transfer and efficient storage management.
- **Scalability and flexibility**: Can handle large volumes of data and scale easily.

## Related Protocols

### Network File System (NFS)
NFS is a distributed file system protocol allowing a user on a client computer to access files over a network in the same way they access local storage.

### Server Message Block (SMB)
SMB is a network file sharing protocol that allows applications to read and write to files and request services from server programs in a computer network.

### File Transfer Protocol (FTP) and Secure FTP (sFTP)
FTP is a standard network protocol used to transfer files from one host to another over a TCP-based network. sFTP is a secure version of FTP that uses SSH to encrypt data transfers.

### Internet Small Computer Systems Interface (iSCSI)
iSCSI is a transport layer protocol that works on top of the TCP/IP protocol, enabling the creation of storage area networks (SANs) by linking data storage facilities.

## Block-level Storage
Block-level storage refers to a type of storage typically used in SAN environments where data is stored in volumes, or blocks, and each block acts as an individual hard drive. This provides the flexibility to configure and manage storage as needed, allowing for high performance and low latency.

## AWS Services Examples

### Block Storage
AWS provides block storage through its service called Amazon Elastic Block Store (EBS). EBS volumes act like raw, unformatted block devices and can be attached to EC2 instances, offering low-latency access to data.

### Object Storage
Amazon Simple Storage Service (S3) is AWS's object storage service. It stores data as objects within resources called "buckets". S3 is designed for scalability, data availability, security, and performance.

### Network File System
Amazon Elastic File System (EFS) is AWS's managed NFS service. It provides scalable file storage for use with AWS Cloud services and on-premises resources.

## Differences Between Block Storage, Object Storage, and Network File System

- **Block Storage**: 
  - Uses raw storage volumes (blocks).
  - Ideal for applications requiring low latency and high performance.
  - Examples: Amazon EBS.

- **Object Storage**: 
  - Stores data as objects in buckets.
  - Scalable and suitable for unstructured data.
  - Examples: Amazon S3.

- **Network File System**:
  - Manages shared file storage.
  - Suitable for workloads that require shared file access.
  - Examples: Amazon EFS.

## Conclusion
Understanding the differences between NAS, SAN, and related protocols, as well as the various types of storage provided by services like AWS, is crucial for designing and managing efficient and scalable storage solutions. Block-level storage offers flexibility and high performance, while object storage is ideal for scalability and managing large datasets. Network file systems provide shared file access, making them suitable for collaborative environments.
