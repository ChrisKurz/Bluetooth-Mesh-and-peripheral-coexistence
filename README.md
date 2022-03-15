# Bluetooth: Mesh and peripheral coexistence

.. contents::
   :local:
   :depth: 2

This sample demonstrates how to combine Bluetooth® mesh and Bluetooth Low Energy features in a single application.

## Requirements

The mesh and peripheral coexistence sample supports the following development kits:

| Hardware platforms  |
|---------------------|
| nrf52840dk_nrf52840 |
| nrf52dk_nrf52832    |
| nrf52833dk_nrf52833 |
| nrf52833dk_nrf52820 |
| nrf21540dk_nrf52840 |

The sample also requires a smartphone with `nRF Connect for Mobile`and one of the following apps:

* [nRF Mesh mobile app for Android](https://play.google.com/store/apps/details?id=no.nordicsemi.android.nrfmeshprovisioner)
* [nRF Mesh mobile app for iOS](https://apps.apple.com/us/app/nrf-mesh/id1380726771)

## Overview

The purpose of this sample is to showcase an application where a peripheral can operate independently of Bluetooth mesh.
It combines the features of the [Bluetooth: Peripheral LBS](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/samples/bluetooth/peripheral_lbs/README.html) sample and the [Bluetooth: Mesh light](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/samples/bluetooth/mesh/light/README.html) sample into a single application.

The [Bluetooth: Peripheral LBS](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/samples/bluetooth/peripheral_lbs/README.html) controls the state of a LED and monitors the state of a button on the device.
Bluetooth mesh controls and monitors the state of a separate LED on the device.
The LBS and Bluetooth mesh do not share any states on the device and operate independently of each other.

To be truly independent, the LBS must be able to advertise its presence independently of Bluetooth mesh.
Because Bluetooth mesh uses the advertiser as its main channel of communication, you must configure the application to allow sharing of this resource.
To achieve this, extended advertising is enabled with two simultaneous advertising sets.
One of these sets is used to handle all Bluetooth mesh communication, while the other is used for advertising the LBS.

__note::
   Extended advertising is a requirement for achieving multiple advertisers in this sample.__

### LED Button Service

When connected, the [Bluetooth: Peripheral LBS](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/samples/bluetooth/peripheral_lbs/README.html) sends the state of **Button 1** on the development kit to the connected device, such as a phone or tablet.
The mobile application on the device can display the received button state and control the state of **LED 2** on the development kit.

### Mesh provisioning

The provisioning is handled by the [Bluetooth mesh provisioning handler for Nordic DKs](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/1.9.0/nrf/libraries/bluetooth_services/mesh/dk_prov.html?highlight=bt_mesh_dk_prov).
It supports four types of out-of-band (OOB) authentication methods, and uses the Hardware Information driver to generate a deterministic UUID to uniquely represent the device.

### Mesh models

The following table shows the mesh light composition data for this sample:

   |  Element 1          |
   |---------------------|
   | Config Server       |
   | Health Server       |
   | Gen. OnOff Server   |

The models are used for the following purposes:

* [bt_mesh_onoff_srv](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/bluetooth_services/mesh/gen_onoff_srv.html?highlight=bt_mesh_onoff_srv#c.bt_mesh_onoff_srv) instance in element 1 controls **LED 1**.
* Config Server allows configurator devices to configure the node remotely.
* Health Server provides ``attention`` callbacks that are used during provisioning to call your attention to the device.
  These callbacks trigger blinking of the LEDs.

The model handling is implemented in [src/model_handler.c](src/model_handler.c), which uses the [DK Buttons and Leds](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/others/dk_buttons_and_leds.html?highlight=dk_buttons_and_leds) library to control each LED on the development kit according to the matching received messages of Generic OnOff Server.

## User interface

Buttons (Common):
   Can be used to input the OOB authentication value during provisioning.
   All buttons have the same functionality during this procedure.

Button 1:
   Send a notification through the LED Button Service with the button pressed or released state.

LEDs (Common):
   Show the OOB authentication value during provisioning if the Push button OOB method is used.

LED 1:
   Controlled by the Generic OnOff Server.

LED 2:
   Controlled remotely by the LED Button Service from the connected device.

## Configuration

See [Configuring your application](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/gs_modifying.html#configure-application) for information about how to permanently or temporarily change the configuration.

### Source file setup

This sample is split into the following source files:

* [main.c](src/main.c) used to handle initialization.
* [model_handler.c](src/model_handler.c) used to handle mesh models.
* [lb_service_handler.c](src/lb_service_hander.c) used to handle the LBS interaction.

### [FEM support](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/samples/bluetooth/mesh/light/README.html#fem-support)

You can add support for the nRF21540 front-end module to this sample by using one of the following options, depending on your hardware:

Build the sample for one board that contains the nRF21540 FEM, such as nrf21540dk_nrf52840.

Manually create a devicetree overlay file that describes how FEM is connected to the nRF5 SoC in your device. See Set devicetree overlays for different ways of adding the overlay file.

Provide nRF21540 FEM capabilities by using a shield, for example the nRF21540 EK shield that is available in the nRF Connect SDK. In this case, build the project for a board connected to the shield you are using with an appropriate variable included in the build command. This variable instructs the build system to append the appropriate devicetree overlay file. For example, to build the sample from the command line for an nRF52833 DK with the nRF21540 EK attached, use the following command within the sample directory:

        west build -b nrf52833dk_nrf52833 -- -DSHIELD=nrf21540_ek
This command builds the application firmware. See Programming nRF21540 EK for information about how to program when you are using a board with a network core, for example nRF5340 DK.

Each of these options adds the description of the nRF21540 FEM to the devicetree. See Working with RF front-end modules for more information about FEM in the nRF Connect SDK.

To add support for other front-end modules, add the respective devicetree file entries to the board devicetree file or the devicetree overlay file.

## Building and running

This sample can be found under `samples/bluetooth/mesh/ble_peripheral_lbs_coex` in the nRF Connect SDK folder structure.

See [Building and programming an application](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/gs_programming.html#gs-programming) for information about how to build and program the application.


### Testing Generic OnOff Server (Bluetooth mesh)

After programming the sample to your development kit, you can test it by using a smartphone with `nRF Mesh mobile app`_ installed.
Testing consists of provisioning the device and configuring it for communication with the Bluetooth mesh models.

#### Provisioning the device

The provisioning assigns an address range to the device, and adds it to the mesh network. Complete the following steps in the nRF Mesh app:

1. Tap Add node to start scanning for unprovisioned mesh devices.
2. Select the Mesh Light device to connect to it.
3. Tap Identify, and then Provision, to provision the device.
4. When prompted, select an OOB method and follow the instructions in the app.

Once the provisioning is complete, the app returns to the Network screen.

#### Configuring models

See :ref:`ug_bt_mesh_model_config_app` for details on how to configure the mesh models with the nRF Mesh mobile app.

Configure the Generic OnOff Server model on the root element of the :guilabel:`Mesh and Peripheral Coex` node:

1. Bind the model to `Application Key 1`.
   Once the model is bound to the application key, you can control **LED 1** on the device.

2. In the model view, tap `ON`/`OFF` (one of the Generic On Off Controls).
   This switches the **LED 1** on the development kit on and off respectively.

### Testing LED Button Service (peripheral)

After programming the sample to your development kit, test it by performing the following steps:

1. Start the `nRF Connect for Mobile` application on your smartphone or tablet.
2. Power on the development kit.
3. Connect to the device from the nRF Connect application.
   The device is advertising as "Mesh and Peripheral Coex".
   The services of the connected device are shown.
4. In :`Nordic LED Button Service`, enable notifications for the `Button` characteristic.
5. Press **Button 1** on the device.
6. Observe that notifications with the following values are displayed:

   * ``Button released`` when **Button 1** is released.
   * ``Button pressed`` when **Button 1** is pressed.

7. Write the following values to the LED characteristic in the `Nordic LED Button Service`:

   * Value ``OFF`` to switch the **LED 2** on the development kit off.
   * Value ``ON`` to switch the **LED 2** on the development kit on.

## Dependencies

This sample uses the following nRF Connect SDK libraries:

* [Generic OnOff Server](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/bluetooth_services/mesh/gen_onoff_srv.html#bt-mesh-onoff-srv-readme)
* [Bluetooth Mesh provisioning handler for Nordic DKs](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/bluetooth_services/mesh/dk_prov.html#bt-mesh-dk-prov)
* [LED Button Service (LBS)](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/bluetooth_services/services/lbs.html#led-button-service-lbs)
* [DK Button and LEDs](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/libraries/others/dk_buttons_and_leds.html#dk-buttons-and-leds-readme)

In addition, it uses the following Zephyr libraries:

* ``include/drivers/hwinfo.h``
* ``include/zephyr/types.h``
* ``lib/libc/minimal/include/errno.h``
* ``include/sys/printk.h``
* ``include/sys/byteorder.h``
* [Kernel Services](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/zephyr/reference/kernel/index.html#kernel-api):

  * ``include/kernel.h``

* [Bluetooth](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/zephyr/reference/bluetooth/index.html#bluetooth-api):

  * ``include/bluetooth/bluetooth.h``
  * ``include/bluetooth/hci.h``
  * ``include/bluetooth/conn.h``
  * ``include/bluetooth/uuid.h``
  * ``include/bluetooth/gatt.h``

* [Bluetooth Mesh Profile](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/zephyr/reference/bluetooth/mesh.html#bluetooth-mesh):

  * ``include/bluetooth/mesh.h``
