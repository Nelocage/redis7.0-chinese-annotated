cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(redis VERSION 6.0)

set(CMAKE_BUILD_TYPE "Debug")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
            "The type of build. Possible values are: Debug, Release, RelWithDebInfo and MinSizeRel.")
endif()

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

# Just for debugging when handling a new platform.
if (false)
    message("C++ compiler supports these language features:")
    foreach(i ${CMAKE_CXX_COMPILE_FEATURES})
        message("  ${i}")
    endforeach()
endif()

#zai src目录生成release.h 比较重要
message(STATUS "Generating release.h...")
execute_process(
        COMMAND sh -c ./mkreleasehdr.sh
        WORKING_DIRECTORY ${REDIS_ROOT}/src/
)

add_subdirectory(deps)
set(SRC_SERVER_TMP
        src/acl.c
        src/adlist.c
        src/ae.c
        #  src/ae_kqueue.c
        src/anet.c
        src/aof.c
        src/bio.c
        src/bitops.c
        src/blocked.c
        src/call_reply.c
        src/childinfo.c
        src/cluster.c
        src/config.c
        src/connection.c
        src/commands.c
        src/crc16.c
        src/crc64.c
        src/db.c
        src/debug.c
        src/defrag.c
        src/dict.c
        src/endianconv.c
        src/evict.c
        src/expire.c
        src/eval.c
        src/functions.c
        src/function_lua.c
        src/geo.c
        src/geohash.c
        src/geohash_helper.c
        src/hyperloglog.c
        src/intset.c
        src/latency.c
        src/lazyfree.c
        src/listpack.c
        src/localtime.c
        src/lolwut.c
        src/lolwut5.c
        src/lolwut6.c
        src/lzf_c.c
        src/lzf_d.c
        src/memtest.c
        src/module.c
        src/multi.c
        src/monotonic.c
        src/mt19937-64.c
        src/networking.c
        src/notify.c
        src/object.c
        src/pqsort.c
        src/pubsub.c
        src/quicklist.c
        src/rand.c
        src/rax.c
        src/rdb.c
        src/redis-check-aof.c
        src/redis-check-rdb.c
        src/release.c
        src/replication.c
        src/rio.c
        src/script.c
        src/script_lua.c
        src/sds.c
        src/sentinel.c
        src/setproctitle.c
        src/setcpuaffinity.c
        src/sha1.c
        src/sha256.c
        src/siphash.c
        src/slowlog.c
        src/sort.c
        src/sparkline.c
        src/syncio.c
        src/syscheck.c
        src/t_hash.c
        src/t_list.c
        src/t_set.c
        src/t_stream.c
        src/t_string.c
        src/t_zset.c
        src/tls.c
        src/tracking.c
        src/util.c
        src/ziplist.c
        src/zipmap.c
        src/zmalloc.c
        src/timeout.c
        src/crcspeed.c
        src/resp_parser.c
        )
set(SRC_SERVER src/server.c ${SRC_SERVER_TMP})


set(SRC_CLI
        src/ae.c
        src/anet.c
        src/sds.c
        src/adlist.c
        src/redis-cli.c
        src/zmalloc.c
        src/release.c
        src/monotonic.c
        src/anet.c
        src/ae.c
        src/crc64.c
        src/crcspeed.c
        src/dict.c
        # src/debug.c
        src/crc16.c
        src/cli_common.c
        # src/t_list.c
        src/mt19937-64.c
        src/siphash.c
        src/redisassert.c
#        src/object.c
#        src/networking.c
#        src/server.c
        )

#这个指令不需要，不用自己制定可执行文件的输出位置，clion自己有 cmake-build-debug
#set(EXECUTABLE_OUTPUT_PATH src)
#库所在目录
link_directories(deps/linenoise/ deps/lua/src deps/hiredis )
include_directories(deps/hdr_histogram)
# redis-server
add_executable(redis-server ${SRC_SERVER})
target_include_directories(redis-server
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        PRIVATE ${REDIS_ROOT}/deps/hdr_histogram)
target_link_libraries(redis-server
        PRIVATE pthread
        PRIVATE m
        PRIVATE lua
        PRIVATE linenoise
        PRIVATE hdr_histogram
        PRIVATE hiredis
        #这一步,是因为出现dlopen找不到等报错,dlopen,dlclose等是系统调用动态库的函数
        ${CMAKE_DL_LIBS}
        )

# redis-cli
add_executable(redis-cli ${SRC_CLI})
target_include_directories(redis-cli
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/lua/src)

target_link_libraries(redis-cli
        PRIVATE pthread
        PRIVATE m
        PRIVATE linenoise
        PRIVATE hiredis
        )

