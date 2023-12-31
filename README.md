| Supported Targets | ESP32 | ESP32-C2 | ESP32-C3 | ESP32-C6 | ESP32-H2 | ESP32-S2 | ESP32-S3 |
| ----------------- | ----- | -------- | -------- | -------- | -------- | -------- | -------- |

# Add support for [星球一号 LVGL 开发板](https://item.taobao.com/item.htm?id=716563448599) 
Including below changes:  
1) Add component esp_lcd_st7796 to support spi LCD controller ST7796
2) Add component esp_lcd_touch_ft5x06 to support i2c touch controller FT6236 
3) Config example:   
<pre>(Top) → Example Configuration
                                   Espressif IoT Development Framework Configuration
    LCD controller model (ST7796)  --->
[*] Enable LCD touch
        LCD touch controller model (FT5X06)  ---></pre>
![idf.py menuconfig](config.png "idf.py menuconfig")

# One more thing
It's better to increase lv memory size if below error occurred:
<pre>
E (11360) task_wdt: Task watchdog got triggered. The following tasks/users did not reset the watchdog in time:
E (11360) task_wdt:  - IDLE (CPU 0)
E (11360) task_wdt: Tasks currently running:
E (11360) task_wdt: CPU 0: main
E (11360) task_wdt: CPU 1: IDLE
E (11360) task_wdt: Print CPU 0 (current core) backtrace
</pre>
I just increase it from 32k to 64k to solve task watchdog issue:
<pre>
(Top) → Component config → LVGL configuration → Memory settings
                                                                          Espressif IoT Development Framework Configuration
[ ] If true use custom malloc/free, otherwise use the built-in `lv_mem_alloc()` and `lv_mem_free()`
(64) Size of the memory used by `lv_mem_alloc` in kilobytes (>= 2kB)
(0x0) Address for the memory pool instead of allocating it as a normal array
(16) Number of the memory buffer
[ ] Use the standard memcpy and memset instead of LVGL's own functions
</pre>
![CONFIG_LV_MEM_SIZE_KILOBYTES](CONFIG_LV_MEM_SIZE_KILOBYTES.png "CONFIG_LV_MEM_SIZE_KILOBYTES")


# SPI LCD and Touch Panel Example

[esp_lcd](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/lcd.html) provides several panel drivers out-of box, e.g. ST7789, SSD1306, NT35510. However, there're a lot of other panels on the market, it's beyond `esp_lcd` component's responsibility to include them all.

`esp_lcd` allows user to add their own panel drivers in the project scope (i.e. panel driver can live outside of esp-idf), so that the upper layer code like LVGL porting code can be reused without any modifications, as long as user-implemented panel driver follows the interface defined in the `esp_lcd` component.

This example shows how to use GC9A01 or ILI9341 display driver from Component manager in esp-idf project. These components are using API provided by `esp_lcd` component. This example will draw a fancy dash board with the LVGL library. For more information about porting the LVGL library, you can also refer to [another lvgl porting example](../i80_controller/README.md).

## Touch controller STMPE610

In this example you can enable touch controller STMPE610 connected via SPI. The SPI connection is shared with LCD screen.

## How to use the example

### Hardware Required

* An ESP development board
* An GC9A01 or ILI9341 LCD panel, with SPI interface (with/without STMPE610 SPI touch)
* An USB cable for power supply and programming

### Hardware Connection

The connection between ESP Board and the LCD is as follows:

```
       ESP Board                       GC9A01/ILI9341 Panel + TOUCH
┌──────────────────────┐              ┌────────────────────┐
│             GND      ├─────────────►│ GND                │
│                      │              │                    │
│             3V3      ├─────────────►│ VCC                │
│                      │              │                    │
│             PCLK     ├─────────────►│ SCL                │
│                      │              │                    │
│             MOSI     ├─────────────►│ MOSI               │
│                      │              │                    │
│             MISO     |◄─────────────┤ MISO               │
│                      │              │                    │
│             RST      ├─────────────►│ RES                │
│                      │              │                    │
│             DC       ├─────────────►│ DC                 │
│                      │              │                    │
│             LCD CS   ├─────────────►│ LCD CS             │
│                      │              │                    │
│             TOUCH CS ├─────────────►│ TOUCH CS           │
│                      │              │                    │
│             BK_LIGHT ├─────────────►│ BLK                │
└──────────────────────┘              └────────────────────┘
```

The GPIO number used by this example can be changed in [lvgl_example_main.c](main/spi_lcd_touch_example_main.c).
Especially, please pay attention to the level used to turn on the LCD backlight, some LCD module needs a low level to turn it on, while others take a high level. You can change the backlight level macro `EXAMPLE_LCD_BK_LIGHT_ON_LEVEL` in [lvgl_example_main.c](main/spi_lcd_touch_example_main.c).

### Build and Flash

Run `idf.py -p PORT build flash monitor` to build, flash and monitor the project. A fancy animation will show up on the LCD as expected.

The first time you run `idf.py` for the example will cost extra time as the build system needs to address the component dependencies and downloads the missing components from registry into `managed_components` folder.

(To exit the serial monitor, type ``Ctrl-]``.)

See the [Getting Started Guide](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html) for full steps to configure and use ESP-IDF to build projects.

### Example Output

```bash
...
I (409) cpu_start: Starting scheduler on APP CPU.
I (419) example: Turn off LCD backlight
I (419) gpio: GPIO[2]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (429) example: Initialize SPI bus
I (439) example: Install panel IO
I (439) gpio: GPIO[5]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (449) example: Install GC9A01 panel driver
I (459) gpio: GPIO[3]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (589) gpio: GPIO[0]| InputEn: 0| OutputEn: 1| OpenDrain: 0| Pullup: 0| Pulldown: 0| Intr:0
I (589) example: Initialize touch controller STMPE610
I (589) STMPE610: TouchPad ID: 0x0811
I (589) STMPE610: TouchPad Ver: 0x03
I (599) example: Turn on LCD backlight
I (599) example: Initialize LVGL library
I (609) example: Register display driver to LVGL
I (619) example: Install LVGL tick timer
I (619) example: Display LVGL Meter Widget
...
```


## Troubleshooting

* Why the LCD doesn't light up?
  * Check the backlight's turn-on level, and update it in `EXAMPLE_LCD_BK_LIGHT_ON_LEVEL`

For any technical queries, please open an [issue] (https://github.com/espressif/esp-idf/issues) on GitHub. We will get back to you soon.
