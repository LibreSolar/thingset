# ThingSet Zephyr SDK

The ThingSet software development (SDK) for Zephyr leverages the Zephyr RTOS APIs to provide ready-to-use implementations for the following interfaces:

- Serial (UART or USB CDC-ACM)
- CAN
- Bluetooth
- LoRaWAN
- Flash or EEPROM (for non-volatile data storage)

Further interfaces like MQTT and WebSocket over WiFi are currently under development.

The ThingSet device library is used internally for data processing.

**Source repository:** [ThingSet/thingset-zephyr-sdk](https://github.com/ThingSet/thingset-zephyr-sdk)

## Example

First of all, the ThingSet library and an interface provided by the ThingSet SDK have to be enabled via Kconfig in the  `prj.conf`. This example uses the serial interface:

```sh
CONFIG_SERIAL=y
CONFIG_UART_INTERRUPT_DRIVEN=y

CONFIG_THINGSET=y
CONFIG_THINGSET_SDK=y
CONFIG_THINGSET_SERIAL=y
```

Afterwards, exposing data via ThingSet just needs very few lines of code:

```c
#include <thingset.h>
#include <thingset/sdk.h>
#include <zephyr/kernel.h>

/* local variables/functions to be exposed via ThingSet */
float room_temp = 18.3;
float target_temp = 21.0;
bool heater_on = true;
void reset(void);

/* ThingSet object definitions */
THINGSET_ADD_ITEM_FLOAT(ID_ROOT, 0x61, "rRoomTemp_degC", &room_temp, 1, THINGSET_ANY_R, 0);
THINGSET_ADD_ITEM_FLOAT(ID_ROOT, 0x62, "wTargetTemp_degC", &target_temp, 1, THINGSET_ANY_RW, 0);
THINGSET_ADD_ITEM_BOOL(ID_ROOT, 0x71, "rHeaterOn", &heater_on, THINGSET_ANY_R, 0);
THINGSET_ADD_FN_VOID(ID_ROOT, 0xE1, "xReset", &reset, THINGSET_ANY_RW);

void reset(void)
{
    printf("Reset function called!\n");
}

int main(void)
{
    /*
     * Implement your application code here (e.g. controlling of the heater).
     *
     * ThingSet requests are automatically handled by the SDK in the background.
     */

    return 0;
}
```

The `THINGSET_ADD_*` macros use Zephyr's [Iterable Sections](https://docs.zephyrproject.org/latest/kernel/iterable_sections/index.html) internally. They can be used anywhere in the code to add data items to the global ThingSet database.

The actual processing of the data through the serial interface happens in a dedicated thread in the background. You can fully focus on your application development.

If the used board supports Bluetooth, a simple `CONFIG_THINGSET_BLE=y` would also make the data available via Bluetooth (in addition to the serial interface).
