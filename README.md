# SOC Design For Testing Digital IP chips (RISC V and SRAM)
This Project explains the importance of components used in the PCB design and also how data flows within the design.

# Table of Contents
- [Brief Introduction to Testing flow]()
- [Introduction to FT2232H]()
- [Data transfer from USB port to FT2232H]()
- [Data transfer from FT2232H to SPI flash memory]()
- [Future work]()
- [Acknowledgements]()

# Brief Introduction to Testing flow
An IDE ( Integrated Development Environment) is used for programming the PCB design. For an IDE to recognize the hardware board, a Board Support Package(BSP) which includes hardware description and linker files for memory mapping are used and after programming, the hex files are generated. The resultant bitstream is passed to the following PCB design.

![pcb design](https://user-images.githubusercontent.com/74853558/99901876-d6eb3500-2cdf-11eb-98e2-f7399bf56f47.jpg)

# Introduction to FT2232H
The serial port of the computer usually meets RS-232 standard. It has signal voltage swing of around -13 to +13V. The PCB design uses TTL serial (Transistor-Transistor logic) which has signal voltage level between 0 and VDD ( 3.3V/1.8V). A common communication protocol is needed for data transfer, hence FT2232H is used for converting USB to serial converter.
  
  For the data transfer from host to the target (board), we connect USB cable ( USB type A to the PC and Type B to the board).  USB has 4 signals, 5V power supply signal, ground and 2 data signals(D+ and D-). 

![1](https://user-images.githubusercontent.com/74853558/99901951-50832300-2ce0-11eb-83eb-0b9878771b1e.jpg)

The data is transferred from these two data lines using NRZI encoding.

![2](https://user-images.githubusercontent.com/74853558/99904990-128ffa00-2cf4-11eb-8bde-f9693f8e3624.jpg)

# Data transfer from USB port to FT2232H
According to USB communication protocol, data is sent through packets. It consists of 3 common packets.

![3](https://user-images.githubusercontent.com/74853558/99904992-13c12700-2cf4-11eb-96e3-813c6a7b75de.jpg) 

Data field can carry varying size of data depending upon the speed at which data is transferred.
- For low speed devices, data can be upto 8B
- For full speed devices, data can be upto 1023B
- For high speed devices, data can be upto 1024B

![4](https://user-images.githubusercontent.com/74853558/99904993-1459bd80-2cf4-11eb-8905-544269d66b9f.jpg)

Basically, the data transaction can be done in 4 types:
- Control: It is generally used by host for sending commands and parameters.
- Interrupt: It is used by devices for sending small amounts of data. It is a polled message from the host to request specific data of the device
- Bulk: this is used by devices for sending large amounts of data. 
- Isochronous: this is generally used by devices like audio channels etc, for streaming real time data. 
FTDI supports bulk data transaction. Data is sent through endpoint buffers within the device. FT2232H can have a maximum packet size of 512B.

![5](https://user-images.githubusercontent.com/74853558/99904994-14f25400-2cf4-11eb-9d85-3402415096fe.jpg)

The following example code is considered to demonstrate how the data flows from one block to other. ( Image courtesy: [Shivanishah/riscv](https://github.com/shivanishah269/risc-v-core/blob/master/Images/main_ABI.png))

![14](https://user-images.githubusercontent.com/74853558/99946186-f3e53e00-2d9b-11eb-94d2-cc161b408bdc.jpg)

The example code has a bitstream of 44bytes. This is sent from host to device through data packet in data field.
![6](https://user-images.githubusercontent.com/74853558/99905235-a57d6400-2cf5-11eb-9f23-9140bd84b50f.jpg)

The data transfer from device to the host is done when host is ready to accept data, it sends token packet. The device then sends data.
![7](https://user-images.githubusercontent.com/74853558/99905237-a7472780-2cf5-11eb-8387-13a4a12ac96b.jpg)

# Data transfer from FT2232H to SPI flash memory

FT2232H has multiple configurations like synchronous bitbang interface, asynchronous bitbang interface and MPSSE (Multi-Protocol Synchronous Serial Engine) interface. MPSSE mode allows communication with different types of synchronous devices like SPI, I2C, JTAG etc.

![8](https://user-images.githubusercontent.com/74853558/99904997-158aea80-2cf4-11eb-925a-c565d1dcbe6b.jpg)

SPI communication has 4 modes for data transfer depending on clock polarity (CPOL) and clock phase (CPHA).  FTDI device can only support mode 0 and mode 2 whereas flash memory (S25FL128S) has only 2 clocking modes i.e mode 0 and mode 3.

![9](https://user-images.githubusercontent.com/74853558/99904999-16238100-2cf4-11eb-9220-c5b521bf7785.jpg)
The data transfer between FT2232H and S25FL128S is done in the form of units called commands. These commands consist of instructions which tells the device operation and may also have address, latency period followed by data. The input data into the device is always latched in at the rising edge of the SCK signal and the output data is always available from the falling edge of the SCK clock signal.

![10](https://user-images.githubusercontent.com/74853558/99905000-16bc1780-2cf4-11eb-9f24-1c89a508696d.jpg)

![11](https://user-images.githubusercontent.com/74853558/99905001-16bc1780-2cf4-11eb-96d1-caabca6888cc.jpg)

The memory is divided into sectors either a combination of 64KB and 4KB or a size of 256KB. For a sector size of 256KB, there can be total of 64 sectors for the memory and each sector is again divided into pages. Each page size can be a maximum of 512B. 

![12](https://user-images.githubusercontent.com/74853558/99905002-1754ae00-2cf4-11eb-8c51-300a52c790a7.jpg)

The data is transferred in byte units and after each byte transmission the address byte is incremented automatically. For writing the data, the maximum data that can be written in one operation is upto 512B. For more data transfer, again instruction followed by address is sent.

![13](https://user-images.githubusercontent.com/74853558/99905003-1754ae00-2cf4-11eb-8f00-3c057b3d4974.jpg)
