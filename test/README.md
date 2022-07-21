# Test Project

## Check

```shell
export INTERLOK_PARENT_VERSION=v4 #v4 is default
gradle clean check
```

## Install

```shell
export INTERLOK_PARENT_VERSION=v4 #v4 is default
gradle clean install
cd ./build/distribution
VALUE_IN_ENV=value3 java -DvalueSystemProperty=value4 -jar ./lib/interlok-boot.jar
curl http://localhost:8090/hello
```
