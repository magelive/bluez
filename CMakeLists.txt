cmake_minimum_required(VERSION 3.5)
project(bluez VERSION 5.56 LANGUAGES C CXX)

INCLUDE (CheckFunctionExists)
INCLUDE (CheckIncludeFiles)
INCLUDE (CheckTypeSize)
INCLUDE (CheckCCompilerFlag)
INCLUDE (TestBigEndian) 
INCLUDE (CheckLibraryExists)
INCLUDE (FindPkgConfig)
include (GNUInstallDirs)
include (CheckVariableExists)

# ................ configure.ac
#set(CMAKE_INSTALL_PREFIX "/usr/local")

if (MAINTAINER_MODE)
    FIND_PROGRAM(COVERAGE NAMES lcov)
    FIND_PROGRAM(DBUS_RUN_SESSION NAMES dbus-run-session)
    FIND_PROGRAM(VALGRIND NAMES valgrind)
    CHECK_INCLUDE_FILES(valgrind/memcheck.h HAVE_VALGRIND_MEMCHECK_H) 
endif()

CHECK_FUNCTION_EXISTS(explicit_bzero HAVE_EXPLICIT_BZERO)
CHECK_FUNCTION_EXISTS(rawmemchr HAVE_RAWMEMCHR)

CHECK_FUNCTION_EXISTS(signalfd HAVE_SIGNALFD)
if (NOT HAVE_SIGNALFD)
    message(FATAL_ERROR "signalfd support is required")
endif()

FIND_LIBRARY(RT NAMES rt)
CHECK_LIBRARY_EXISTS(rt clock_gettime ${RT} RT_VERIFIED)
if (NOT RT_VERIFIED)
    message(FATAL_ERROR "realtime clock support is required")
endif()

FIND_LIBRARY(PTHREAD NAMES pthread)
CHECK_LIBRARY_EXISTS(pthread pthread_create ${PTHREAD} PTHREAD_VERIFIED)
if (NOT PTHREAD_VERIFIED)
    message(FATAL_ERROR "posix thread support is required")
endif()

FIND_LIBRARY(DL NAMES dl)
CHECK_LIBRARY_EXISTS(dl dlopen ${DL} DL_VERIFIED)
if (NOT DL_VERIFIED)
    message(FATAL_ERROR "dynamic linking loader is required")
endif()

CHECK_INCLUDE_FILES(linux/types.h HAVE_LINUX_TYPES_H)

PKG_CHECK_MODULES(GLIB glib-2.0>=2.28)
if (NOT GLIB_FOUND)
    message(FATAL_ERROR "GLib >= 2.28 is required")
endif()

set(CFLAGS ${CFLAGS} ${GLIB_CFLAGS})
set(LIBS ${LIBS} ${GLIB_LIBRARIES})
#message(STATUS "GLIB version: ${GLIB_VERSION}")
#message(STATUS "GLIB CFLAGS: ${GLIB_CFLAGS}")
#message(STATUS "GLIB LIBS: ${GLIB_LIBRARIES}")

if (ENABLE_THREADS)
    set(NEED_THREADS 1)
    PKG_CHECK_MODULES(GTHREAD gthread-2.0>=2.16)
    if (NOT GTHREAD_FOUND)
        message(FATAL_ERROR "gthread >= 2.16 is required")
    endif()
    set(CFLAGS ${CFLAGS} ${GTHREAD_CFLAGS})
    set(LIBS ${LIBS} ${GTHREAD_LIBRARIES})
endif()

PKG_CHECK_MODULES(DBUS dbus-1>=1.6)
if (NOT DBUS_FOUND)
    message(FATAL_ERROR "D-Bus >= 1.6 is required")
endif()
set(CFLAGS ${CFLAGS} ${DBUS_CFLAGS})
set(LIBS ${LIBS} ${DBUS_LIBRARIES})

if (NOT DBUS_CONFDIR)
    PKG_GET_VARIABLE(DBUS_CONFDIR dbus-1 sysconfdir)
    if (NOT DBUS_CONFDIR)
        message(FATAL_ERROR "D-Bus configuration directory is required")
    endif()
endif()

if (NOT DBUS_SYSTEMBUSDIR)
    PKG_GET_VARIABLE(DBUS_SYSTEMBUSDIR dbus-1 system_bus_services_dir)
    if (NOT DBUS_SYSTEMBUSDIR)
        message(FATAL_ERROR "D-Bus system bus services directory is required")
    endif()
endif()


if (NOT DBUS_SESSIONBUSDIR)
    PKG_GET_VARIABLE(DBUS_SESSIONBUSDIR dbus-1 session_bus_services_dir)
    if (NOT DBUS_SESSIONBUSDIR)
        message(FATAL_ERROR "D-Bus session bus services directory is required")
    endif()
endif()


if (NOT ZSH_COMPLETIONDIR)
    set(ZSH_COMPLETIONDIR "${CMAKE_INSTALL_FULL_DATAROOTDIR}/zsh/site-functions")
else()
    set(ZSH_COMPLETIONS 1)
endif()

message(STATUS "ZSH_COMPLETIONDIR ist ${ZSH_COMPLETIONDIR}")
if (NOT ZSH_COMPLETIONS)
    message(STATUS "ZSH_COMPLETIONS not set")
endif()

if (ENABLE_BACKTRACE)
    CHECK_INCLUDE_FILES(elfutils/libdwfl.h HAVE_ELFUTILS_H)
    if (NOT HAVE_ELFUTILS_H)
        message(FATAL_ERROR "elfutils support is required")
    endif()
    set(HAVE_BACKTRACE_SUPPORT 1)
    SET(BACKTRACE_LIBRARIES dw)
    set(BACKTRACE_CFLAGS "")
    set(LIBS "${LIBS} dw")
endif()

if (ENABLE_LIBRARY)
    set(LIBRARY 1)
endif()

if(ENABLE_TEST)
    set(TEST 1)
endif()

if (ENABLE_NFC)
    set(NFC 1)
endif()

if (ENABLE_SAP)
    set(SAP 1)
endif()

set(A2DP 1)
if (DISABLE_A2DP)
    set(A2DP 0)
endif()

set(AVRCP 1)
if(DISABLE_AVRCP)
    set(AVRCP 0)
endif()

set(NETWORK 1)
if (DISABLE_NETWORK)
    set(NETWORK 0)
endif()

set(HID 1)
if (DISABLE_HID)
    set(HID 0)
endif()

set(HOG 1)
if (DISABLE_HOG)
    set(HOG 0)
endif()


if (ENABLE_HEALTH)
    set(HEALTH 1)
endif()

set(TOOLS 1)
if (DISABLE_TOOLS)
    set(TOOLS 0)
endif()

set(MONITOR 1)
if (DISABLE_MONITOR)
    set(MONITOR 0)
endif()

