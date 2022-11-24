
include_directories(./tinyusb/src)
AUX_SOURCE_DIRECTORY(./tinyusb/src DIR_TUSB_SRCS)
AUX_SOURCE_DIRECTORY(./tinyusb/src/common DIR_TUSB_SRCS)
AUX_SOURCE_DIRECTORY(./tinyusb/src/device DIR_TUSB_SRCS)
AUX_SOURCE_DIRECTORY(./tinyusb/src/class/cdc DIR_TUSB_SRCS)
AUX_SOURCE_DIRECTORY(./tinyusb/src/class/msc DIR_TUSB_SRCS)


include_directories(./dhara/dhara)
AUX_SOURCE_DIRECTORY(./dhara/dhara DIR_DHARA_SRCS)




ADD_LIBRARY(tusb ${DIR_TUSB_SRCS}) 
ADD_LIBRARY(dhara ${DIR_DHARA_SRCS}) 



