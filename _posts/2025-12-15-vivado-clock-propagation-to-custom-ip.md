---
id: 2025-12-15-vivado-clock-propagation-to-custom-ip
aliases: []
tags: []
---

# Propagating Clock Frequency from Vivado IPI to Custom AXI IP

## The Problem

When creating custom AXI IP cores in Vivado, you often need to know the operating frequency of the clock signal. While Vivado's IP Integrator (IPI) automatically propagates clock frequencies between connected components, getting that frequency value into your custom IP's parameters requires specific configuration.

Without proper setup, you'll find that:
- The clock frequency parameter shows a default value (e.g., 100 MHz) regardless of the actual clock source
- Changing the clock source in your block design doesn't update your IP's frequency parameter
- Your IP can't adapt its timing or behavior based on the actual clock frequency

## The Solution

The solution involves three key steps:
1. Making your frequency parameter user-editable and propagate-able
2. Creating a bidirectional link between the clock interface and your parameter
3. Implementing propagation logic in the block design TCL scripts

Let's walk through each step.

## Step 1: Configure the User Parameter

First, ensure your frequency parameter in `component.xml` is set up correctly. The parameter must be user-resolvable and enabled:

```xml
<spirit:parameter>
  <spirit:name>FREQ_HZ</spirit:name>
  <spirit:displayName>Frequency (Hz)</spirit:displayName>
  <spirit:value spirit:format="long" spirit:resolve="user" spirit:id="PARAM_VALUE.FREQ_HZ">100000000</spirit:value>
  <spirit:vendorExtensions>
    <xilinx:parameterInfo>
      <xilinx:enablement>
        <xilinx:isEnabled xilinx:resolve="user" xilinx:id="PARAM_ENABLEMENT.FREQ_HZ">true</xilinx:isEnabled>
      </xilinx:enablement>
    </xilinx:parameterInfo>
  </spirit:vendorExtensions>
</spirit:parameter>
```

**Key points:**
- `spirit:resolve="user"` - Allows the parameter to be modified by propagation scripts
- `isEnabled = true` - Makes the parameter editable in the GUI
- Default value of `100000000` (100 MHz) provides a sensible fallback

**Common mistake:** Setting `spirit:resolve="generated"` or `isEnabled = false` will prevent the parameter from being updated, even if your propagation scripts are correct.

## Step 2: Link Clock Interface to Parameter

The critical step is creating a dependency between your clock bus interface's FREQ_HZ and your user parameter. Find the clock bus interface definition in `component.xml` (typically named `S_AXI_CLK` or similar):

```xml
<spirit:busInterface>
  <spirit:name>S_AXI_CLK</spirit:name>
  <spirit:busType spirit:vendor="xilinx.com" spirit:library="signal" spirit:name="clock" spirit:version="1.0"/>
  <spirit:abstractionType spirit:vendor="xilinx.com" spirit:library="signal" spirit:name="clock_rtl" spirit:version="1.0"/>
  <spirit:slave/>
  <spirit:portMaps>
    <spirit:portMap>
      <spirit:logicalPort>
        <spirit:name>CLK</spirit:name>
      </spirit:logicalPort>
      <spirit:physicalPort>
        <spirit:name>s_axi_aclk</spirit:name>
      </spirit:physicalPort>
    </spirit:portMap>
  </spirit:portMaps>
  <spirit:parameters>
    <spirit:parameter>
      <spirit:name>ASSOCIATED_BUSIF</spirit:name>
      <spirit:value spirit:id="BUSIFPARAM_VALUE.S_AXI_CLK.ASSOCIATED_BUSIF">S_AXI</spirit:value>
    </spirit:parameter>
    <spirit:parameter>
      <spirit:name>ASSOCIATED_RESET</spirit:name>
      <spirit:value spirit:id="BUSIFPARAM_VALUE.S_AXI_CLK.ASSOCIATED_RESET">s_axi_aresetn</spirit:value>
    </spirit:parameter>
    <spirit:parameter>
      <spirit:name>FREQ_TOLERANCE_HZ</spirit:name>
      <spirit:value spirit:id="BUSIFPARAM_VALUE.S_AXI_CLK.FREQ_TOLERANCE_HZ">-1</spirit:value>
    </spirit:parameter>
    <spirit:parameter>
      <spirit:name>FREQ_HZ</spirit:name>
      <spirit:value spirit:format="long" spirit:resolve="user" spirit:id="BUSIFPARAM_VALUE.S_AXI_CLK.FREQ_HZ" spirit:dependency="(spirit:decode(id('PARAM_VALUE.FREQ_HZ')) * 1)">100000000</spirit:value>
    </spirit:parameter>
  </spirit:parameters>
</spirit:busInterface>
```

**The magic ingredient:** The `spirit:dependency` attribute:

```xml
spirit:dependency="(spirit:decode(id('PARAM_VALUE.FREQ_HZ')) * 1)"
```

This expression creates a bidirectional relationship:
- When the clock source changes in IPI → the bus interface FREQ_HZ updates → your PARAM_VALUE.FREQ_HZ updates
- When you manually change PARAM_VALUE.FREQ_HZ → the bus interface FREQ_HZ updates

