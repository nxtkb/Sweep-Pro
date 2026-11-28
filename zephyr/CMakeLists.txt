if("$ENV{GITHUB_REF_TYPE}" STREQUAL "tag"
   AND NOT CONFIG_ZMK_SETTINGS_RESET_ON_START
   AND NOT CONFIG_NXTKB_ADAFRUIT_BLE_DFU)
  message(FATAL_ERROR "Release firmware must include the Bluetooth DFU entry")
endif()
