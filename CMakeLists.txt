cmake_minimum_required(VERSION 3.11)

project(quectel
        VERSION 1.6.2
        DESCRIPTION "Connectivity Management Tool for USB network adapter of Quectel wireless cellular modules."
        # HOMEPAGE_URL ""
        # LANGUAGES C
        )

set(GLIBC_VERION "2.28")

add_compile_options(
    -Wall 
    -Werror 
    -O1 
    #-s
)

# add_link_options(
#     # -Wl,--wrap
# )

link_libraries(
    pthread
    dl
    rt
)

# set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--version-script=glibc_version.map")

set(QL_CM_SRC
    QmiWwanCM.c GobiNetCM.c main.c MPQMUX.c QMIThread.c util.c qmap_bridge_mode.c mbim-cm.c device.c
    atc.c atchannel.c at_tok.c
)

if(1)
    set(QL_CM_DHCP udhcpc.c)
else()
endif()

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/glibc_version.map" "GLIBC_${GLIBC_VERION} {\n\t*;\n};")

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

# Задаємо список VID_PID (кожен елемент - пара VID PID)
set(VID_PID_LIST
    "2c7c 0125"  # Перший пристрій
    # "2c7c 0129"  # Другий пристрій
    # "2c7c 0121"  # Третій пристрій
)

set(INSTALL_BINDIR "/usr/local/bin/")

# Створюємо або очищуємо файл правил
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/99-${PROJECT_NAME}.rules" "# udev rules for ${PROJECT_NAME}\n")

# Обробляємо кожен елемент списку
foreach(item ${VID_PID_LIST})
    # Розбиваємо рядок на VID та PID (через пробіл)
    string(REPLACE " " ";" vid_pid_pair ${item})
    list(GET vid_pid_pair 0 VID)
    list(GET vid_pid_pair 1 PID)

    # Додаємо правило для поточного VID/PID
    file(APPEND "${CMAKE_CURRENT_BINARY_DIR}/99-${PROJECT_NAME}.rules"
        "\n# Device ${VID}:${PID}\n"
        "ACTION==\"add|change\", SUBSYSTEM==\"usb\", ATTR{idVendor}==\"${VID}\","
        " ATTR{idProduct}==\"${PID}\", RUN+=\"${INSTALL_BINDIR}${PROJECT_NAME}.sh\"\n"
    )
endforeach()

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.sh" "#!/bin/bash\n")

file(APPEND "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.sh"
    "# Цей скрипт запускає програму від імені root\n"
    "\n"
    "# Логування (для дебагу)\n"
    "echo \"[$(date)] Resetting EG25-G connected!\" >> /var/log/udev_root.log\n"
    "\n"
    "sleep 10"
    "\n"
    "# Запуск програми (шлях до бінарника)\n"
    "${INSTALL_BINDIR}${PROJECT_NAME}-CM >> /var/log/${PROJECT_NAME}-CM.log 2>&1 &\n"
    "\n"
)

execute_process(COMMAND chmod a+x "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.sh")


install(TARGETS ${PROJECT_NAME}-qmi-proxy DESTINATION ${INSTALL_BINDIR})
install(TARGETS ${PROJECT_NAME}-mbim-proxy DESTINATION ${INSTALL_BINDIR})
install(TARGETS ${PROJECT_NAME}-CM DESTINATION ${INSTALL_BINDIR})

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/99-${PROJECT_NAME}.rules DESTINATION /etc/udev/rules.d/)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.sh DESTINATION ${INSTALL_BINDIR})
install(PROGRAMS ${CMAKE_SOURCE_DIR}/default.script DESTINATION /usr/share/udhcpc/)


set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VENDOR "Quectel Wireless Solution, Co., Ltd.")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Quectel Wireless Solution, Co., Ltd.")
set(CPACK_PACKAGE_CONTACT "Sam4uk")# or CPACK_DEBIAN_PACKAGE_MAINTAINER)
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6 (>= ${GLIBC_VERION})")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})

# Автоматичне визначення архітектури для CPack
if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|arm64|armv8|armv7")
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "arm64")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|amd64")
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64")
endif()

set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_SOURCE_DIR}/decription)
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})
set(CPACK_PACKAGE_HOMEPAGE_URL ${PROJECT_HOMEPAGE_URL})


include(CPack)
