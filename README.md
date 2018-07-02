### HDFSWrapper
HDFS C Objects to directly include into libraries with fix to HDFS-13585. Initial source files are from https://github.com/apache/hadoop/tree/branch-2.7.5/hadoop-hdfs-project/hadoop-hdfs/src.

Changes were made to the following source files -
* main/native/libhdfs/os/posix/thread_local_storage.c
* main/native/libhdfs/hdfs.c

The cmake build files have been re-written to just generate object files to be included directly into other libraries.


