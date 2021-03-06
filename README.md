ESC/POS Print Driver for PHP
============================
This project implements a subset of Epson's ESC/POS protocol for thermal receipt printers. It allows you to generate and print receipts with basic formatting, cutting, and barcodes on a compatible printer.

The library was developed to add drop-in support for receipt printing to any PHP app, including web-based point-of-sale (POS) applications.

Basic usage
-----------
The library should be initialised with a file pointer to the printer. For a networked printer, this can be opened via [fsockopen()](http://www.php.net/manual/en/function.fsockopen.php). For a local printer, use [fopen()](http://www.php.net/manual/en/function.fopen.php) to open the printer's device file. If no file pointer is specified, then standard output is used.

A "hello world" receipt can be printed easily (Call this `hello-world.php`):
```php
<?php
require_once(dirname(__FILE__) . "/Escpos.php");
$printer = new Escpos();
$printer -> text("Hello World!\n");
$printer -> cut();
```
This would be executed as:
```
# Networked printer
php hello-world.php | nc 10.x.x.x. 9100
# Local printer
php hello-world.php > /dev/...
```

From your web app, you would pass the output directly to a socket:
```php
<?php
require_once(dirname(__FILE__) . "/Escpos.php");
$fp = fsockopen("10.x.x.x", 9100);
$printer = new Escpos($fp);
$printer -> text("Hello World!\n");
$printer -> cut();
fclose($fp);
```

Or to a local printer:
```php
<?php
require_once(dirname(__FILE__) . "/Escpos.php");
$fp = fopen("/dev/ttyS0", "w+");
$printer = new Escpos($fp);
$printer -> text("Hello World!\n");
$printer -> cut();
fclose($fp);
```

On Linux, your printer device file will be somewhere like `/dev/lp0` (parallel), `/dev/usb/lp1` (USB), `/dev/ttyUSB0` (USB-Serial), `/dev/ttyS0` (serial).

On Windows, the device files will be along the lines of `LPT1` (parallel), `COM1` (serial).

A complete real-world receipt can be found in the code of [Auth](https://github.com/mike42/Auth) in [ReceiptPrinter.php](https://github.com/mike42/Auth/blob/master/lib/misc/ReceiptPrinter.php). It includes justification, boldness, and a barcode.

Other examples are located in the [example/](https://github.com/mike42/escpos-php/blob/master/example/) directory.

Compatibility
-------------

### Interfaces and operating systems
This driver is known to work with the following OS/interface combinations:

<table>
<tr>
<th>&nbsp;</th>
<th>Linux</th>
<th>Mac</th>
<th>Windows</th>
</tr>
<tr>
<th>Ethernet</th>
<td><a href="https://github.com/mike42/escpos-php/tree/master/example/interface/ethernet.php">Yes</a></td>
<td><a href="https://github.com/mike42/escpos-php/tree/master/example/interface/ethernet.php">Yes</a></td>
<td><a href="https://github.com/mike42/escpos-php/tree/master/example/interface/ethernet.php">Yes</a></td>
</tr>
<tr>
<th>USB</th>
<td><a href="https://github.com/mike42/escpos-php/tree/master/example/interface/linux-usb.php">Yes</a></td>
<td>Not tested</td>
<td>Not tested</td>
</tr>
<tr>
<th>USB-serial</th>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<th>Serial</th>
<td>Yes</td>
<td>Yes</td>
<td>Yes</td>
</tr>
<tr>
<th>Parallel</th>
<td>Yes</td>
<td>Not tested</td>
<td>Not tested</td>
</tr>
</table>

### Printers
Many thermal receipt printers support ESC/POS to some degree. This driver has been known to work with:

- Epson TM-T88III
- Epson TM-T88IV
- Epson TM-T70
- Epson TM-T82II
- Epson TM-T20
- Epson TM-T70II
- EPOS TEP 220M

If you use any other printer with this code, please let me know so I can add it to the list.

Available methods
-----------------

### __construct($fp)
Construct new print object.

Parameters:
- `resource $fp`: File pointer to print to. Will open `php://stdout` if none is specified.

### barcode($content, $type)
Print a barcode.

Parameters:

- `string $content`: The information to encode.
- `int $type`: The barcode standard to output. If not specified, `Escpos::BARCODE_CODE39` will be used.

Currently supported barcode standards are (depending on your printer):

- `BARCODE_UPCA`
- `BARCODE_UPCE`
- `BARCODE_JAN13`
- `BARCODE_JAN8`
- `BARCODE_CODE39`
- `BARCODE_ITF`
- `BARCODE_CODABAR`

Note that some barcode standards can only encode numbers, so attempting to print non-numeric codes with them may result in strange behaviour.

### bitImage(EscposImage $image, $size)
See [graphics()](#graphicsescposimage-image-size) below.

### cut($mode, $lines)
Cut the paper.

Parameters:

- `int $mode`: Cut mode, either `Escpos::CUT_FULL` or `Escpos::CUT_PARTIAL`. If not specified, `Escpos::CUT_FULL` will be used.
- `int $lines`: Number of lines to feed before cutting. If not specified, 3 will be used.

### feed($lines)
Print and feed line / Print and feed n lines.

Parameters:

- `int $lines`: Number of lines to feed

### feedReverse($lines)
Print and reverse feed n lines.

Parameters:

- `int $lines`: number of lines to feed. If not specified, 1 line will be fed.

### graphics(EscposImage $image, $size)
Print an image to the printer.

Parameters:

- `EscposImage $img`: The image to print.
- `int $size`: Output size modifier for the image.

Size modifiers are:

- `IMG_DEFAULT` (leave image at original size)
- `IMG_DOUBLE_WIDTH`
- `IMG_DOUBLE_HEIGHT`

A minimal example:

```php
<?php
$img = new EscposImage("logo.png");
$printer -> graphics($img);
```

See the [example/](https://github.com/mike42/escpos-php/blob/master/example/) folder for detailed examples.

The function [bitImage()](#bitimageescposimage-image-size) takes the same parameters, and can be used if your printer doesn't support the newer graphics commands.

### initialize()
Initialize printer. This resets formatting back to the defaults.

### pulse($pin, $on_mm, $off_ms)
Generate a pulse, for opening a cash drawer if one is connected. The default settings (0, 120, 240) should open an Epson drawer.

Parameters:

- `int $pin`: 0 or 1, for pin 2 or pin 5 kick-out connector respectively.
- `int $on_ms`: pulse ON time, in milliseconds.
- `int $off_ms`: pulse OFF time, in milliseconds.


### selectPrintMode($mode)
Select print mode(s).

Parameters:

- `int $mode`: The mode to use. Default is `Escpos::MODE_FONT_A`, with no special formatting. This has a similar effect to running `initialize()`.

Several MODE_* constants can be OR'd together passed to this function's `$mode` argument. The valid modes are:

- `MODE_FONT_A`
- `MODE_FONT_B`
- `MODE_EMPHASIZED`
- `MODE_DOUBLE_HEIGHT`
- `MODE_DOUBLE_WIDTH`
- `MODE_UNDERLINE`

### setBarcodeHeight($height)
Set barcode height.

Parameters:

- `int $height`: Height in dots. If not specified, 8 will be used.

### setDoubleStrike($on)
Turn double-strike mode on/off.

Parameters:

- `boolean $on`: true for double strike, false for no double strike.

### setEmphasis($on)
Turn emphasized mode on/off.

Parameters:

- `boolean $on`: true for emphasis, false for no emphasis.

### setFont($font)
Select font. Most printers have two fonts (Fonts A and B), and some have a third (Font C).

Parameters:

- `int $font`: The font to use. Must be either `Escpos::FONT_A`, `Escpos::FONT_B`, or `Escpos::FONT_C`.

### setJustification($justification)
Select justification.

Parameters:

- `int $justification`: One of `Escpos::JUSTIFY_LEFT`, `Escpos::JUSTIFY_CENTER`, or `Escpos::JUSTIFY_RIGHT`.

### setReverseColors($on)
Set black/white reverse mode on or off. In this mode, text is printed white on a black background.

Parameters:

- `boolean $on`: True to enable, false to disable.

### setUnderline($underline)
Set underline for printed text.

Parameters:

- `int $underline`: Either `true`/`false`, or one of `Escpos::UNDERLINE_NONE`, `Escpos::UNDERLINE_SINGLE` or `Escpos::UNDERLINE_DOUBLE`. Defaults to `Escpos::UNDERLINE_SINGLE`.

### text($str)
Add text to the buffer. Text should either be followed by a line-break, or `feed()` should be called after this.

Parameters:

- `string $str`: The string to print.

Further notes
-------------
Posts I've written up for people who are learning how to use receipt printers:

* [What is ESC/POS, and how do I use it?](http://mike.bitrevision.com/blog/what-is-escpos-and-how-do-i-use-it), which documents the output of test.php.
* [Setting up an Epson receipt printer](http://mike.bitrevision.com/blog/2014-20-26-setting-up-an-epson-receipt-printer)
* [Getting a USB receipt printer working on Linux](http://mike.bitrevision.com/blog/2015-03-getting-a-usb-receipt-printer-working-on-linux)

Other versions
--------------
Some forks of this project have been developed by others for specific use cases. Improvements from the following projects have been incorporated into escpos-php:

- [wdoyle/EpsonESCPOS-PHP](https://github.com/wdoyle/EpsonESCPOS-PHP)
- [ronisaha/php-esc-pos](https://github.com/ronisaha/php-esc-pos)

Vendor documentation
--------------------
Epson notes that not all of its printers support all ESC/POS features, and includes a table in their documentation:

* [FAQ about ESC/POS from Epson](http://content.epson.de/fileadmin/content/files/RSD/downloads/escpos.pdf)

Note that many printers produced by other vendors use the same standard, and are compatible by varying degrees.

