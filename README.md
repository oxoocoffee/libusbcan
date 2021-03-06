# libusbcan

libusbcan provides a simple, pure C API for the [ViewTool Ginkgo USB-CAN](http://www.viewtool.com/index.php?option=com_content&view=article&id=201&Itemid=27) interfaces. It also adds functionality absent from the standard Viewtool API. The most significant are callbacks for received messages are per bus rather than per device and callbacks can have an argument associated with them. Where possible, the Linux SocketCAN data structures are used.

ViewTool only distributes binaries for their USB-CAN/Ginkgo libraries, so platform support is limited to the 4 they
provide: OS X, Linux x86, Linux x86_64, and Raspberry Pi. Building libusbcan requires [CMake](http://cmake.org) 3.0.1
or later. Once you have CMake installed, just

	mkdir build
	cd build
	cmake ../
	make

This will build the library and the example utilities.

# Library lifecycle

libusbcan does some internal bookkeeping that must be performed explicitly before and after it is used.

	bool usbcan_library_init();
	bool usbcan_library_close();

# Application lifecycle

	bool usbcan_init(uint32_t dev, uint32_t bus, struct usbcan_bus_config *config);

	struct usbcan_bus_config {
		uint32_t           speed;
		struct can_filter *filters;
		uint8_t            num_filters;
		usbcan_cb          cb;
		void              *arg;
	};

	CAN_SPEED_1000KBPS
	CAN_SPEED_500KBPS
	CAN_SPEED_250KBPS
	CAN_SPEED_125KBPS
	CAN_SPEED_100KBPS
	CAN_SPEED_83KBPS
	CAN_SPEED_50KBPS
	CAN_SPEED_20KBPS
	CAN_SPEED_10KBPS
	
	bool usbcan_start(uint32_t dev, uint32_t bus);
	bool usbcan_reset(uint32_t dev, uint32_t bus);
	bool usbcan_stop(uint32_t dev, uint32_t bus);

# Sending and receiving messages

	struct usbcan_msg {
		uint32_t timestamp;
		struct can_frame frame;
	};

	void(*usbcan_cb)(uint32_t dev, uint32_t bus, struct usbcan_msg *msgs, uint32_t n, void *arg);

	uint32_t usbcan_send(uint32_t dev, uint32_t bus, struct can_frame *frame);
	uint32_t usbcan_send_n(uint32_t dev, uint32_t bus, struct can_frame *frames, uint32_t n);

# Example

    #include <unistd.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <stdint.h>
    #include <signal.h>
	
    #include "usbcan.h"
	
	void usbcandump_exit_handler(int signal) {
		usbcan_library_close();

	    exit(0);
	}
	
	void usbcandump_callback(uint32_t dev, uint32_t bus, struct usbcan_msg *msgs, uint32_t n, void *arg) {
		for (int i = 0; i < len; i++) {
			printf(" usbcan%i:%i  %X   [%i] ", dev, bus, msgs[i].frame.can_id, msgs[i].frame.can_dlc);
			for(int j = 0; j < 8; j++)
			{
				printf(" %02X", msgs[i].frame.data[j]);
			}
			printf("\n");
		}
	}

	int main(void) {
		struct sigaction int_act;
		int_act.sa_handler = usbcandump_exit_handler;
		sigaction(SIGINT, &int_act, NULL);

	    setbuf(stdout, NULL);
	
		if (!usbcan_library_init()) {
			exit(-1);
		}
		
		struct usbcan_bus_config config;
		config.speed = CAN_SPEED_500KBPS;
		config.filters = NULL;
		config.num_filters = 0;
		config.cb = usbcandump_callback;
		config.arg = NULL;
		
		if (!usbcan_init(0, CAN1, &config)) {
			exit(-1);
		}
		
		if (!usbcan_start(0, CAN1)) {
			exit(-1);
		}
		
		pause();
		
		return 0;
	}



