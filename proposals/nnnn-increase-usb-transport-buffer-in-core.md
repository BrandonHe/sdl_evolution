# Increase USB Transport Buffer in Core

* Proposal: [SDL-NNNN](nnnn-increase-usb-transport-buffer-in-core.md)
* Author: [BrandonHe](https://github.com/brandonhe)
* Status: **Awaiting review**
* Impacted Platforms: [Core]

## Introduction

This proposal aims to achieve better video streaming peformance for navigation and projection by increaseing the in endpoint USB transport buffer in SDL Core. 

## Motivation

Currently, the SDL recieve video streaming data from libusb in usbConnection depends on USB endpoint device max packet size, the packets size are 64 bytes in USB full-speed accessories, and 512 bytes in USB high-speed accessories. Therefore, the big video streaming frame data are have to be splited into multiple 512 bytes packets to transmit to in_buffer_, the later process in up level transmission are also have to be splited by multiple packeges, at last, these splited data should assemble to a whole frame data in Protocol layer. It takes much of asynchronize read/write operation, occupies much of CPU process time. The performance of video streaming is decreased by a mass of desemble and assemble frame work in long process video streaming. In our testing, the latency can reach to 1000ms.

## Proposed solution

### Overall Sequence
The SDL Core is using open source libusb libraries to process low level system USB data transmission, the data flow is as below:

1. USBTransport(Proxy) --> libusb -->
2. UsbConnection::OnInTransfer --> TransportManagerImpl::PostEvent -->
3. MessageLoopThread::PostMessage --> MessageLoopThread::Handler -->
4. TransportManagerImpl::Handler --> ProtocolHandlerImpl::OnTMMessageReceived -->
5. MessageLoopThread::PostMessage --> MessageLoopThread::Handler -->
6. ProtocolHandlerImpl::Handler --> StreamAdapter::SendData -->
7. StreamerAdapter::Streamer::threadMain --> PipeStreamerAdapter::PipeStreamer::Send -->
8. Gstreamer-->Display. 

There have many synchronize/asynchronize thread operationx, also many read/write operations in video streaming data transmission. This is a very long flow to handle video streaming data. When in buffer receives a packet size of raw data, SDL sends it to TransportManager handle, pack the raw message by transport to up level process, and then the whole frame data are packed in SDL protocol, then send to Media Manager.

This proposal suggests increase the dynamic allocation in buffer size from endpoint device discriptor variable `in_endpoint_max_packet_size` to 16KBytes. When SDL read data from an endpoints object, should ensure that SDL create a big enough new buffer to store the USB packet data. The Android accessory protocol supports packet buffers up to 16384 bytes, in document [developer.android.com](https://developer.android.com/guide/topics/connectivity/usb/accessory.html), it suggests we can choose to always declare our buffer to be of this size. For iOS video streaming data via USB connection, the buffer size is defined in kInBufferSize.

The proposal originally combined with [proposal-0069](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0069-enhance-video-streaming-performance-for-android.md) to improve the video streaming performance, this proposal was rejected, for an accepted proposal analysis can refer [proposal-0100](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0100-new-data-transfer-interface-for-streaming-android.md). This has much video streaming performance improvement both from proxy and core. We tested more than 4 hours, the video streaming latency keep **less than 300ms** in 4Mbps bit rate and 30fs fame rate.


### Detailed design

Add additional variable `in_buffer_size_` in usb_connetion.h:
```c++
class UsbConnection : public Connection {
  public:
  protected:
  private:
    unsigned char* in_buffer_;
    uint16_t in_buffer_size_;
    libusb_transfer* in_transfer_;

```
The variable is used in usb_connection.cc:

```c++
#define TRANSPORT_USB_BUFFER_MAX_SIZE	(16*1024)

  if(in_endpoint_max_packet_size_ < TRANSPORT_USB_BUFFER_MAX_SIZE){
  	in_buffer_size_ = TRANSPORT_USB_BUFFER_MAX_SIZE;
  }
  else {
  	in_buffer_size_ = in_endpoint_max_packet_size_;
  }

  in_buffer_ = new unsigned char[in_buffer_size_];
```
### Verification Test
In the verifification test we did, below tables are the result:

- sdl_core: [version 4.4.0](https://github.com/smartdevicelink/sdl_core/release/tag/4.4.0)
- sdl_android: [version 4.4.0](https://github.com/smartdevicelink/sdl_android/release/tag/4.4.0)
- OS: VirtualBox Ubuntu 16.04 x64
- Gstreamer: 1.0
- Decoding solution: software decoding
- Resolution: 800x480, 30fps
- Android Device: Samsung Note5(Android 6.0)

#### Not include propsal testing:
| in_buffer_ size | Bit Rate | Frame Rate | Video Latency(ms) |
|-----------------|----------|:------------------:|---------------------------|
| 512 bytes       |  1Mpbs   |  avg:30 min:22 max:33  | avg:254 min:213 max:269 |
| 512 bytes       |  2Mpbs   |  avg:30 min:22 max:32  | avg:289 min:266 max:374 |
| 512 bytes       |  4Mpbs   |  avg:15 min:14 max:18  | avg:630 min:561 max:699 |
| 512 bytes       |  6Mpbs   |  avg:10 min:9 max:13  | avg:889 min:774 max:978 |

#### Include proposal testing:

| in_buffer_ size | Bit Rate | Frame Rate | Video Latency(ms) |
|-----------------|----------|:------------------:|---------------------------|
| 16K bytes       |  1Mpbs   |  avg:30 min:21 max:33  | avg:252 min:227 max:286 |
| 16K bytes       |  2Mpbs   |  avg:30 min:22 max:32  | avg:254 min:217 max:282 |
| 16K bytes       |  4Mpbs   |  avg:30 min:22 max:33  | avg:246 min:220 max:293 |
| 16K bytes       |  6Mpbs   |  avg:30 min:22 max:33  | avg:240 min:215 max:282 |

After 4 hours testing:

| in_buffer_ size | Bit Rate | Frame Rate | Video Latency(ms) |
|-----------------|----------|:------------------:|---------------------------|
| 16K bytes       |  4Mpbs   |  avg:30 min:22 max:32  | avg:275 min:221 max:362 |

## Potential downsides

This requires system allocate in_buffer heap size from 512 bytes up to 16 Kbytes.

## Impact on existing code

The impact on existing code should be small since only the `in_buffer_` variable size is updated.

## Alternatives considered

The alernative solution is to reconstruct the video streaming data flow, deprecate data flow from each layer handler, create a `startStream` function in UsbConnection and directly write data to Pipe, and then Gstreamer directly decode data from this Pipe. All frame control data will be processed in another function, such as frameType, data `serviceType` is `KAudio` or `kMobileNav`.