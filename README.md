# OSD_CRC_Enabling

## Intel ISA-L lib

https://github.com/intel/isa-l/tree/master/crc

## Oceanbase CRC code

https://github.com/oceanbase/oceanbase/blob/master/deps/oblib/src/lib/checksum/ob_crc64.cpp

The Oceanbase download ISAL lib dynamicaly on deps/3rd/pkg/devdeps-isa-l-static-2.22.0-3.el8.x86_64.rpm. After unpackage the rpm. The ISAL lib is located on deps/3rd/usr/local/oceanbase/deps/devel/lib/libisal.a and the ISAL header file are install on deps/3rd/usr/local/oceanbase/deps/devel/include/isa-l/. All the header files are:

![image](https://user-images.githubusercontent.com/3771594/165451093-6662a3b7-397d-447d-b52c-ba1421db28a3.png)

Since the Oceanbase itself include the ISAL header files and lib, we do not need to use the main stream ISAL code. Just including the crc64.h head file is fine. <br>

The ob_crc64.cpp file calling the crc by following code:

![image](https://user-images.githubusercontent.com/3771594/165454812-f0af6f73-6f1e-4f75-97eb-a3b3a9460956.png)

So after include the crc64.h header file, we can replace current crc function with the ISAL crc function as following code:

![image](https://user-images.githubusercontent.com/3771594/165455159-f084ff49-0c67-40ec-b7a6-a085b48d923d.png)
