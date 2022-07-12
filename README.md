# OSD_CRC_Enabling

## forked URL

https://github.com/epeenabling/oceanbase_crc_opt/

## PR
https://github.com/oceanbase/oceanbase/pull/985

## Intel ISA-L lib

https://github.com/intel/isa-l/tree/master/crc

## Oceanbase CRC code

### Why need to optimizate the Oceanbase CRC 

![image](https://user-images.githubusercontent.com/3771594/165459875-e93957e5-a294-4c2a-8da6-f35e24aae796.png)

This picture shows that Oceanbase will do CRC checksum while running Sysbench OLTP Read write senario. The time to trigger the CRC checksum is that Oceanbase compact the table. But one thing should be noticed is that the compation time is very short during Sysbench OLTP Read-Write test.

### The code need to be changed

https://github.com/oceanbase/oceanbase/blob/master/deps/oblib/src/lib/checksum/ob_crc64.cpp

The Oceanbase download ISAL lib dynamicaly on deps/3rd/pkg/devdeps-isa-l-static-2.22.0-3.el8.x86_64.rpm. After unpackage the rpm. The ISAL lib is located on deps/3rd/usr/local/oceanbase/deps/devel/lib/libisal.a and the ISAL header file are install on deps/3rd/usr/local/oceanbase/deps/devel/include/isa-l/. All the header files are:

![image](https://user-images.githubusercontent.com/3771594/165451093-6662a3b7-397d-447d-b52c-ba1421db28a3.png)

Since the Oceanbase itself include the ISAL header files and lib, we do not need to use the main stream ISAL code. Just including the crc64.h head file is fine. <br>

The ob_crc64.cpp file calling the crc by following code:

![image](https://user-images.githubusercontent.com/3771594/165454812-f0af6f73-6f1e-4f75-97eb-a3b3a9460956.png)

So after include the crc64.h header file, we can replace current crc function with the ISAL crc function as following code:

![image](https://user-images.githubusercontent.com/3771594/165455159-f084ff49-0c67-40ec-b7a6-a085b48d923d.png)

### Testing code
In order to compare the performance before and after changing code. The test code is as below:

![image](https://user-images.githubusercontent.com/3771594/165455966-f370bd4f-1f92-490d-b160-6cb42a2bbf7d.png)

The performance result is :

![image](https://user-images.githubusercontent.com/3771594/165456574-ba685233-2e89-49f5-b9b5-b74f85d23c9a.png)

#Oceanbase Configuration

### Major compaction 

http://www.foox.io/docs-cn/5.overview/6.storage-architecture/5.dump-and-merge/

### freeze_trigger_percentage
https://open.oceanbase.com/docs/observer-cn/V3.1.1/10000000000013481

* alter system set freeze_trigger_percentage=75

freeze_trigger_percentage 用于设置触发全局冻结的租户使用内存阈值。

属性	描述
参数类型	整型
默认值	50
取值范围	[1, 99]
是否重启 OBServer 生效	否
如果开启了自动触发全局冻结，则当租户使用的内存达到该阈值时，会触发全局冻结。自动触发全局冻结开关的设置请参见 enable_global_freeze_trigger。

### sysbench configuration

https://stackoverflow.com/questions/59152355/how-to-change-the-read-write-ratio-of-sysbench-oltp-read-write-script

#### 40% write
/home/zhr/sysbench-1.0.20/src/;sysbench ./lua/oltp_read_write.lua --delete_inserts=4  --mysql-host=127.0.0.1 --mysql-port=3881 --mysql-db=test --mysql-user=root@sysbench_tenant --table_size=1000000 --tables=30 --threads=1000 --report-interval=5 --time=3600 run 

/home/zhr/sysbench-1.0.20/src/;sysbench ./lua/oltp_read_write.lua --index_updates=8  --mysql-host=127.0.0.1 --mysql-port=3881 --mysql-db=test --mysql-user=root@sysbench_tenant --table_size=1000000 --tables=30 --threads=1000 --report-interval=5 --time=3600 run 