set(UDEV 1)
if (DISABLE_UDEV)
    set(UDEV 0)
endif()

if (TOOLS AND UDEV)
    PKG_CHECK_MODULES(UDEV libudev>=172)
    if (NOT UDEV_FOUND)
        message(FATAL_ERROR "libudev >= 172 is required")
    endif()

    set(CFLAGS ${CFLAGS} ${UDEV_CFLAGS})
    set(LIBS ${LIBS} ${UDEV_LIBRARIES})
    CHECK_LIBRARY_EXISTS(udev udev_hwdb_new ${UDEV} HAVE_UDEV_HWDB_NEW)
endif()

if (UDEV AND NOT UDEV_DIR)
    PKG_GET_VARIABLE(UDEV_DIR udev udevdir)
    if (NOT UDEV_DIR)
        message(FATAL_ERROR "udev directory is required")
    endif()
endif()

set(CUPS 1)
if (DISABLE_CUPS)
    set(CUPS 0)
endif()

if (ENABLE_MESH)
    set(MESH 1)
    PKG_CHECK_MODULES(JSONC json-c>=0.13)
    if (NOT JSONC_FOUND)
        message(FATAL_ERROR "json-c >= 0.13 is required")
    endif()
    set(CFLAGS ${CFLAGS} ${JSONC_CFLAGS})
    set(LIBS ${LIBS} ${JSONC_LIBRARIES})
endif()

if (ENABLE_MIDI)
    set(MIDI 1)
    PKG_CHECK_MODULES(ALSA alsa)
    if (NOT ALSA_FOUND)
        message(FATAL_ERROR "ALSA lib is required for MIDI support")
    endif()
	set(CFLAGS ${CFLAGS} ${ALSA_CFLAGS})
	set(LIBS ${LIBS} ${ALSA_LIBRARIES})
endif()

set(OBEX 1)
if (DISABLE_OBEX)
    set(OBEX 0)
else()
    PKG_CHECK_MODULES(ICAL libical)
    if (NOT ICAL_FOUND)
        message(FATAL_ERROR "libical is required")
    endif()
    set(CFLAGS ${CFLAGS} ${ICAL_CFLAGS})
    set(LIBS ${LIBS} ${ICAL_LIBRARIES})
endif()

if (ENABLE_BTPCLIENT)
    set(BTPCLIENT 1)
endif()

if (ENABLE_EXTERNAL_ELL)
    PKG_CHECK_MODULES(ELL ell>=0.37)
    if(NOT ELL_FOUND)
        message(FATAL_ERROR "Embedded Linux library >= 0.37 is required")
    endif()

    set(CFLAGS ${CFLAGS} ${ELL_CFLAGS})
    set(LIBS ${LIBS} ${ELL_LIBRARIES})
    set(EXTERNAL_ELL 1)
else()
    if (NOT ENABLE_BTPCLIENT AND NOT ENABLE_MESH)
        set(EXTERNAL_ELL 1)
    endif()
endif()

if (ENABLE_BTPCLIENT OR ENABLE_MESH)
    set(LIBSHARED_ELL 1)
endif()

set(CLIENT 1)
if (DISABLE_CLIENT)
    set(CLIENT 0)
endif()

if (NOT DISABLE_CLIENT OR ENABLE_MESH)
    CHECK_INCLUDE_FILES(readline/readline.h HAVE_READLINE_READLINE_H)
    if (NOT HAVE_READLINE_READLINE_H)
        message(FATAL_ERROR "readline header files are required")
    endif()
endif()

if (HAVE_READLINE_READLINE_H)
    set(READLINE 1)
endif()

set(SYSTEMD 1)
if (DISABLE_SYSTEMD)
    set(SYSTEMD 0)
else()
    if (NOT SYSTEMD_SYSTEMUNITDIR)
        PKG_GET_VARIABLE(SYSTEMD_SYSTEMUNITDIR systemd systemdsystemunitdir)
    
        if (NOT SYSTEMD_SYSTEMUNITDIR)
            message(FATAL_ERROR "systemd system unit directory is required")
        endif()
    endif()

    if (NOT SYSTEMD_USERUNITDIR)
        PKG_GET_VARIABLE(SYSTEMD_USERUNITDIR systemd systemduserunitdir)
        if (NOT SYSTEMD_USERUNITDIR)
            message(FATAL_ERROR "systemd user unit directory is required")
        endif()
    endif()
endif()

set(DATAFILES 1)
if (DISABLE_DATAFILES)
    set(DATAFILES 0)
endif()

if (ENABLE_MANPAGES)
    set(MANPAGES 1)
endif()

if (ENABLE_TESTING)
    set(TESTING 1)
endif()

if(ENABLE_EXPERIMENTAL)
    set(EXPERIMENTAL)
endif()

if(ENABLE_DEPRECATED)
    set(DEPRECATED 1)
endif()

if (ENABLE_SIXAXIS)
    set(SIXAXIS 1)
endif()

if (ENABLE_HID2HCI)
    set(HID2HCI)
endif()

if (ENABLE_LOGGER)
    set(LOGGER 1)
endif()

if (${CMAKE_INSTALL_PREFIX} EQUAL NONE)
    if (${CMAKE_INSTALL_LOCALSTATEDIR} EQUAL ${CMAKE_INSTALL_PREFIX}/var)
        set(CMAKE_INSTALL_LOCALSTATEDIR /var)
    endif()
    set(${CMAKE_INSTALL_PREFIX} ${CMAKE_SYSTEM_PREFIX_PATH})
endif()

if (${CMAKE_INSTALL_LOCALSTATEDIR} EQUAL ${CMAKE_INSTALL_PREFIX}/var)
    set(STORAGEDIR ${CMAKE_INSTALL_PREFIX}/var/lib/bluetooth)
else()
    set(STORAGEDIR ${CMAKE_INSTALL_LOCALSTATEDIR}/lib/bluetooth)
endif()

message(STATUS ">>>>>>>>>>>>> CMAKE_INSTALL_LOCALSTATEDIR is ${CMAKE_INSTALL_LOCALSTATEDIR}")
message(STATUS ">>>>>>>>>>>>> (CMAKE_INSTALL_PREFIX)/var is ${CMAKE_INSTALL_PREFIX}/var")
if (${CMAKE_INSTALL_SYSCONFDIR} EQUAL ${CMAKE_INSTALL_PREFIX}/etc)
    set(CONFIGDIR ${CMAKE_INSTALL_PREFIX}/etc/bluetooth)
else()
    set(CONFIGDIR ${CMAKE_INSTALL_SYSCONFDIR}/bluetooth)
endif()

set(MESH_STORAGEDIR ${STORAGEDIR}/mesh)

