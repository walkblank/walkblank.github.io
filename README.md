# walkblank.github.io

static char device_open(void)
{
        struct usb_bus *bus;
        struct usb_device *dev;
        usb_dev_handle* device_handle;

  static int USBInitDone = 0;
  int i, iret = 0, ret;

  if (USBInitDone != 0) return 0;

        // initialize USB subsystem
        usb_init();
        usb_set_debug(0);
        usb_find_busses();
        usb_find_devices();

        for (i=0; i<MAX_PORTNUM; ++i)
          usbhnd[i] = NULL;

        DEBUGP("libusb demo begin, qiushui_007 test ok\n");
        for (bus = usb_get_busses(); bus; bus = bus->next) {
                for (dev = bus->devices; dev; dev = dev->next) {
                        DEBUGP("%s/%s     %04X/%04X\n", bus->dirname, dev->filename,
                                                dev->descriptor.idVendor, dev->descriptor.idProduct);
                        if (dev->descriptor.idVendor == VENDOR_ID && dev->descriptor.idProduct == PRODUCT_ID) {
                                DEBUGP("Find device \n");

                                device_handle = usb_open(dev);
                                iret = 1;
                                if (device_handle == NULL) {
                                        iret = 0;
                                }
                                else {
                                        ret = 0;
                                        ret = usb_detach_kernel_driver_np(device_handle, 0);
                                        DEBUGP("usb_detach_kernel_driver_np: ret %d\n", ret);

                                        ret = usb_set_configuration(device_handle, 1);
                                        DEBUGP("usb_set_configuration: ret %d\n", ret);
                                        if (ret < 0) {
                                                 perror("Failed to set configuration\n");
                                                 usb_close(device_handle);
                                    device_handle = NULL;
                                        }

                /* 在操作设备前是必须调用的, 0是指用默认的设备 */
                ret = usb_claim_interface(device_handle, 0);
                DEBUGP("usb_claim_interface: ret %d\n", ret);
                      if (ret < 0) {
                        perror("Cannot claim interface");
                        usb_close(device_handle);
                        device_handle = NULL;

                        return 0;
                      }

                                        break;
                                }
                        }
                }
        }

        USBInitDone = 1 ;
        usbhnd[g_portnum] = device_handle;

        return iret;
}

执行结果:
//---- DS2490测试
usb_detach_kernel_driver_np: ret -61
usb_set_configuration: ret 0
usb_claim_interface: ret 0

//--- CR95HF测试
usb_detach_kernel_driver_np: ret -22
usb_set_configuration: ret 0
usb_claim_interface: ret -2
Cannot claim interface: No such file or directory
