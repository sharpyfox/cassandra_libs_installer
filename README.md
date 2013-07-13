#cassandra libs installer#

Cassandra thrift libraries builder and installer. It download cassandra thrift interface file, generate source files with thrift and build them.

##How to build##

The build-chain for this library is CMake.

```
git clone https://github.com/sharpyfox/cassandra_libs_installer.git
cd cassandra_libs_installer
cmake . && make && make install
```

You can pinch cassandra interface version by `CASSANDRA_VERSION` build flag. Default version is `1.1.9`

##How to uninstall##

```
make uninstall
```

###Enjoy!###