if (ENABLE_ANDROID)
    set(ANDROID 1)

    PKG_CHECK_MODULES(SBC sbc>=1.2)
    if (NOT SBC_FOUND)
		message(FATAL_ERROR "SBC library >= 1.2 is required")
    endif()

	set(CFLAGS ${CFLAGS} ${SBC_CFLAGS})
	set(LIBS ${LIBS} ${SBC_LIBRARIES})

    PKG_CHECK_MODULES(SPEEXDSP speexdsp>=1.2)
    if (NOT SPEEXDSP_FOUND)
        message(FATAL_ERROR "library >= 1.2 is required")
    endif()

    set(CFLAGS ${CFLAGS} ${SPEEXDSP_CFLAGS})
	set(LIBS ${LIBS} ${SPEEXDSP_LIBRARIES})

endif()

set(ANDROID_STORAGEDIR ${STORAGEDIR}/android)

CHECK_INCLUDE_FILES(inttypes.h HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES(linux/if_alg.h HAVE_LINUX_IF_ALG_H)
CHECK_INCLUDE_FILES(memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILES(stdlib.h HAVE_STDLIB_H)
CHECK_INCLUDE_FILES(stdint.h HAVE_STDINT_H)
CHECK_INCLUDE_FILES(strings.h HAVE_STRINGS_H)
CHECK_INCLUDE_FILES(string.h HAVE_STRING_H)
CHECK_INCLUDE_FILES(sys/stat.h HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILES(sys/types.h HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)

foreach(ac_kw __restrict __restrict__ _Restrict restrict)
CHECK_C_SOURCE_COMPILES(
    "int test (void *${ac_kw} x); int main (void) {return 0;}"
    HAVE_RESTRICT)
if (HAVE_RESTRICT)
    set(restrict ${ac_kw})
    break()
endif()
endforeach()

ADD_DEFINITIONS(-DHAVE_CONFIG_H)
configure_file(config.h.cmake include/config.h @ONLY)

#.....................Makefile.am...................

set(AM_CFLAGS ${WARNING_CFLAGS} ${MISC_CFLAGS} ${UDEV_CFLAGS} ${ELL_CFLAGS})

if (DATAFILES)
    set(dbusdir ${DBUS_CONFDIR}/dbus-1/system.d)
    set(dbus_DATA src/bluetooth.conf)
    set(confdir ${CMAKE_INSTALL_SYSCONFDIR}/bluetooth)
    #set(conf_DATA "")
    set(statedir ${CMAKE_INSTALL_LOCALSTATEDIR}/lib/bluetooth)
    #set(state_DATA "")
endif()

if (SYSTEMD)
    set(systemdsystemunitdir ${SYSTEMD_SYSTEMUNITDIR})
    set(systemdsystemunit_DATA src/bluetooth.service)
    set(dbussystembusdir ${DBUS_SYSTEMBUSDIR})
    set(dbussystembus_DATA src/org.bluez.service)
endif()

set(EXTRA_DIST ${EXTRA_DIST} src/bluetooth.service.in src/org.bluez.service)
set(plugindir ${CMAKE_INSTALL_LIBDIR}/bluetooth/plugins)

if (MAINTAINER_MODE)
set(build_plugindir ${CMAKE_CURRENT_BINARY_DIR}/plugins)
else( )
set(build_plugindir ${plugindir})
endif()



set(lib_sources lib/bluetooth.c lib/hci.c lib/sdp.c)
set(lib_headers lib/bluetooth.h lib/hci.h lib/hci_lib.h
                lib/sco.h lib/l2cap.h lib/sdp.h lib/sdp_lib.h
                lib/rfcomm.h lib/bnep.h lib/cmtp.h lib/hidp.h)

set(extra_headers lib/mgmt.h lib/uuid.h lib/a2mp.h lib/amp.h)
set(extra_sources lib/uuid.c)
set(BUILT_SOURCES ${local_headers} ${ell_built_sources} src/builtin.h)

FILE(COPY ${lib_headers} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/include/bluetooth)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR}/include)
INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR})
INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/src)

if (LIBRARY)
    set(bluetooth_static "bluetooth_static")
    set(bluetooth_shared "bluetooth_shared")

    ADD_LIBRARY(${bluetooth_static} STATIC ${LIB_SRCS} ${LIB_HEADERS})
    ADD_LIBRARY(${bluetooth_shared} SHARED ${LIB_SRCS} ${LIB_HEADERS})

    SET_TARGET_PROPERTIES(${bluetooth_static} PROPERTIES STATIC_LIBRARY_FLAGS "-version-info 22:4:19")
    SET_TARGET_PROPERTIES(${bluetooth_shared} PROPERTIES LINK_FLAGS "-version-info 22:4:19")
    
    SET_TARGET_PROPERTIES(${bluetooth_static} PROPERTIES OUTPUT_NAME bluetooth)
    SET_TARGET_PROPERTIES(${bluetooth_shared} PROPERTIES OUTPUT_NAME bluetooth)

    INSTALL(TARGETS ${bluetooth_static} ARCHIVE DESTINATION lib)
    INSTALL(TARGETS ${bluetooth_shared} LIBRARY DESTINATION lib)
endif()

ADD_LIBRARY(bluetooth-internal STATIC 
    ${LIB_HEADERS} ${LIB_SRCS} ${extra_headers} ${extra_sources})
#FILE(COPY bluetooth-internal DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/lib)

set(GDBUS_INTERNAL_SRCS gdbus/gdbus.h gdbus/mainloop.c 
                        gdbus/watch.c gdbus/object.c 
                        gdbus/client.c gdbus/polkit.c)
ADD_LIBRARY(gdbus-internal STATIC ${GDBUS_INTERNAL_SRCS})
#FILE(COPY gdbus-internal DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/gdbus)

if (EXTERNAL_ELL)
    set(ell_cflags ${ELL_CFLAGS})
    set(ell_ldadd ${ELL_LIBRARIES})
    #set(ell_dependencies "")
    #set(ell_built_sources "")
else()
    set(ell_built_sources ell/internal ell/ell.h)
    set(ell_ldadd ell-internal)
    
    set(ell_headers ell/util.h ell/log.h  ell/queue.h ell/hashmap.h
                    ell/random.h ell/signal.h ell/timeout.h ell/cipher.h
                    ell/checksum.h ell/io.h ell/idle.h ell/main.h ell/settings.h
                    ell/strv.h ell/string.h ell/utf8.h ell/dbus.h ell/dbus-service.h
                    ell/dbus-client.h ell/key.h ell/cert.h ell/pem.h ell/base64.h 
                    ell/asn1-private.h ell/cert-private.h ell/pem-private.h ell/uuid.h)

    set(ell_sources ell/private.h ell/missing.h
                    ell/util.c ell/log.c ell/queue.c ell/hashmap.c ell/random.c
                    ell/signal.c ell/timeout.c ell/io.c ell/idle.c ell/main.c
                    ell/settings.c ell/strv.c ell/string.c ell/cipher.c
                    ell/checksum.c ell/pem.c ell/cert.c ell/cert-crypto.c
                    ell/key.c ell/base64.c ell/utf8.c ell/dbus-private.h
                    ell/dbus.c ell/dbus-message.c ell/dbus-util.c ell/dbus-service.c
                    ell/dbus-client.c ell/dbus-name-cache.c ell/dbus-filter.c
                    ell/gvariant-private.h ell/gvariant-util.c ell/siphash-private.h
                    ell/siphash.c ell/uuid.c)

    ADD_LIBRARY(ell-internal STATIC ${ell_headers} ${ell_sources})
    #FILE(COPY ell-internal DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/ell)
