if(CONFIG_OPENTHREAD_ENABLED)
    set(public_include_dirs
        "include"
        "openthread/include")

    set(private_include_dirs
        "openthread/examples/platforms"
        "openthread/include/openthread"
        "openthread/src"
        "openthread/src/core"
        "openthread/src/lib/hdlc"
        "openthread/src/lib/spinel"
        "openthread/src/ncp"
        "private_include")

        set(src_dirs
        "openthread/examples/platforms/utils"
        "openthread/src/core/api"
        "openthread/src/core/common"
        "openthread/src/core/crypto"
        "openthread/src/core/diags"
        "openthread/src/core/mac"
        "openthread/src/core/radio"
        "openthread/src/core/thread"
        "openthread/src/core/utils"
        "openthread/src/lib/hdlc"
        "openthread/src/lib/spinel")

    set(exclude_srcs
        "openthread/examples/platforms/utils/logging_rtt.c"
        "openthread/examples/platforms/utils/soft_source_match_table.c"
        "openthread/src/core/common/extension_example.cpp")

    if(CONFIG_OPENTHREAD_FTD OR CONFIG_OPENTHREAD_MTD)
        list(APPEND src_dirs
            "openthread/examples/apps/cli"
            "openthread/src/core/backbone_router"
            "openthread/src/core/coap"
            "openthread/src/core/meshcop"
            "openthread/src/core/net"
            "openthread/src/cli"
            "openthread/src/lib/platform")

        list(APPEND exclude_srcs
            "openthread/examples/apps/cli/main.c")

    elseif(CONFIG_OPENTHREAD_RADIO)
        list(APPEND src_dirs
            "openthread/src/ncp"
            "openthread/examples/apps/ncp")

        list(APPEND exclude_srcs
            "openthread/examples/apps/ncp/main.c"
            "openthread/src/core/api/backbone_router_api.cpp"
            "openthread/src/core/api/entropy_api.cpp"
            "openthread/src/core/api/netdata_api.cpp"
            "openthread/src/core/api/link_api.cpp"
            "openthread/src/core/api/icmp6_api.cpp"
            "openthread/src/core/api/ip6_api.cpp"
            "openthread/src/core/api/udp_api.cpp"
            "openthread/src/core/common/settings.cpp"
            "openthread/src/core/api/dataset_api.cpp"
            "openthread/src/core/api/random_crypto_api.cpp"
            "openthread/src/core/common/heap_string.cpp"
            "openthread/src/core/common/time_ticker.cpp"
            "openthread/src/core/common/notifier.cpp"
            "openthread/src/core/api/dns_api.cpp"
            "openthread/src/core/api/message_api.cpp"
            "openthread/src/core/api/tcp_api.cpp"
            "openthread/src/core/thread/energy_scan_server.cpp"
            "openthread/src/core/thread/mesh_forwarder.cpp"
            "openthread/src/core/thread/mle.cpp"
            "openthread/src/core/thread/neighbor_table.cpp"
            "openthread/src/core/thread/tmf.cpp"
            "openthread/src/core/thread/network_data_leader_ftd.cpp"
            "openthread/src/core/thread/discover_scanner.cpp"
            "openthread/src/core/thread/lowpan.cpp"
            "openthread/src/core/thread/network_data_leader.cpp"
            "openthread/src/core/thread/mesh_forwarder_mtd.cpp"
            "openthread/src/core/thread/key_manager.cpp"
            "openthread/src/core/thread/network_data_types.cpp"
            "openthread/src/core/thread/mesh_forwarder_ftd.cpp"
            "openthread/src/core/thread/network_data.cpp"
            "openthread/src/core/thread/network_data_service.cpp"
            "openthread/src/core/thread/announce_begin_server.cpp"
            "openthread/src/core/thread/announce_sender.cpp"
            "openthread/src/core/thread/thread_netif.cpp"
            "openthread/src/core/thread/topology.cpp"
            "openthread/src/core/thread/panid_query_server.cpp"
            "openthread/src/core/mac/channel_mask.cpp"
            "openthread/src/core/mac/data_poll_handler.cpp"
            "openthread/src/core/mac/data_poll_sender.cpp"
            "openthread/src/core/mac/mac.cpp"
            "openthread/src/core/mac/mac_filter.cpp"
            "openthread/src/core/mac/mac_links.cpp"
            "openthread/src/core/utils/child_supervision.cpp")
    endif()

    if(CONFIG_OPENTHREAD_BORDER_ROUTER)
        list(APPEND src_dirs
            "openthread/src/core/border_router")
    endif()

    if(CONFIG_OPENTHREAD_FTD)
        set_source_files_properties("openthread/src/core/net/srp_server.cpp"
            PROPERTIES COMPILE_FLAGS
            -Wno-maybe-uninitialized)
    endif()

    if(CONFIG_OPENTHREAD_FTD)
        set(device_type "OPENTHREAD_FTD=1")
    elseif(CONFIG_OPENTHREAD_MTD)
        set(device_type "OPENTHREAD_MTD=1")
    elseif(CONFIG_OPENTHREAD_RADIO)
        set(device_type "OPENTHREAD_RADIO=1")
    endif()

