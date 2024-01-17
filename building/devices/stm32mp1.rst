.. _stm32mp1:

########
STM32MP1
########

The instructions here will tell how to run OP-TEE on one of the supported
STM32MP1 boards.

Supported boards
****************

+---------------------+--------------------+------------+-------------------------------+
| Board Name          | Manufacturer       | Boot media | Hardware Description          |
+=====================+====================+============+===============================+
| `STM32MP135F-DK`_   | STMicroelectronics | SDcard     | `Wiki STM32MP135x-DK`_        |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157A-DK1`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157D-DK1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-DK2`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-DK2`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-EV1`_  | STMicroelectronics | SDCard (1) | `Wiki STM32MP157x-EV1`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-EV1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+

(1): STM32MP157x-EV1 boards also integrate an eMMC device, a NOR flash and a
Nand flash the system can boot on. OP-TEE distribution however only supports
booting from the SDcard slot.

.. _stm32mp1_build_instructions:

Build instructions
******************

Follow the instructions at ":ref:`get_and_build_the_solution`".

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------------------+--------------------------------------+
| Board Name             | Build configuration directive        |
+========================+======================================+
| `STM32MP135F-DK`_      | ``PLATFORM=stm32mp1-135F_DK``        |
+------------------------+--------------------------------------+
| `STM32MP157A-DK1`_     | ``PLATFORM=stm32mp1-157A_DK1_SCMI``  |
| `STM32MP157D-DK1`_     |                                      |
+------------------------+--------------------------------------+
| `STM32MP157C-DK2`_     | ``PLATFORM=stm32mp1-157C_DK2_SCMI``  |
| `STM32MP157F-DK2`_     |                                      |
+------------------------+--------------------------------------+
| `STM32MP157C-EV1`_     | ``PLATFORM=stm32mp1-157C_EV1_SCMI``  |
| `STM32MP157F-EV1`_     |                                      |
+------------------------+--------------------------------------+

For compatibility with existing Linux kernel and U-Boot boards (DTS files),
STM32MP15 flavors can also be built with RCC secure hardening disabled
in which case some SoC main resources (clock and reset controllers) are
assigned to non-secure world. These are listed in table below.

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------------------+--------------------------------------+
| Board Name             | Build configuration directive        |
+========================+======================================+
| `STM32MP157A-DK1`_     | ``PLATFORM=stm32mp1-157A_DK1``       |
| `STM32MP157D-DK1`_     |                                      |
+------------------------+--------------------------------------+
| `STM32MP157C-DK2`_     | ``PLATFORM=stm32mp1-157C_DK2``       |
| `STM32MP157F-DK2`_     |                                      |
+------------------------+--------------------------------------+
| `STM32MP157C-EV1`_     | ``PLATFORM=stm32mp1-157C_EV1``       |
| `STM32MP157F-EV1`_     |                                      |
+------------------------+--------------------------------------+

When the build completes, generated image file sdcard.img can be found
in the generated binary images directory ``../out/bin/`` from build
root path. The images is a GPT multipartition image you can raw copy
to the target SDcard using a tool like dd.

A usual short fecth/build/load shell sequence is like the one below:

.. code-block:: bash

  $ repo init -u https://github.com/OP-TEE/manifest.git -m stm32mp1.xml
  $ repo sync
  $ cd build
  $ make toolchains
  $ make PLATFORM=stm32mp1-157C_DK2 all
  $ dd if=../out/bin/sdcard.img of=/dev/sdX conv=fdatasync status=progress
  $ sgdisk -e /dev/sdX

Command ``sgdisk -e`` fixes the GPT backup data which location depends on
storage device effective size.

Build directives
****************

The main and mandatory build directive is ``PLATFORM`` value that states the
platform and possibly its flavor. Table above, in section
:ref:`stm32mp1_build_instructions`, lists value for ``PLATFORM`` supported
by `stm32mp1.mk`_ makefile.