endif()

set(CLEANFILES ${CLEANFILES} ${ell_built_sources})

set(shared_sources  src/shared/io.h src/shared/timeout.h 
                    src/shared/queue.h src/shared/queue.c 
                    src/shared/util.h src/shared/util.c 
                    src/shared/mgmt.h src/shared/mgmt.c 
                    src/shared/crypto.h src/shared/crypto.c 
                    src/shared/ecc.h src/shared/ecc.c 
                    src/shared/ringbuf.h src/shared/ringbuf.c 
                    src/shared/tester.h src/shared/tester.c 
                    src/shared/hci.h src/shared/hci.c 
                    src/shared/hci-crypto.h src/shared/hci-crypto.c 
                    src/shared/hfp.h src/shared/hfp.c 
                    src/shared/uhid.h src/shared/uhid.c 
                    src/shared/pcap.h src/shared/pcap.c 
                    src/shared/btsnoop.h src/shared/btsnoop.c 
                    src/shared/ad.h src/shared/ad.c 
                    src/shared/att-types.h 
                    src/shared/att.h src/shared/att.c 
                    src/shared/gatt-helpers.h src/shared/gatt-helpers.c 
                    src/shared/gatt-client.h src/shared/gatt-client.c 
                    src/shared/gatt-server.h src/shared/gatt-server.c 
                    src/shared/gatt-db.h src/shared/gatt-db.c 
                    src/shared/gap.h src/shared/gap.c 
                    src/shared/log.h src/shared/log.c 
                    src/shared/tty.h)

if (READLINE)
    set(${shared_sources} ${shared_sources} src/shared/shell.c src/shared/shell.h)
endif()

set(shared_glib_SRCS ${shared_sources}
                    src/shared/io-glib.c 
                    src/shared/timeout-glib.c 
                    src/shared/mainloop-glib.c 
                    src/shared/mainloop-notify.h 
                    src/shared/mainloop-notify.c)
ADD_LIBRARY(shared-glib STATIC ${shared_glib_SRCS})
#FILE(COPY shared-glib DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/src)

set(shared_mainloop_SRCS ${shared_sources}
				src/shared/io-mainloop.c 
				src/shared/timeout-mainloop.c 
				src/shared/mainloop.h src/shared/mainloop.c 
				src/shared/mainloop-notify.h 
				src/shared/mainloop-notify.c)
ADD_LIBRARY(shared-mainloop STATIC ${shared_mainloop_SRCS})
#FILE(COPY shared-mainloop DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/src)

if (LIBSHARED_ELL)
    set(shared_ell_SRCS ${shared_sources}
                    src/shared/io-ell.c 
                    src/shared/timeout-ell.c 
                    src/shared/mainloop.h 
                    src/shared/mainloop-ell.c)
    ADD_LIBRARY(shared-ell STATIC ${shared_ell_SRCS})
    #FILE(COPY shared-ell DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/src)
endif()

set(attrib_sources attrib/att.h attrib/att-database.h attrib/att.c 
		attrib/gatt.h attrib/gatt.c 
		attrib/gattrib.h attrib/gattrib.c 
		attrib/gatt-service.h attrib/gatt-service.c)

set(btio_sources btio/btio.h btio/btio.c)

set(gobex_sources gobex/gobex.h gobex/gobex.c 
			    gobex/gobex-defs.h gobex/gobex-defs.c 
			    gobex/gobex-packet.c gobex/gobex-packet.h 
			    gobex/gobex-header.c gobex/gobex-header.h 
			    gobex/gobex-transfer.c gobex/gobex-debug.h 
			    gobex/gobex-apparam.c gobex/gobex-apparam.h)

#........ Makefile.plugins......
set(builtin_modules hostname wiimote 
                    autopair policy)

set(builtin_modules_SRCS plugins/hostname.c plugins/wiimote.c 
                        plugins/autopair.c plugins/policy.c)

if (NFC)
    set(builtin_modules ${builtin_modules} neard)
    set(builtin_sources ${builtin_sources} plugins/neard.c)
endif()
                        
if (SAP)
    set(builtin_modules ${builtin_modules} sap)
    set(builtin_sources ${builtin_sources} profiles/sap/main.c profiles/sap/manager.h 
                                    profiles/sap/manager.c profiles/sap/server.h 
                                    profiles/sap/server.c profiles/sap/sap.h 
                                    profiles/sap/sap-dummy.c)
endif()
                        
if (A2DP)
    set(builtin_modules ${builtin_modules} a2dp)
    set(builtin_sources ${builtin_sources} profiles/audio/source.h profiles/audio/source.c 
                        profiles/audio/sink.h profiles/audio/sink.c 
                        profiles/audio/a2dp.h profiles/audio/a2dp.c 
                        profiles/audio/avdtp.h profiles/audio/avdtp.c 
                        profiles/audio/media.h profiles/audio/media.c 
                        profiles/audio/transport.h profiles/audio/transport.c 
                        profiles/audio/a2dp-codecs.h)
endif()
                        
                        
if (AVRCP)
    set(builtin_modules ${builtin_modules} avrcp)
    set(builtin_sources ${builtin_sources} profiles/audio/control.h profiles/audio/control.c 
                        profiles/audio/avctp.h profiles/audio/avctp.c 
                        profiles/audio/avrcp.h profiles/audio/avrcp.c 
                        profiles/audio/player.h profiles/audio/player.c)
endif()
                        
if (NETWORK)
    set(builtin_modules ${builtin_modules} network)
    set(builtin_sources ${builtin_sources} profiles/network/manager.c 
                        profiles/network/bnep.h profiles/network/bnep.c 
                        profiles/network/server.h profiles/network/server.c 
                        profiles/network/connection.h 
                        profiles/network/connection.c)
endif()
                        
if (HID)
    set(builtin_modules ${builtin_modules} input)
    set(builtin_sources ${builtin_sources} profiles/input/manager.c 
            profiles/input/server.h profiles/input/server.c 
            profiles/input/device.h profiles/input/device.c 
            profiles/input/hidp_defs.h profiles/input/sixaxis.h)
endif()

