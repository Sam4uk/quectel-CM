cmake_minimum_required(VERSION 3.11)

project(quectel)

add_compile_options(
    -Wall 
    -Werror 
    -O1 
    #-s
)

link_libraries(
    # pthread
    # dl
    # rt
)

set(QL_CM_SRC
    QmiWwanCM.c GobiNetCM.c main.c MPQMUX.c QMIThread.c util.c qmap_bridge_mode.c mbim-cm.c device.c
    atc.c atchannel.c at_tok.c
)

if(1)
    set(QL_CM_DHCP udhcpc.c)
else()
endif()


add_executable(${PROJECT_NAME}-qmi-proxy quectel-qmi-proxy.c)
add_executable(${PROJECT_NAME}-mbim-proxy quectel-mbim-proxy.c)
add_executable(${PROJECT_NAME}-CM ${QL_CM_SRC} ${QL_CM_DHCP})

target_link_libraries(${PROJECT_NAME}-CM
    PRIVATE
    pthread
    dl
    rt
)

# ifneq ($(CROSS_COMPILE),)
# CROSS-COMPILE:=$(CROSS_COMPILE)
# endif
# #CROSS-COMPILE:=/workspace/buildroot/buildroot-qemu_mips_malta_defconfig/output/host/usr/bin/mips-buildroot-linux-uclibc-
# #CROSS-COMPILE:=/workspace/buildroot/buildroot-qemu_arm_vexpress_defconfig/output/host/usr/bin/arm-buildroot-linux-uclibcgnueabi-
# #CROSS-COMPILE:=/workspace/buildroot-git/qemu_mips64_malta/output/host/usr/bin/mips-gnu-linux-
# ifeq ($(CC),cc)
# CC:=$(CROSS-COMPILE)gcc
# endif
# LD:=$(CROSS-COMPILE)ld

# QL_CM_SRC=QmiWwanCM.c GobiNetCM.c main.c MPQMUX.c QMIThread.c util.c qmap_bridge_mode.c mbim-cm.c device.c
# QL_CM_SRC+=atc.c atchannel.c at_tok.c
# #QL_CM_SRC+=qrtr.c rmnetctl.c
# ifeq (1,1)
# QL_CM_DHCP=udhcpc.c
# else
# LIBMNL=libmnl/ifutils.c libmnl/attr.c libmnl/callback.c libmnl/nlmsg.c libmnl/socket.c
# DHCP=libmnl/dhcp/dhcpclient.c libmnl/dhcp/dhcpmsg.c libmnl/dhcp/packet.c
# QL_CM_DHCP=udhcpc_netlink.c
# QL_CM_DHCP+=${LIBMNL}
# endif

# CFLAGS += -Wall -Werror -O1 #-s
# LDFLAGS += -lpthread -ldl -lrt

# release: clean qmi-proxy mbim-proxy
# 	$(CC) ${CFLAGS} ${QL_CM_SRC} ${QL_CM_DHCP} -o quectel-CM ${LDFLAGS}

# debug: clean
# 	$(CC) ${CFLAGS} -g -DCM_DEBUG ${QL_CM_SRC} ${QL_CM_DHCP} -o quectel-CM -lpthread -ldl -lrt

# qmi-proxy:
# 	$(CC) ${CFLAGS} quectel-qmi-proxy.c -o quectel-qmi-proxy ${LDFLAGS} 

# mbim-proxy:
# 	$(CC) ${CFLAGS} quectel-mbim-proxy.c -o quectel-mbim-proxy ${LDFLAGS} 

# clean:
# 	rm -rf *.o libmnl/*.o quectel-CM quectel-qmi-proxy quectel-mbim-proxy

file(WRITE ${PROJECT_BINARY_DIR}/.gitignore
    [=====[
# CMAKE generated file: DO NOT EDIT!
*
*.*
]=====]
)
