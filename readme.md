# Live Link Face iPhone App Deserialization

See `livelink.py` for a working deserializer for [Unreal Engine's Live Link App](https://apps.apple.com/us/app/live-link-face/id1495370836). [Documentation can be found here](https://docs.unrealengine.com/4.27/en-US/AnimatingObjects/SkeletalMeshAnimation/FacialRecordingiPhone/).

Apple's Face AR Kit uses Deep Learning to read your face and spit out 61 float values. Unreal Engine's Live Link Face App then packages those 61 float values, and dumps them to a UDP port. We can listen and record these bytes with netcat to create the livelink.udp file found in this repo. The command is

```bash
nc -ul 11111 > livelink.udp 2>&1
```

Fortunately for us, the Unreal Engine is open-source. So we can just peek the AppleARKitLiveLinkSource.cpp file to reverse the serialization format the Unreal Engine uses.

## Usage

**READING FROM A SOCKET**

```python
import socket
UDP_IP = '0.0.0.0'
UDP_PORT = 11111

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind((UDP_IP, UDP_PORT))
while True:
    livelink_data, _ = sock.recvfrom(500)
    livelink = LivelinkBuffer(livelink_data).parse_livelink()
    print(livelink.face_blend_shapes_values)  # numpy array
    # Print a specific blendshape value from [0, 1]
    print(livelink.face_blend_shapes_values[int(BlendShapes.MouthRight)])
```

**READING FROM A FILE**

```bash
nc -ul 11111 > livelink.udp 2>&1
```

```python
with open('./livelink.udp', 'rb') as f:
    livelink_buffer = LivelinkBuffer(f.read())

while livelink.offset < len(livelink.buffer):
    livelink = livelink_buffer.parse_livelink()
    print(livelink.face_blend_shapes_values)  # numpy array
    # Print a specific blendshape value from [0, 1]
    print(livelink.face_blend_shapes_values[int(BlendShapes.MouthRight)])
```
