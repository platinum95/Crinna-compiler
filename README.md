# Crinna
Kind of like a mix of Verilog, C, and PHDL? Subject to (lots of) change.

primitives:
    - All the C-primitives for non-synthesised stuff

    - resistor( ohms );
    - capacitor( uF );
    - diode();
    - Other primitives I've forgotten

New components can be specified through module definitions.
Can have some standard-library of common components that aren't primitives like 
triacs, transistors, LEDs and so forth.

Source files are headers (defining module interfaces) and module-files defining 
the module itself. 
Each source can be synthesised down to a netlist and doesn't need to be 
re-synthesised unless its source changes, like C.


These really just define the interface between components, without proper 
specification of electrical properties (other than for resister/capacitor), but
there's nothing stopping you from defining more in custom modules with C-type
primitives.
General idea is to have the type-based constraints done separately, which will
basically just give more specification to the components in a design.

Footprint of types not considered until .brd (or similar) generation. Again can
have some spec/constraint files for this.


Sample Hello RC module defining a basic hipass filter:

```
// helloRC.h
template< int freq >
module HelloRC
    interface( net inSignal, net outSignal, net gnd );
public:
    <some public components>
private:
    <some private components>
endmodule
```


```
// helloRC.pl
// Simple RC hipass filter
template< int freq >
module HelloRC( net inSignal, net outSignal, net gnd )
    
    // C-like primitives not synthesised but used to generate module instance
    int rVal = 1000; //ohm
    int cVal = 1.0/( 2 * 3.14 * rVal * freq ) * 1e6; // (uF)

    resistor r1( rVal );
    capacitor c1( cVal );

    // "anonymous" net
    c1.a = inSignal;

    // Named net
    net sampleNet;

    c1.b = sampleNet;
    r1.a = sampleNet;

    r1.b = gnd;
endmodule
```

This module can then be used in other modules like so:
```
#include <helloRC.h>

module filterBank( net inSignal, net outSignal, net gnd )
    net f1Out, f2Out;
    HelloRC< 1000 > f1( inSignal, f1Out, gnd );
    HelloRC< 512 > f2( f1Out, f2Out, gnd );
    HelloRC< 2048 > f3( f2Out, outSignal, gnd );
endmodule
```

