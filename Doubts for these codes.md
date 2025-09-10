#### **1) Why do we write full\_adder dut ( ... ); ?**

You’re instantiating the design (DUT = Device Under Test) inside the testbench.



Syntax: module\_type instance\_name ( .port(signal), ... );



dut is just a convention. You could name it U0, fa\_i, my\_chip, etc. The name doesn’t affect behavior.





**Nuvvu ne code lo iche ports, testbench lo e ports ki match chestunav. Andhuke a port ki a ani b port ki b ani rastunav.**









#### **2) Do I always need $dumpfile / $dumpvars for waveforms?**



For Icarus/Verilator/EDA Playground, yes—if you want a VCD waveform (EPWave/GTKWave).



They tell the simulator “start writing a VCD file and include these signals.”



Common, portable pattern:



initial begin

&nbsp; $dumpfile("wave.vcd");       // file name (you can change it)

&nbsp; $dumpvars(0, full\_adder\_tb); // dump everything under this TB hierarchy

end



Variations you can (and should) use



Change file name per test:



$dumpfile("fa\_basic.vcd");





Limit scope/size:



$dumpvars(1, full\_adder\_tb);        // only one level deep

$dumpvars(0, full\_adder\_tb.dut);    // only DUT hierarchy





Dump specific signals:



$dumpvars(0, full\_adder\_tb.a, full\_adder\_tb.b, full\_adder\_tb.c,

&nbsp;            full\_adder\_tb.sum, full\_adder\_tb.carry);





Other simulators: ModelSim/Questa typically use their own commands (log -r /\*; run -all) or $wlfdumpvars. But for Icarus/EDA Playground, stick to $dumpfile/$dumpvars.





#### 

#### 

#### **3) Why do we write $monitor(...) ?**



$monitor auto-prints whenever any of its arguments change. You set it once.



Great for quick visibility while debugging.

How is it different from $display and $strobe?



$display(...) prints right now when you call it.



$strobe(...) prints at the end of the current time slot (after all updates settle).



$monitor(...) prints automatically on any change to its arguments (until $monitoroff).





#### 

#### 

#### **“Do I have to repeat the same lines in every TB?” (No—make them yours)**



You can (and should) standardize a TB skeleton and tweak it per design:



Minimal TB skeleton (reusable):



`timescale 1ns/1ps



module tb\_name;

&nbsp; // 1) Declarations

&nbsp; reg  clk;          // if needed

&nbsp; reg  reset\_n;      // if needed

&nbsp; reg  \[N-1:0] in;   // your inputs

&nbsp; wire \[M-1:0] out;  // your outputs



&nbsp; // 2) DUT instantiation

&nbsp; my\_module dut (/\* .ports(...) \*/);



&nbsp; // 3) Waves

&nbsp; initial begin

&nbsp;   $dumpfile("tb\_name.vcd");

&nbsp;   $dumpvars(0, tb\_name);

&nbsp; end



&nbsp; // 4) Optional clock/reset

&nbsp; initial clk = 0;

&nbsp; always #5 clk = ~clk;



&nbsp; initial begin

&nbsp;   reset\_n = 0; #12; reset\_n = 1;  // release reset after a bit

&nbsp; end



&nbsp; // 5) Monitor (optional)

&nbsp; initial $monitor("t=%0t | in=%h out=%h", $time, in, out);



&nbsp; // 6) Stimulus

&nbsp; initial begin

&nbsp;   // drive inputs here

&nbsp;   // check results (see self-checking pattern below)

&nbsp;   #100 $finish;

&nbsp; end

endmodule









#### **Quick rules to avoid future doubt**



DUT instantiation: module\_type instance\_name ( .port(signal) );



Waves: $dumpfile/$dumpvars once per TB (you can change filename/scope).



Printing:



$monitor once if you want auto updates,



$display for explicit prints,



add assertions / $fatal for self-checking tests.



Timescale: keep `timescale 1ns/1ps at the top of every file—your #10 means 10 ns then.



Naming: Keep \*\_tb for testbenches, dut for instance name—it makes wave browsing easier.









