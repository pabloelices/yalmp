# YALMP

YALMP (Yet Another Lightweight Messaging Protocol) is a messaging protocol designed to send telemetry data from a drone to a ground control station with minimal overhead.

**Note**: This repository if part of my final year project and is by no means production ready.

## 1. Features

- Up to 256 message topics.
- Up to 255 bytes of payload per packet.
- 4 bytes of overhead per packet.
- Integrity verification of packets via checksum.
- No packet drop detection.
- No packet authentication.

## 2. Packet structure

| Field | Index (Bytes) | Description |
|--:|:-:|:--|
| Start Flag (SF) | 0 | Denotes the start of a new packet. It has a fixed value of 0x4E.
| Message ID (MI) | 1 | Defines how the payload should be processed.
| Payload Length (PL) | 2 | The payload's length in bytes.
| Payload (P) | 3 to 3 + PL - 1 if PL > 0 | The payload itself.
| Checksum (C) | 3 + PL | The packet's checksum.

## 3. Checksum computation

Checksums are computed adding the packet's fields (excluding the SF) modulo 256 and then applying a bitwise not. C++ / Qt example:  

```c++
quint8 Checksum(quint8 messageId, const QByteArray& payload)
{
  quint8 checksum {messageId};

  checksum += static_cast<quint8>(payload.count());

  for (int i = 0; i < payload.count(); i++)
  {
    checksum += static_cast<quint8>(payload.at(i));
  }

  checksum = ~checksum;

  return checksum;
}
```

## 4. Examples

### 4.1. Example 1 

To send an empty message with *MI = 0x00* send the following packet:

| SF | MI | PL | C |
|:-:|:-:|:-:|:-:|
| 0x4E | 0x00 | 0x00 | 0xFF |

### 4.2. Example 2

To send the message "Hello" with *MI = 0x7F* send the following packet:

| SF | MI | PL | P<sub>1</sub> | P<sub>2</sub> | P<sub>3</sub> | P<sub>4</sub> | P<sub>5</sub> | C |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 0x4E | 0x7F | 0x05 | 0x48 | 0x65 | 0x6C | 0x6C | 0x6F | 0x87 |
|||| H | e | l | l | o ||

### 4.3. Example 3

To send the message "0.1.0" with *MI = 0xFF* send the following packet:

| SF | MI | PL | P<sub>1</sub> | P<sub>2</sub> | P<sub>3</sub> | P<sub>4</sub> | P<sub>5</sub> | C |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 0x4E | 0xFF | 0x05 | 0x30 | 0x2E | 0x31 | 0x2E | 0x30 | 0x0E |
|||| 0 | . | 1 | . | 0 ||

## 5. License

MIT License. See [LICENSE](LICENSE "LICENSE") for details.
