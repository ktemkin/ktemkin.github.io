---
title: "Including non-nMigen Instances in nMigen"
date: 2021-08-18T04:30:18-06:00
draft: false
---

A common question in the `#nMigen` Libera.Chat channel asks how one can include a non-nMigen component---e.g. a library cell or a component from a Verilog design---so it seemed worth writing up a quick note we can link to answer  that.

#### Invoking a Component

The key is to create an `nmigen.Instance` component, as in the following example:

```python
m.submodules.my_component = Instance("CustomFlipFlop",
		# Attributes
		a_HasAsyncReset = False,
		
		# Parameters
		p_NumberInputs  = "1",
		p_NumberOutputs = "1",

		# Inputs
		i_D = my_input_signal,
		
		# Ouptuts
		o_Q = my_output_signal,
		
		# Bidirectional / inouts
		io_DZ = my_tristate_pin
)
```

The first parameter to the `Instance` constructor is the name of the component to be instantiated, which usually matches e.g. your library cell instance, or your Verilog module name. The remaining (keyword) arguments are the inputs and outputs to the Instance. As these inputs/outputs to this function currently lack type information, we'll need to prefix each name with a short prefix, depending on its type:

| Prefix | Corresponding HDL Type     |
|:-------|:---------------------------|
| `a_`   | Attribute                  |
| `p_`   | Parameter                  | 
| `i_`   | Input Port                 |
| `o_`   | Output Port                |
| `io_`  | Bidirectional / inout port |

#### Including HDL Sources

If you're using an external HDL source -- for example, a Verilog file -- you'll need to tell nMigen to include the relevant HDL source in your build plan. This is accomplished by calling the `add_file` method on the `platform` being used to  build your project, which takes the following signature:

```py
platform.add_file(filename, file_contents)
```

Here, `filename` is the name that will be used internally to the nMigen build plan (including by the target synthesis tool); and `file_contents` is either the file's contents (as a `str` or `bytes` object), or a file-like object.

An example of adding a single Verilog file to a design is as follows:

```py
if __name__ == "__main__":
    platform = ICE40HX1KBlinkEVNPlatform()
    
    # Add our Verilog file.
    platform.add_file("my_verilog_design.v", open("path_to_my_verilog_design.v"))
    
    platform.build(Blinky(), do_program=True)
```

As this is Python, it's entirely possible to easily add e.g. all `.v` files in the current folder:

```py
import glob

if __name__ == "__main__":
    platform = ICE40HX1KBlinkEVNPlatform()
    
    # Add all Verilog files in our working directory:
    for filename in glob.glob("*.v"): 
        platform.add_file(filename", open(filename))
    
    platform.build(Blinky(), do_program=True)
```