STM32MP15 RCC security state
============================

One can note that there are 2 series of configurations for STM32MP15
variants: with RCC security hardening enable (e.g.
``PLATFORM=stm32mp1-157C_EV1_SCMI``) or disabled (e.g.
``PLATFORM=stm32mp1-157C_EV1``).

The STM32MP15 boards configurations with RCC security hardening enable
assigns some main clock and reset controllers to the secure world.
These configurations comply with U-Boot and Linux stm32mp157*-scmi.dts
DTS board files where SCMI protocols are used to expose some secure
SoC resources to non-secure world. Such a configuration is required
when assigning SoC peripherals (crypto engines, hardware busses, etc...)
to the secure world.

Note that RCC secure hardening is always enabled on STM32MP13 platform
flavors (``PLATFORM=stm32mp1-135F_DK``).

Standard build directive
=========================

Any ``CFG_*`` OP-TEE configuration directives can be passed to the build
sequence as part of the make command line arguments. For example, one can
override OP-TEE core log level with:

.. code-block:: bash

  $ make PLATFORM=stm32mp1-157C_DK2_SCMI CFG_TEE_CORE_LOG_LEVEL=0 all

Any ``BR2_*`` Buildroot configuration directive can be passed to the
build sequence as part of the make command line argument. For example,
one can enable Buildroot busybox watchdog support with:

.. code-block:: bash

  $ make PLATFORM=stm32mp1-157C_DK2_SCMI BR2_PACKAGE_BUSYBOX_WATCHDOG=y all

``WITH_RPMB_TEST`` build directive
==================================

OP-TEE distribution build script for stm32mp1 platforms supports
configuration directive ``WITH_RPMB_TEST=y|n`` (default ``n``).

When enabled, the platforms embeds OP-TEE's RPMB_FS secure storage
(``CFG_RPMB_FS=y``) with test configuration directove (``CFG_RPMB_TESTKEY=y``
and ``CFG_REE_FS_ALLOW_RESET=y``).

This configuration can only be used with boards that have a physical RPMB
provisioned with OP-TEE RPMB test key.

``WITH_SRAM1_PAGER_POOL`` build directive
=========================================

OP-TEE distribution build script for stm32mp1 platforms supports
configuration directive ``WITH_SRAM1_PAGER_POOL=y|n``.

The configuration only relates the STM32MP15 boards when the platform
embed OP-TEE pager support. When enabled, SRAM1 internal RAM is assigned
to OP-TEE secure firmware and used a page pool by OP-TEE pager.

``WITH_SRAM1_PAGER_POOL`` is default enabled for platform flavors
``PLATFORM=stm32mp1-157C_*_SCMI`` and default disabled for other flavors.

.. _STM32MP135F-DK: https://www.st.com/en/evaluation-tools/stm32mp135f-dk.html
.. _STM32MP157A-DK1: https://www.st.com/en/evaluation-tools/stm32mp157a-dk1.html
.. _STM32MP157D-DK1: https://www.st.com/en/evaluation-tools/stm32mp157d-dk1.html
.. _STM32MP157C-DK2: https://www.st.com/en/evaluation-tools/stm32mp157c-dk2.html
.. _STM32MP157F-DK2: https://www.st.com/en/evaluation-tools/stm32mp157f-dk2.html
.. _STM32MP157C-EV1: https://www.st.com/en/evaluation-tools/stm32mp157c-ev1.html
.. _STM32MP157F-EV1: https://www.st.com/en/evaluation-tools/stm32mp157f-ev1.html
.. _Wiki STM32MP135x-DK: https://wiki.st.com/stm32mpu/wiki/STM32MP135x-DK_-_hardware_description
.. _Wiki STM32MP157x-DKx: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-DKx_-_hardware_description
.. _Wiki STM32MP157x-EV1: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-EV1_-_hardware_description
.. _stm32mp1.mk: https://github.com/OP-TEE/build/blob/master/stm32mp1.mk
