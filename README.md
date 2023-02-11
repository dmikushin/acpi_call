# acpi\_call
This is a fork of [acpi_call](https://github.com/mkottman/acpi_call).

A simple kernel module that allows one to call ACPI methods by writing the
method name followed by its arguments to `/proc/acpi/call`.

## Building

    sudo ./one-time-setup
    sudo reboot
    make
    sudo make dkms-add
    sudo make dkms-install

The `one-time-setup` script above is intended to to have dkms automatically
sign the module after building, as described [here](https://gist.github.com/s-h-a-d-o-w/53c2215e955c3326c6ec8f812a0d2f27).

## Usage

    echo '<call>' | sudo tee /proc/acpi/call

You can then retrieve the result of the call by checking your dmesg or:

    sudo cat /proc/acpi/call

An example to turn off the discrete graphics card in a dual graphics
environment (like NVIDIA Optimus):

    # turn off discrete graphics card
    echo '\_SB.PCI0.PEG1.GFX0.DOFF' > /proc/acpi/call
    # turn it back on
    echo '\_SB.PCI0.PEG1.GFX0.DON' > /proc/acpi/call

These work on my ASUS K52J notebook, but may not work for you. See a [list of methods to try](http://linux-hybrid-graphics.blogspot.com/)
or try running the provided script `examples/turn_off_gpu.sh`.

It SHOULD be ok to test all of the methods, until you see a drop in battery
drain rate (`grep rate /proc/acpi/battery/BAT0/state`), however it comes
with NO WARRANTY - it may hang your computer/laptop, fail to work, etc.

You can pass parameters to `acpi_call` by writing them after the method,
separated by single space. Currently, you can pass the following parameter
types:

* ACPI_INTEGER - by writing NNN or 0xNNN, where NNN is an integer/hex
* ACPI_STRING - by enclosing the string in quotes: "hello, world"
* ACPI_BUFFER - by writing bXXXX, where XXXX is a hex string without spaces,
                or by writing { b1, b2, b3, b4 }, where b1-4 are integers

The status after a call can be read back from `/proc/acpi/call`:

* 'not called' - nothing to report
* 'Error: <description>' - the call failed
* '0xNN' - the call succeeded, and returned an integer
* '"..."' - the call succeeded, and returned a string
* '{0xNN, ...}' - the call succeeded, and returned a buffer
* '[...]' - the call succeeded, and returned a package which may contain the
   above types (integer, string and buffer) and other package types