The `* 1` operation forces evaluation of the expression as an integer value.

## Step 3: Enable Parameter Propagation in bd.tcl

In your `bd/bd.tcl` file, you need to mark the parameter as propagate-overrideable. Add this to the `init` procedure:

```tcl
proc init { cellpath otherInfo } {
  # Allow this parameter to be overwritten by the propagate function
  set ip_handle [get_bd_cells $cellpath]
  bd::mark_propagate_overrideable $ip_handle "FREQ_HZ"
  
  # ... rest of init code
}
```

This tells Vivado that the FREQ_HZ parameter should be updated when properties propagate through the block design.

## Step 4: Add GUI Parameter Mapping (xgui)

In your `xgui/YOURIP_v1_0.tcl` file, ensure you have the update procedure for the model parameter:

```tcl
proc update_MODELPARAM_VALUE.FREQ_HZ { MODELPARAM_VALUE.FREQ_HZ PARAM_VALUE.FREQ_HZ IPINST } {
  # Procedure called to set VHDL generic/Verilog parameter value(s) based on TCL parameter value
  set_property value [get_property value ${PARAM_VALUE.FREQ_HZ}] ${MODELPARAM_VALUE.FREQ_HZ}
}
```

This ensures the GUI parameter value gets passed to your Verilog/VHDL module as a parameter.

## How It Works

Here's the complete flow when you connect a clock in IPI:

1. **Clock Connection**: You connect a clock source (e.g., processing system at 88.8 MHz) to your IP's clock input
2. **Vivado Propagation**: IPI's propagation engine runs and updates the `S_AXI_CLK` interface's FREQ_HZ
3. **Dependency Evaluation**: The `spirit:dependency` expression is evaluated, linking the interface FREQ_HZ to PARAM_VALUE.FREQ_HZ
4. **Parameter Update**: Your custom FREQ_HZ parameter is updated to 88.8 MHz
5. **Hardware Generation**: The new frequency value is passed to your RTL module as a parameter

## Testing the Configuration

1. **Add your IP to a block design** in Vivado IPI
2. **Connect a clock source** with a known frequency (e.g., ZYNQ processing_system7 FCLK_CLK0)
3. **Check the IP configuration** - Right-click your IP and select "Customize"
4. **Verify the FREQ_HZ parameter** matches the connected clock frequency

If it doesn't update:
- Check that `spirit:resolve="user"` and `isEnabled = true` in component.xml
- Verify the `spirit:dependency` attribute is correctly formatted
- Ensure `bd::mark_propagate_overrideable` is called in bd.tcl init
- Try refreshing the IP catalog and upgrading the IP in your block design

## Common Pitfalls

### Pitfall 1: Parameter Set to "generated"
```xml
<!-- WRONG -->
<spirit:value spirit:resolve="generated" ...>
```
This prevents user and script modifications.

### Pitfall 2: Missing Dependency Link
Without the `spirit:dependency` attribute, there's no connection between the clock interface and your parameter.

### Pitfall 3: Trying to Read from Pin or Net in post_propagate
Attempting to read frequency from pins or nets in `post_propagate` is unreliable:
```tcl
# DON'T DO THIS - unreliable
set freq [get_property CONFIG.FREQ_HZ [get_bd_pins "$cellpath/s_axi_aclk"]]
```

The dependency approach is more robust and follows Vivado's intended design flow.

### Pitfall 4: Forgetting to Upgrade IP
After modifying component.xml, you must:
1. Refresh the IP catalog (Tools → Report → Refresh IP Catalog)
2. Upgrade the IP in your block design (Right-click → Upgrade IP)

## Use Cases

This technique is valuable for:
- **Timing-critical logic**: Adapting timeout counters or state machine delays based on actual clock frequency
- **Baud rate generators**: Calculating divider values for UART or other serial protocols
- **Performance monitoring**: Reporting throughput in real-world units (MB/s instead of cycles)
- **Clock domain crossing**: Validating frequency relationships between multiple clocks
- **Power estimation**: Providing accurate frequency information to power analysis tools

## Complete Example Structure

Your IP should have these key files properly configured:

```
your_ip_1_0/
├── component.xml              # Parameter definitions with dependencies
├── bd/
│   └── bd.tcl                 # mark_propagate_overrideable in init
├── xgui/
│   └── your_ip_v1_0.tcl      # update_MODELPARAM_VALUE procedures
└── hdl/
    └── your_ip.v              # Module with FREQ_HZ parameter
```

## Conclusion

Propagating clock frequencies from Vivado IPI to custom IP parameters requires careful coordination between component.xml, bd.tcl, and xgui TCL scripts. The key insight is using the `spirit:dependency` attribute to create a bidirectional link between the clock interface's FREQ_HZ and your user parameter.

This approach is more robust than attempting to read frequency values from pins or nets in propagation scripts, as it leverages Vivado's built-in parameter dependency evaluation system.

Once configured correctly, your IP will automatically update its frequency parameter whenever the clock source changes, enabling adaptive timing and performance-aware behavior in your hardware designs.