if (HOG)
    set(builtin_modules ${builtin_modules} hog)
    set(builtin_sources ${builtin_sources} profiles/input/hog.c profiles/input/uhid_copy.h 
            profiles/input/hog-lib.c profiles/input/hog-lib.h 
            profiles/deviceinfo/dis.c profiles/deviceinfo/dis.h 
            profiles/battery/bas.c profiles/battery/bas.h 
            profiles/scanparam/scpp.c profiles/scanparam/scpp.h 
            profiles/input/suspend.h profiles/input/suspend-none.c)

    set(EXTRA_DIST ${EXTRA_DIST} profiles/input/suspend-dummy.c)
endif()

if (HEALTH)
    set(builtin_modules ${builtin_modules} health)
    set(builtin_sources ${builtin_sources} profiles/health/mcap.h profiles/health/mcap.c 
            profiles/health/hdp_main.c profiles/health/hdp_types.h 
            profiles/health/hdp_manager.h 
            profiles/health/hdp_manager.c 
            profiles/health/hdp.h profiles/health/hdp.c 
            profiles/health/hdp_util.h profiles/health/hdp_util.c)
endif()

set(builtin_modules ${builtin_modules} gap scanparam deviceinfo)
set(builtin_sources ${builtin_sources} profiles/gap/gas.c 
            profiles/scanparam/scan.c profiles/deviceinfo/deviceinfo.c)

if (MIDI)
    set(builtin_modules ${builtin_modules} midi)
    set(builtin_sources ${builtin_sources} profiles/midi/midi.c 
                profiles/midi/libmidi.h profiles/midi/libmidi.c)
    set(builtin_cppflags ${builtin_cppflags} ${ALSA_CFLAGS})
    set(builtin_ldadd ${builtin_ldadd} ${ALSA_LIBRARIES})
endif()

set(builtin_modules ${builtin_modules} battery)
set(builtin_sources ${builtin_sources} profiles/battery/battery.c)

if (SIXAXIS)
    set(sixaxis_static "sixaxis_static")
    set(sixaxis_shared "sixaxis_shared")

    ADD_LIBRARY(${sixaxis_static} STATIC plugins/sixaxis.c)
    ADD_LIBRARY(${sixaxis_shared} SHARED plugins/sixaxis.c)

    TARGET_LINK_LIBRARIES(${sixaxis_static} INTERFACE ${UDEV_LIBRARIES})    
    TARGET_LINK_LIBRARIES(${sixaxis_shared} INTERFACE ${UDEV_LIBRARIES})

    SET_TARGET_PROPERTIES(${sixaxis_static} PROPERTIES STATIC_LIBRARY_FLAGS "${AM_LDFLAGS} -module -avoid-version")
    SET_TARGET_PROPERTIES(${sixaxis_shared} PROPERTIES LINK_FLAGS "${AM_LDFLAGS} -module -avoid-version")

    SET_TARGET_PROPERTIES(${sixaxis_static} PROPERTIES COMPILE_FLAGS "${AM_CFLAGS} -fvisibility=hidden")

    SET_TARGET_PROPERTIES(${sixaxis_static} PROPERTIES OUTPUT_NAME sixaxis)
    SET_TARGET_PROPERTIES(${sixaxis_shared} PROPERTIES OUTPUT_NAME sixaxis)

    FILE(COPY ${sixaxis_static} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/plugins)
    FILE(COPY ${sixaxis_shared} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/plugins)

    INSTALL(TARGETS ${sixaxis_static} ARCHIVE DESTINATION lib)
    INSTALL(TARGETS ${sixaxis_shared} LIBRARY DESTINATION lib)
endif()

if (MAINTAINER_MODE)
    set(external_dummy_static "external-dummy_static")
    set(external_dummy_shared "external-dummy_shared")

    ADD_LIBRARY(${external_dummy_static} STATIC plugins/external-dummy.c)
    ADD_LIBRARY(${external_dummy_shared} SHARED plugins/external-dummy.c)

    TARGET_LINK_LIBRARIES(${external_dummy_static} INTERFACE ${UDEV_LIBRARIES})    
    TARGET_LINK_LIBRARIES(${external_dummy_shared} INTERFACE ${UDEV_LIBRARIES})

    SET_TARGET_PROPERTIES(${external_dummy_static} PROPERTIES STATIC_LIBRARY_FLAGS "${AM_LDFLAGS} -module -avoid-version -no-undefined")
    SET_TARGET_PROPERTIES(${external_dummy_shared} PROPERTIES LINK_FLAGS "${AM_LDFLAGS} -module -avoid-version")

    SET_TARGET_PROPERTIES(${external_dummy_static} PROPERTIES COMPILE_FLAGS "${AM_CFLAGS} -fvisibility=hidden")

    SET_TARGET_PROPERTIES(${external_dummy_static} PROPERTIES OUTPUT_NAME external-dummy)
    SET_TARGET_PROPERTIES(${external_dummy_shared} PROPERTIES OUTPUT_NAME external-dummy)

    FILE(COPY ${external_dummy_static} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/plugins)
    FILE(COPY ${external_dummy_shared} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/plugins)

    INSTALL(TARGETS ${external_dummy_static} ARCHIVE DESTINATION lib)
    INSTALL(TARGETS ${external_dummy_shared} LIBRARY DESTINATION lib)
endif()

set(src_bluetoothd_SOURCES ${builtin_sources}
                            ${attrib_sources} 
                            ${btio_sources}
                            src/bluetooth.ver
                            src/main.c src/log.h src/log.c
                            src/backtrace.h src/backtrace.c
                            src/rfkill.c src/btd.h src/sdpd.h
                            src/sdpd-server.c src/sdpd-request.c
                            src/sdpd-service.c src/sdpd-database.c
                            src/attrib-server.h src/attrib-server.c
                            src/gatt-database.h src/gatt-database.c
                            src/sdp-xml.h src/sdp-xml.c
                            src/sdp-client.h src/sdp-client.c
                            src/textfile.h src/textfile.c
                            src/uuid-helper.h src/uuid-helper.c
                            src/uinput.h
                            src/plugin.h src/plugin.c
                            src/storage.h src/storage.c
                            src/advertising.h src/advertising.c
                            src/agent.h src/agent.c
                            src/error.h src/error.c
                            src/adapter.h src/adapter.c
                            src/profile.h src/profile.c
                            src/service.h src/service.c
                            src/gatt-client.h src/gatt-client.c
                            src/device.h src/device.c
                            src/dbus-common.c src/dbus-common.h
                            src/eir.h src/eir.c
                            src/adv_monitor.h src/adv_monitor.c
                            src/battery.h src/battery.c)

ADD_DEPENDENCIES(blutoothd bluetooth-internal gdbus-internal shared-glib)

ADD_EXECUTABLE(bluetoothd ${src_bluetoothd_SOURCES})
TARGET_LINK_LIBRARIES(bluetoothd PRIVATE bluetooth-internal gdbus-internal shared-glib)
TARGET_LINK_LIBRARIES(bluetoothd INTERFACE ${BACKTRACE_LIBRARIES} 
                            ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} dl rt ${builtin_ldadd})