endif()

execute_process(
    COMMAND git rev-parse --short HEAD
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    OUTPUT_VARIABLE IDF_VERSION_FOR_OPENTHREAD_PACKAGE OUTPUT_STRIP_TRAILING_WHITESPACE
)

execute_process(
    COMMAND git rev-parse --short HEAD
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/openthread
    OUTPUT_VARIABLE OPENTHREAD_VERSION OUTPUT_STRIP_TRAILING_WHITESPACE
)

idf_component_register(SRC_DIRS "${src_dirs}"
                       EXCLUDE_SRCS "${exclude_srcs}"
                       INCLUDE_DIRS "${public_include_dirs}"
                       PRIV_INCLUDE_DIRS "${private_include_dirs}"
                       REQUIRES mbedtls ieee802154 console)

if(CONFIG_OPENTHREAD_ENABLED)
    if(CONFIG_OPENTHREAD_RADIO)
        set(CONFIG_FILE_TYPE "radio")
    elseif(CONFIG_OPENTHREAD_FTD)
        set(CONFIG_FILE_TYPE "ftd")
    elseif(CONFIG_OPENTHREAD_MTD)
        set(CONFIG_FILE_TYPE "mtd")
    endif()

    target_compile_definitions(
        ${COMPONENT_LIB}
        PUBLIC
        "OPENTHREAD_CONFIG_FILE=\"openthread-core-esp32x-${CONFIG_FILE_TYPE}-config.h\""
            "${device_type}"
        PRIVATE
        "PACKAGE_VERSION=\"${IDF_VERSION_FOR_OPENTHREAD_PACKAGE}-${OPENTHREAD_VERSION}\"")

    if(CONFIG_OPENTHREAD_ESP_LIB_FROM_INTERNAL_SRC)
        idf_component_get_property(openthread_port_lib openthread_port COMPONENT_LIB)
        idf_component_get_property(esp_system_lib esp_system COMPONENT_LIB)
        target_link_libraries(${COMPONENT_LIB} PUBLIC
                              $<TARGET_FILE:${openthread_port_lib}>
                              $<TARGET_FILE:${esp_system_lib}>)

        if(CONFIG_OPENTHREAD_BORDER_ROUTER)
            idf_component_get_property(openthread_br_lib openthread_br COMPONENT_LIB)
            target_link_libraries(${COMPONENT_LIB} PUBLIC $<TARGET_FILE:${openthread_br_lib}>)
        endif()

    else()
        add_prebuilt_library(openthread_port "${CMAKE_CURRENT_SOURCE_DIR}/lib/${idf_target}/libopenthread_port.a"
                            REQUIRES openthread)
        add_prebuilt_library(openthread_br "${CMAKE_CURRENT_SOURCE_DIR}/lib/${idf_target}/libopenthread_br.a"
                            REQUIRES openthread)

        target_link_libraries(${COMPONENT_LIB} INTERFACE openthread_port)

        if(CONFIG_OPENTHREAD_BORDER_ROUTER)
            target_link_libraries(${COMPONENT_LIB} INTERFACE openthread_br)
        endif()

    endif()

endif()
