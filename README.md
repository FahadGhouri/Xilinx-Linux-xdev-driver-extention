# Xilinx Zynq xdevcfg Linux device driver

Xilinx by default provides the xdevcfg interface Zynq driver with Petalinux distribution. The successor to this driver interface is the FPGA - Manager in later Petalinux versions. The interface was primarily written to program the FPGA over the PCAP interface through Linux running on Zynq. It also uses the internal AES/HMAC decryption block to program the FPGA with an encrypted bit stream. What it does not do by default is to let the user use this decryption block for anything else other than for FPGA bit streams.

This version of the driver extends xdevcfg interface to let the user use the AES/HMAC decryption block for any data block requested by the user. Implicitly, the data block needs to be encrypted as Zynq specifies. 


## /sys

The driver uses the Linux sysfs interface to control multiple switches inside the driver. This version extends the interface to add two additional control switches:

- Enable Loopback
- Reboot Flag

## /dev

The driver registers this interface as a device under the name "xdevcfg" and is available for reading from or writing to the FPGA (bit stream). This extension disables the read operation to this interface. That means bit stream can only be written to the FPGA but not read.

Additionally, now the write operation can also take any encrypted data block, pass it through the Zynq decryption block and return the decrypted data to the writer. This functionality is controlled from the new control switch (Enable Loopback) added in the linux /sysfs.

## How to use

Programming the FPGA is the same as the default driver. The bit stream needs to be written to the /dev interface. The Encrypted switch needs to be enabled for an encrypted bit stream. Zynq needs to be started in Secure mode for having this functionality. In Non-Secure boot, the AES/HMAC interface is permanently disabled.

For decrypting any other data block and returning the decrypted data, we need to enable the 'Enable Loopback' flag in /sys. The encrypted data switch needs to be enabled too. 
The first 4 bytes of the encrypted data written to the interface needs to be the byte length of the expected decrypted data length. The length parameter is of 4 Bytes (1 Word) and needs to be arranged in Little Endian format.

   | Decrypted Length (0)| Decrypted Length (1) | Decrypted Length (2) | Decrypted Length (3) | Encrypted Data Byte (0) | Encrypted Data Byte (1) | ...... | Encrypted Data Byte (n) |
   
For more details on how Zynq hardware AES/HMAC block works please refer to the Technical Support Manual.