SET_TARGET_PROPERTIES(bluetoothd PROPERTIES LINK_FLAGS 
                "-Wl,--export-dynamic -Wl,--version-script=${CMAKE_SOURCE_DIR}/src/bluetooth.ver")
SET_TARGET_PROPERTIES(bluetoothd PROPERTIES COMPILE_FLAGS 
    "${AM_CFLAGS} -DBLUETOOTH_PLUGIN_BUILTIN  -DPLUGINDIR=\"${build_plugindir}\" ${BACKTRACE_CFLAGS} ${builtin_cppflags}")

set(builtin_files ${builtin_nodist})

set(nodist_src_bluetoothd_SOURCES ${builtin_files})

set(CLEANFILES ${CLEANFILES} ${builtin_files} src/bluetooth.service)

set(man_MANS src/bluetoothd.8)

set(EXTRA_DIST ${EXTRA_DIST} src/genbuiltin src/bluetooth.conf
			src/main.conf profiles/network/network.conf
			profiles/input/input.conf)

#.......Makefile.tools
if (CLIENT)
    SET(bluetoothctl_SOURCES client/main.c
					client/display.h client/display.c 
					client/agent.h client/agent.c 
					client/advertising.h 
					client/advertising.c 
					client/adv_monitor.h 
					client/adv_monitor.c 
					client/gatt.h client/gatt.c)
    ADD_EXECUTABLE(bluetoothctl ${bluetoothctl_SOURCES})
    TARGET_LINK_LIBRARIES(bluetoothctl PRIVATE gdbus-internal shared-glib)
    TARGET_LINK_LIBRARIES(bluetoothctl INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} readline)
    INSTALL(TARGETS bluetoothctl RUNTIME DESTINATION bin)
endif()

if (ZSH_COMPLETIONS)
    set(zshcompletiondir ${ZSH_COMPLETIONDIR})
    set(dist_zshcompletion_DATA  completion/zsh/_bluetoothctl)
endif()

if (MONITOR)
    set(monitor_btmon_SOURCES monitor/main.c monitor/bt.h 
				monitor/display.h monitor/display.c 
				monitor/hcidump.h monitor/hcidump.c 
				monitor/ellisys.h monitor/ellisys.c 
				monitor/control.h monitor/control.c 
				monitor/packet.h monitor/packet.c 
				monitor/vendor.h monitor/vendor.c 
				monitor/lmp.h monitor/lmp.c 
				monitor/crc.h monitor/crc.c 
				monitor/ll.h monitor/ll.c 
				monitor/l2cap.h monitor/l2cap.c 
				monitor/sdp.h monitor/sdp.c 
				monitor/avctp.h monitor/avctp.c 
				monitor/avdtp.h monitor/avdtp.c 
				monitor/a2dp.h monitor/a2dp.c 
				monitor/rfcomm.h monitor/rfcomm.c 
				monitor/bnep.h monitor/bnep.c 
				monitor/hwdb.h monitor/hwdb.c 
				monitor/keys.h monitor/keys.c 
				monitor/analyze.h monitor/analyze.c 
				monitor/intel.h monitor/intel.c 
				monitor/broadcom.h monitor/broadcom.c 
				monitor/jlink.h monitor/jlink.c 
				monitor/tty.h)
    ADD_EXECUTABLE(btmon ${monitor_btmon_SOURCES})
    TARGET_LINK_LIBRARIES(btmon PRIVATE bluetooth-internal shared-mainloop)
    TARGET_LINK_LIBRARIES(btmon INTERFACE ${UDEV_LIBRARIES} dl)
    INSTALL(TARGETS btmon RUNTIME DESTINATION bin)
endif()

if (LOGGER)
    ADD_EXECUTABLE(btmon-logger tools/btmon-logger.c)
    ADD_DEPENDENCIES(btmon-logger shared-mainloop tools/bluetooth-logger.service)
    TARGET_LINK_LIBRARIES(btmon-logger PRIVATE shared-mainloop)
    if (SYSTEMD)
        set(systemdsystemunit_DATA ${systemdsystemunit_DATA} tools/bluetooth-logger.service)
    endif()
    INSTALL(TARGETS btmon-logger RUNTIME DESTINATION bin)
endif()

set(CLEANFILES tools/bluetooth-logger.service)
set(EXTRA_DIST tools/bluetooth-logger.service.in)

