# image_encoder.py
# Summary
Hide an image(or a text) into an image, and decode it with Python.

# Arrangement
### Modules
I am going to use **numpy** and **Image from PIL**
```python
from PIL import Image
import numpy as np
```


### Binary to Base10
This is a function converts a binary string to base10.
```python
def binary_to_base10(binary):
    bin_size = len(binary)
    demi = 0
    power = 0
    for i in reversed(range(bin_size)):
        if (binary[i] == "1"):
            demi = demi + pow(2, power)
        power = power + 1
    return demi
```


### Convert an image to a binary string
Convert a 3D vector stores RGB values to a binary string.
```python
def RGB_to_binary(imgArray):
    ret = ""
    for x in range(imgArray.shape[0]):
        for y in range(imgArray.shape[1]):
            for z in range(3):
                rgb = imgArray[x][y][z]
                ret = ret + str(f'{rgb:08b}')
    return ret
```


### Reformat a size of the binary string
Adjust a size of the binary string to integer-bytes.
```python
def reformat(binary):
    size = len(binary)
    if size % 8 != 0:
        size = 8 * (size // 8 + 1)
    ret = binary
    count = 0
    while len(ret) + count < size:
        ret = "0" + ret
    return ret
```

### Extract an image
In this code, if Nth bits of hidden binary is 1, turn Nth RGB value to odd, otherwise, turn it to even, and we can reverse these steps when decodeing it.
```python
def img_decode(imgArray):
    ret = ""
    for x in range(imgArray.shape[0]):
        for y in range(imgArray.shape[1]):
            for z in range(3):
                rgb = imgArray[x][y][z]
                if rgb % 2 == 0:
                    ret = ret + "0"
                else:
                    ret = ret + "1"
    return ret
```

# Implement
### Input
Get vectors which store RGB values of each images, and get a binary string to hide.
```python
im1 = Image.open("ORIGINAL IMAGE")
originalArray = np.array(im1)
im2 = Image.open("HIDE IMAGE")
hideArray = np.array(im2)
hideBinary = RGB_to_binary(hideArray)
size = len(hideBinary)
count = 0
hideHeight = reformat(str(np.base_repr(hideArray.shape[0], base=2)))
hideHeightLen = reformat(str(np.base_repr(len(hideHeight) // 8, base=2)))
hideWidth = reformat(str(np.base_repr(hideArray.shape[1], base=2)))
hideWidthLen = reformat(str(np.base_repr(len(hideWidth) // 8, base=2)))
hideBinary = hideHeightLen + hideWidthLen + hideHeight + hideWidth + hideBinary
```
**hideBinary** is composed of following values.<br>
hideHeightLen, hideWidthLen：How long is Height, Width?(bytes)
hideHeight, hideWidth：Height, Width of an image to hide
RGB_to_binary(hideArray)：RGB values of the image

### Hide
```python
for x in range(originalArray.shape[0]):
    for y in range(originalArray.shape[1]):
        for rgb in range(3):
            if count < size:
                check = hideBinary[count]
                if check == "1":
                    if originalArray[x][y][rgb] % 2 == 0:
                        originalArray[x][y][rgb] = originalArray[x][y][rgb]+1
                else:
                    if originalArray[x][y][rgb] % 2 == 1:
                        originalArray[x][y][rgb] = originalArray[x][y][rgb]-1
            count = count + 1
```
If Nth bits of hidden binary is 1, turn Nth RGB value to odd, otherwise, turn it to even.

### Save
```python
newImageArray = np.asarray(originalArray)
newImg = Image.fromarray(newImageArray)
newImg.save("encoded.png")
            count = count + 1
```

### Decode
```python
count = 0
encodedHeightLen = binary_to_base10(encodedBinary[0:8])
encodedWidthLen = binary_to_base10(encodedBinary[8:16])
index = encodedHeightLen * 8
encodedHeight = binary_to_base10(encodedBinary[16:16 + index])
encodedWidth = binary_to_base10(
    encodedBinary[16 + index:16 + index + encodedWidthLen * 8])
index = 16 + index + encodedWidthLen * 8
encodedIMG = encodedBinary[index:index + (encodedHeight * encodedWidth * 24)]
encodedRGB = []
for i in range(encodedHeight):
    ADD = []
    for j in range(encodedWidth):
        ADD.append([])
    encodedRGB.append(ADD)


Wcount = 0
Hcount = 0
for i in range(0, len(encodedIMG), 24):
    R = int(encodedIMG[i:i + 8], 2)
    G = int(encodedIMG[i + 8:i + 16], 2)
    B = int(encodedIMG[i + 16:i + 24], 2)
    ADD = [255 - R, 255 - G, 255 - B]
    encodedRGB[Hcount][Wcount] = ADD
    Wcount = Wcount + 1
    if Wcount >= encodedWidth:
        Wcount = 0
        Hcount = Hcount + 1


newImageArray = np.asarray(encodedRGB)
newImg = Image.fromarray((newImageArray * 255).astype(np.uint8))
newImg.save("decoded.png")
```

# Finally
If input images are large, it takes so long to run this code, so I want to make this faster.