if (TESTING)
    set(emulator_btvirt_SOURCES emulator/main.c monitor/bt.h 
                    emulator/serial.h emulator/serial.c 
                    emulator/server.h emulator/server.c 
                    emulator/vhci.h emulator/vhci.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c 
                    emulator/phy.h emulator/phy.c 
                    emulator/amp.h emulator/amp.c 
                    emulator/le.h emulator/le.c)
    ADD_EXECUTABLE(btvirt ${emulator_btvirt_SOURCES})
    TARGET_LINK_LIBRARIES(btvirt PRIVATE bluetooth-internal shared-mainloop)


    ADD_EXECUTABLE(b1ee emulator/b1ee.c)
    TARGET_LINK_LIBRARIES(b1ee PRIVATE shared-mainloop)

    ADD_EXECUTABLE(hfp emulator/hfp.c)
    TARGET_LINK_LIBRARIES(hfp PRIVATE shared-mainloop)

    ADD_EXECUTABLE(btsensor peripheral/main.c 
                    peripheral/efivars.h peripheral/efivars.c 
                    peripheral/attach.h peripheral/attach.c 
                    peripheral/log.h peripheral/log.c 
                    peripheral/gap.h peripheral/gap.c 
                    peripheral/gatt.h peripheral/gatt.c)
    TARGET_LINK_LIBRARIES(btsensor PRIVATE shared-mainloop bluetooth-internal)

    ADD_EXECUTABLE(3dsp tools/3dsp.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(3dsp PRIVATE shared-mainloop)

    ADD_EXECUTABLE(mgmt_tester tools/mgmt-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c)
    TARGET_LINK_LIBRARIES(mgmt_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(mgmt_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(l2cap_tester tools/l2cap-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c)
    TARGET_LINK_LIBRARIES(l2cap_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(l2cap_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(rfcomm_tester tools/rfcomm-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c)
    TARGET_LINK_LIBRARIES(rfcomm_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(rfcomm_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(bnep_tester tools/bnep-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c
    TARGET_LINK_LIBRARIES(bnep_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(bnep_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(smp_tester tools/smp-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c)
    TARGET_LINK_LIBRARIES(smp_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(smp_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(gap_tester tools/gap-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c)
    TARGET_LINK_LIBRARIES(gap_tester PRIVATE  bluetooth-internal gdbus-internal shared-glib)
    TARGET_LINK_LIBRARIES(gap_tester INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES})

    ADD_EXECUTABLE(sco_tester tools/sco-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c
    TARGET_LINK_LIBRARIES(sco_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(sco_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(hci_tester tools/hci-tester.c monitor/bt.h
    TARGET_LINK_LIBRARIES(hci_tester PRIVATE shared-glib)
    TARGET_LINK_LIBRARIES(hci_tester INTERFACE ${GLIB_LIBRARIES})

    ADD_EXECUTABLE(userchan_tester tools/userchan-tester.c monitor/bt.h 
                    emulator/hciemu.h emulator/hciemu.c 
                    emulator/btdev.h emulator/btdev.c 
                    emulator/bthost.h emulator/bthost.c 
                    emulator/smp.c
    TARGET_LINK_LIBRARIES(userchan_tester PRIVATE bluetooth-internal shared-glib)
    TARGET_LINK_LIBRARIES(userchan_tester INTERFACE ${GLIB_LIBRARIES})
endif()


if (TOOLS)
    set(install_PROGRAMS rctest l2test l2ping bluemoon hex2hcd mpris-proxy btattach)
    
    ADD_EXECUTABLE(bdaddr tools/bdaddr.c src/oui.h src/oui.c)
    TARGET_LINK_LIBRARIES(bdaddr PRIVATE bluetooth-internal)
    TARGET_LINK_LIBRARIES(bdaddr INTERFACE ${GLIB_LIBRARIES})
    
    ADD_EXECUTABLE(avinfo tools/avinfo.c)
    TARGET_LINK_LIBRARIES(avinfo PRIVATE bluetooth-internal)

    ADD_EXECUTABLE(avtest tools/avtest.c)
    TARGET_LINK_LIBRARIES(avtest PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(scotest tools/scotest.c)
    TARGET_LINK_LIBRARIES(scotest PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(amptest tools/amptest.c)
    TARGET_LINK_LIBRARIES(amptest PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(hwdb tools/hwdb.c)
    TARGET_LINK_LIBRARIES(hwdb PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(hcieventmask tools/hcieventmask.c)
    TARGET_LINK_LIBRARIES(hcieventmask PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(btinfo tools/btinfo.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(btinfo shared-mainloop)
    
    ADD_EXECUTABLE(btattach tools/btattach.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(btattach shared-mainloop)
    
    ADD_EXECUTABLE(btconfig tools/btconfig.c)
    TARGET_LINK_LIBRARIES(btconfig shared-mainloop)
    
    ADD_EXECUTABLE(btsnoop tools/btsnoop.c)
    TARGET_LINK_LIBRARIES(btsnoop shared-mainloop)
    
    ADD_EXECUTABLE(btproxy tools/btproxy.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(btproxy shared-mainloop)
    
    ADD_EXECUTABLE(btiotest tools/btiotest.c btio/btio.h btio/btio.c
    TARGET_LINK_LIBRARIES(btiotest PRIVATE bluetooth-internal)
    TARGET_LINK_LIBRARIES(btiotest INTERFACE ${GLIB_LIBRARIES}
    
    ADD_EXECUTABLE(mcaptest tools/mcaptest.c 
                    btio/btio.h btio/btio.c 
                    src/log.c src/log.h 
                    profiles/health/mcap.h profiles/health/mcap.c
    TARGET_LINK_LIBRARIES(mcaptest PRIVATE bluetooth-internal shared-mainloop)
    TARGET_LINK_LIBRARIES(mcaptest INTERFACE ${GLIB_LIBRARIES} rt)
    
    ADD_EXECUTABLE(bneptest tools/bneptest.c 
                    btio/btio.h btio/btio.c 
                    src/log.h src/log.c 
                    profiles/network/bnep.h profiles/network/bnep.c)
    TARGET_LINK_LIBRARIES(bneptest PRIVATE bluetooth-internal shared-mainloop)
    TARGET_LINK_LIBRARIES(bneptest INTERFACE ${GLIB_LIBRARIES})
    
    ADD_EXECUTABLE(cltest tools/cltest.c)
    TARGET_LINK_LIBRARIES(cltest PRIVATE bluetooth-internal src/libshared-mainloop)
    
    ADD_EXECUTABLE(oobtest tools/oobtest.c)
    TARGET_LINK_LIBRARIES(oobtest PRIVATE bluetooth-internal src/libshared-mainloop)
    
    ADD_EXECUTABLE(advtest tools/advtest.c)
    TARGET_LINK_LIBRARIES(advtest PRIVATE bluetooth-internal src/libshared-mainloop)
    
    ADD_EXECUTABLE(seq2bseq tools/seq2bseq.c)
    
    ADD_EXECUTABLE(nokfw tools/nokfw.c)
    
    ADD_EXECUTABLE(rtlfw tools/rtlfw.c)
    
    ADD_EXECUTABLE(create_image tools/create-image.c)
    
    ADD_EXECUTABLE(eddystone tools/eddystone.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(eddystone shared-mainloop)
    
    ADD_EXECUTABLE(ibeacon tools/ibeacon.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(ibeacon shared-mainloop)
    
    ADD_EXECUTABLE(btgatt_client tools/btgatt-client.c src/uuid-helper.c)
    TARGET_LINK_LIBRARIES(btgatt_client PRIVATE shared-mainloop bluetooth-internal)
    
    ADD_EXECUTABLE(btgatt_server tools/btgatt-server.c src/uuid-helper.c)
    TARGET_LINK_LIBRARIES(btgatt_server PRIVATE shared-mainloop bluetooth-internal)
    
    ADD_EXECUTABLE(rctest tools/rctest.c)
    TARGET_LINK_LIBRARIES(rctest PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(l2test tools/l2test.c)
    TARGET_LINK_LIBRARIES(l2test PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(l2ping tools/l2ping.c)
    TARGET_LINK_LIBRARIES(l2ping PRIVATE bluetooth-internal)
    
    ADD_EXECUTABLE(bluemoon tools/bluemoon.c monitor/bt.h)
    TARGET_LINK_LIBRARIES(bluemoon PRIVATE shared-mainloop)
    
    ADD_EXECUTABLE(hex2hcd tools/hex2hcd.c)
    
    ADD_EXECUTABLE(mpris_proxy tools/mpris-proxy.c)
    TARGET_LINK_LIBRARIES(mpris_proxy PRIVATE gdbus-internal)
    TARGET_LINK_LIBRARIES(mpris_proxy INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} 
    
    ADD_EXECUTABLE(gatt_service tools/gatt-service.c)
    TARGET_LINK_LIBRARIES(gatt_service INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES}  
    TARGET_LINK_LIBRARIES(gatt_service PRIVATE gdbus/libgdbus-internal)
    
    ADD_EXECUTABLE(iapd profiles/iap/main.c)
    TARGET_LINK_LIBRARIES(iapd PRIVATE gdbus-internal)
    TARGET_LINK_LIBRARIES(iapd INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} 

    INSTALL(TARGETS ${install_PROGRAMS} RUNTIME DESTINATION bin)

    set(dist_man_MANS ${dist_man_MANS} tools/rctest.1 tools/l2ping.1 tools/btattach.1)

    set(EXTRA_DIST ${EXTRA_DIST} tools/bdaddr.1)

    if (MESH)
        set(meshctl_SOURCES tools/meshctl.c 
                        tools/mesh/agent.h tools/mesh/agent.c 
                        tools/mesh/config-model.h
                        tools/mesh-gatt/mesh-net.h 
                        tools/mesh-gatt/node.h tools/mesh-gatt/node.c 
                        tools/mesh-gatt/gatt.h tools/mesh-gatt/gatt.c 
                        tools/mesh-gatt/crypto.h
                        tools/mesh-gatt/crypto.c 
                        tools/mesh-gatt/keys.h 
                        tools/mesh-gatt/net.h tools/mesh-gatt/net.c 
                        tools/mesh-gatt/prov.h tools/mesh-gatt/prov.c 
                        tools/mesh-gatt/util.h tools/mesh-gatt/util.c 
                        tools/mesh-gatt/prov-db.h 
                        tools/mesh-gatt/prov-db.c 
                        tools/mesh-gatt/config-client.c 
                        tools/mesh-gatt/config-server.c 
                        tools/mesh-gatt/onoff-model.h 
                        tools/mesh-gatt/onoff-model.c)
        ADD_EXECUTABLE(meshctl ${meshctl_SOURCES})
        TARGET_LINK_LIBRARIES(meshctl PRIVATE gdbus-internal shared-glib bluetooth-internal)
        TARGET_LINK_LIBRARIES(meshctl INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} json-c readline)
        INSTALL(TARGET meshctl RUNTIME DESTINATION bin)

        
        set(mesh_cfgclient_SOURCES tools/mesh-cfgclient.c 
                        tools/mesh/model.h tools/mesh/config-model.h 
                        tools/mesh/cfgcli.h tools/mesh/cfgcli.c 
                        tools/mesh/keys.h tools/mesh/keys.c 
                        tools/mesh/util.h tools/mesh/util.c 
                        tools/mesh/remote.h tools/mesh/remote.c 
                        tools/mesh/agent.h tools/mesh/agent.c 
                        tools/mesh/mesh-db.h tools/mesh/mesh-db.c 
                        mesh/util.h mesh/util.c 
                        mesh/crypto.h mesh/crypto.c)
        ADD_EXECUTABLE(mesh_cfgclient ${mesh_cfgclient_SOURCES})
        TARGET_LINK_LIBRARIES(mesh_cfgclient PRIVATE bluetooth-internal shared-ell)
        TARGET_LINK_LIBRARIES(mesh_cfgclient INTERFACE ${GLIB_LIBRARIES} ${DBUS_LIBRARIES} ${ell_ldadd} json-c readline)
        INSTALL(TARGET mesh_cfgclient RUNTIME DESTINATION bin)
    endif()
    
    set(EXTRA_DIST ${EXTRA_DIST} tools/mesh-gatt/local_node.json tools/mesh-gatt/prov_db.json)

    if (DEPRECATED)
        set(install_PROGRAMS hciattach hciconfig hcitool hcidump rfcomm sdptool ciptool)

        set(hciattach_SOURCES tools/hciattach.c tools/hciattach.h
                            tools/hciattach_st.c tools/hciattach_ti.c
                            tools/hciattach_tialt.c tools/hciattach_ath3k.c
                            tools/hciattach_qualcomm.c tools/hciattach_intel.c
                            tools/hciattach_bcm43xx.c)
        ADD_EXECUTABLE(hciattach ${hciattach_SOURCES})
        TARGET_LINK_LIBRARIES(hciattach PRIVATE bluetooth-internal)

        ADD_EXECUTABLE(hciconfig tools/hciconfig.c)
        TARGET_LINK_LIBRARIES(hciconfig PRIVATE bluetooth-internal)

        ADD_EXECUTABLE(hcitool tools/hcitool.c src/oui.h src/oui.c)
        TARGET_LINK_LIBRARIES(hcitool PRIVATE bluetooth-internal)
        TARGET_LINK_LIBRARIES(hcitool INTERFACE ${UDEV_LIBRARIES})

        ADD_EXECUTABLE(hcidump_SOURCES tools/hcidump.c
                        tools/parser/parser.h tools/parser/parser.c
                        tools/parser/lmp.c tools/parser/hci.c
                        tools/parser/l2cap.h tools/parser/l2cap.c
                        tools/parser/amp.c tools/parser/smp.c
                        tools/parser/att.c tools/parser/sdp.h tools/parser/sdp.c
                        tools/parser/rfcomm.h tools/parser/rfcomm.c
                        tools/parser/bnep.c tools/parser/cmtp.c
                        tools/parser/hidp.c tools/parser/hcrp.c
                        tools/parser/avdtp.c tools/parser/avctp.c
                        tools/parser/avrcp.c tools/parser/sap.c
                        tools/parser/obex.c tools/parser/capi.c
                        tools/parser/ppp.c tools/parser/tcpip.c
                        tools/parser/ericsson.c tools/parser/csr.c
                        tools/parser/bpa.c)
        TARGET_LINK_LIBRARIES(hcidump PRIVATE bluetooth-internal.la)

        ADD_EXECUTABLE(sdptool tools/sdptool.c src/sdp-xml.h src/sdp-xml.c)
        TARGET_LINK_LIBRARIES(sdptool PRIVATE bluetooth-internal)
        TARGET_LINK_LIBRARIES( $(GLIB_LIBS)

        ADD_EXECUTABLE(sdptool tools/ciptool.c)
        TARGET_LINK_LIBRARIES(ciptool PRIVATE bluetooth-internal.la)       

        TARGET_LINK_LIBRARIES(rfcomm PRIVATE bluetooth-internal.la

        set(dist_man_MANS ${dist_man_MANS} tools/hciattach.1 tools/hciconfig.1
                    tools/hcitool.1 tools/hcidump.1
                    tools/rfcomm.1 tools/sdptool.1 tools/ciptool.1)
    else()
        set(EXTRA_DIST ${EXTRA_DIST} tools/hciattach.1 tools/hciconfig.1
                    tools/hcitool.1 tools/hcidump.1 
                    tools/rfcomm.1 tools/sdptool.1 tools/ciptool.1)
    endif()

endif()



ADD_DEFINITIONS(${CFLAGS})
ADD_DEFINITIONS(-DVERSION="${PROJECT_VERSION}")
message(STATUS ">>>>>>>>>>>>>>>>>>>>> VERSION=${PROJECT_VERSION}")