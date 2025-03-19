# SCL_Tapeout

### **1. Define Die Dimensions & Initialize Floorplan**
```tcl
set dieWidth 1415
set dieHeight 810

floorPlan -site CoreSite -noSnapToGrid -d $dieWidth $dieHeight 30 30 30 30
```
- Defines die width and height.
- Initializes floorplan with `CoreSite` and **30-unit margins** on all sides.

---

### **2. Enable Floorplan Mode & Place Standard Cells**
```tcl
setPlaceMode -fp true
place_design
```
- Ensures placement follows floorplan constraints.
- Runs automatic **standard cell placement**.

---

### **3. Add Halos (Keep-out Margins) Around Macros**
```tcl
addHaloToBlock {12 12 12 12} -allBlock
```
- Adds a **12-unit halo** around **all macros** to prevent routing congestion.

---

### **4. Manually Place Memory Instances**
```tcl
selectInst inst_mem
setObjFPlanBox Instance inst_mem 70 362.59 657.04 747.88

selectInst data_mem
setObjFPlanBox Instance data_mem 739 362.59 1291.513 747.88
```
- Selects and places **`inst_mem`** at `(70, 362.59) to (657.04, 747.88)`.
- Selects and places **`data_mem`** at `(739, 362.59) to (1291.513, 747.88)`.

---

### **5. Connect Power and Ground Nets**
```tcl
globalNetConnect VDD -type pgpin -pin VDD -verbose -netlistOverride -override
globalNetConnect VSS -type pgpin -pin VSS -verbose -netlistOverride -override
globalNetConnect VSSO -type pgpin -pin VDDO -override -verbose -netlistOverride
globalNetConnect VSSO -type pgpin -pin VSSO -override -verbose -netlistOverride
```
- Connects **VDD** and **VSS** to the global power and ground nets.
  
### **6. Assign Pin Locations**
```tcl
getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Bottom -layer 1 -spreadType start -spacing 190.84 -start 190.84 0.0 -pin {{input_gpio_pins[0]} {input_gpio_pins[1]}}
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Bottom -layer 1 -spreadType start -spacing 0.56 -start 572.0 0.0 -pin resetn
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -use CLOCK -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection clockwise -side Bottom -layer 1 -spreadType start -spacing 0.56 -start 762.0 0.0 -pin clk
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Bottom -layer 1 -spreadType start -spacing 0.56 -start 953.0 0.0 -pin uart_rxd
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Bottom -layer 1 -spreadType start -spacing 0.56 -start 1144.0 0.0 -pin uart_rx_en
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Left -layer 1 -spreadType start -spacing 100 -start 0.0 250.0 -pin uart_rx_break
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Left -layer 1 -spreadType start -spacing 0.56 -start 0.0 500.0 -pin uart_rx_valid
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection clockwise -side Top -layer 1 -spreadType start -spacing 150 -start 150.0 $dieHeight -pin {{uart_rx_data[0]} {uart_rx_data[1]} {uart_rx_data[2]} {uart_rx_data[3]} {uart_rx_data[4]} {uart_rx_data[5]} {uart_rx_data[6]} {uart_rx_data[7]}}
setPinAssignMode -pinEditInBatch false

getPinAssignMode -pinEditInBatch -quiet
setPinAssignMode -pinEditInBatch true
editPin -pinWidth 0.28 -pinDepth 0.88 -fixOverlap 1 -unit MICRON -spreadDirection counterclockwise -side Right -layer 1 -spreadType start -spacing 187.5 -start $dieWidth 187.5 -pin {{output_gpio_pins[0]} {output_gpio_pins[1]} {output_gpio_pins[2]}}
setPinAssignMode -pinEditInBatch false
```
- Assigns pin locations on **Bottom**, **Left**, **Top**, and **Right** sides of the die.
- Uses **0.28 μm width** and **0.88 μm depth** for most pins, with specific spacing and starting coordinates.
- Pins include GPIO, UART signals, clock (`clk`), and reset (`resetn`), placed in batch mode with overlap fixing enabled.

### **7. Define Power Rings for Core and Macros**
```tcl
earlyGlobalRoute

snapFPlan -guide -block -stdCell -ioPad -pin -pinGuide -routeBlk -pinBlk -ptnCore -placeBlk -macroPin
setDrawView fplan

setAddRingMode -ring_target default -extend_over_row 0 -ignore_rows 0 -avoid_short 0 -skip_crossing_trunks none -stacked_via_top_layer TOP_M -stacked_via_bottom_layer M1 -via_using_exact_crossover_size 1 -orthogonal_only true -skip_via_on_pin {standardcell} -skip_via_on_wire_shape {noshape}
addRing -nets {VDD VSS} -type core_rings -follow core -layer {top M5 bottom M5 left TOP_M right TOP_M} -width {top 6 bottom 6 left 6 right 6} -spacing {top 5 bottom 5 left 5 right 5} -offset {top 7.2 bottom 3 left 3 right 3} -center 0 -threshold 0 -jog_distance 0 -snap_wire_center_to_grid None

selectInst inst_mem
addRing -type block_rings -nets {VDD VSS} -around selected -layer {top M5 bottom M5 left TOP_M right TOP_M} -width 4 -spacing 1 -offset 1

selectInst data_mem
addRing -type block_rings -nets {VDD VSS} -around selected -layer {top M5 bottom M5 left TOP_M right TOP_M} -width 4 -spacing 1 -offset 1
```
- Runs **`earlyGlobalRoute`** for initial routing estimation.
- Initializes floorplan with `snapFPlan` and sets view to floorplan mode.
- Adds **core power rings** for `VDD` and `VSS` using `M5` (top/bottom) and `TOP_M` (left/right) with **6 μm width** and **5 μm spacing**.
- Adds **block rings** around `inst_mem` and `data_mem` with **4 μm width**, **1 μm spacing**, and **1 μm offset**.

### **8. Power Routing and Optimization**
#### **8.1 Special Routing for Power Nets**
```tcl
sroute -connect { blockPin padPin padRing corePin floatingStripe } -layerChangeRange { M1 TOP_M } -blockPinTarget { nearestTarget } -padPinPortConnect { allPort oneGeom } -padPinTarget { nearestTarget } -corePinTarget { firstAfterRowEnd } -floatingStripeTarget { blockring padring ring stripe ringpin blockpin followpin } -allowJogging 1 -crossoverViaLayerRange { M1 TOP_M } -nets { VDD VSS } -allowLayerChange 1 -blockPin useLef -targetViaLayerRange { M1 TOP_M }
```
- Routes **VDD** and **VSS** connections between block pins, pad pins, and core pins.
- Uses layers `M1` to `TOP_M`, allows jogging, and targets nearest connections.

#### **8.2 Add Power Stripes**
```tcl
addStripe -skip_via_on_wire_shape Noshape -block_ring_top_layer_limit TOP_M -max_same_layer_jog_length 0 -padcore_ring_bottom_layer_limit M5 -set_to_set_distance 60 -skip_via_on_pin Standardcell -stacked_via_top_layer TOP_M -stacked_via_bottom_layer M1 -padcore_ring_top_layer_limit TOP_M -spacing 1 -xleft_offset 10 -merge_stripes_value 0.56 -layer TOP_M -block_ring_bottom_layer_limit M5 -width 2 -nets {VDD VSS}
```
- Adds **2 μm wide** power stripes for `VDD` and `VSS` on `TOP_M`.
- Sets **60 μm set-to-set distance**, **1 μm spacing**, and **10 μm left offset**.

#### **8.3 Placement Configuration**
```tcl
setPlaceMode -fp true
setPlaceMode -place_global_ignore_scan 0
setPlaceMode -place_global_reorder_scan true
setPlaceMode -place_detail_swap_eeq_cells true
setPlaceMode -place_detail_no_filler_without_implant true

placeDesign -prePlaceOpt
```
- Configures placement mode with floorplan constraints.
- Enables scan reordering and runs placement with pre-optimization.

#### **8.4 Pre-CTS Optimization**
```tcl
source ccopt1.spec
setOptMode -fixCap true -fixTran true -fixFanoutLoad false
optDesign -preCTS
```
- Loads timing specs from `ccopt1.spec`.
- Optimizes capacitance and transition violations before CTS.

#### **8.5 Clock Tree Synthesis (CTS)**
```tcl
setCTSMode -engine ccopt
ccopt_design
```
- Configures CTS with the `ccopt` engine and synthesizes the clock tree.

#### **8.6 Post-CTS Optimization**
```tcl
optDesign -postCTS
```
- Optimizes design after clock tree synthesis.

#### **8.7 Global and Detailed Routing**
```tcl
routeDesign -globalDetail
```
- Performs global and detailed routing for all nets.

#### **8.8 Post-Route Optimization**
```tcl
optDesign -postroute
```
- Final optimization after routing to fix timing and other violations.


### **9. Fillers, Power Stripe Adjustments, and Timing Analysis**
#### **9.1 Add Filler Cells**
```tcl
addFiller -cell feedth9 -prefix FILLER -doDRC
addFiller -cell feedth3 -prefix FILLER -doDRC
addFiller -cell feedth -prefix FILLER -doDRC
```
- Adds filler cells (`feedth9`, `feedth3`, `feedth`) with DRC checks to fill gaps in the layout.

#### **9.2 Configure Analysis and Optimization Modes**
```tcl
setAnalysisMode -analysisType onChipVariation -cppr both
setOptMode -fixCap true -fixTran true -fixFanoutLoad false
```
- Sets analysis mode to **on-chip variation** with common path pessimism removal (CPPR) for both paths.
- Enables optimization for capacitance and transition fixes.

#### **9.3 Adjust VSS Power Stripes (Top Section)**
```tcl
selectWire 212.4600 703.7800 214.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 272.4600 703.7800 274.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 332.4600 703.7800 334.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 392.4600 703.7800 394.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 452.4600 703.7800 454.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 512.4600 703.7800 514.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 572.4600 703.7800 574.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 632.4600 703.7800 634.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 752.4600 703.7800 754.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 812.4600 703.7800 814.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 872.4600 703.7800 874.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 932.4600 703.7800 934.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 992.4600 703.7800 994.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 1052.4600 703.7800 1054.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 1112.4600 703.7800 1114.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 1172.4600 703.7800 1174.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 1232.4600 703.7800 1234.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
deselectAll
selectWire 1292.4600 703.3400 1294.4600 739.6800 6 VSS
editResize -direction y -offset 4.93 -side low -keep_center_line auto
```
- Resizes multiple `VSS` stripes on layer 6 (top section) by **4.93 units** downward.

#### **9.4 Adjust VSS Power Stripes (Bottom Section)**
```tcl
selectWire 212.4600 5.9200 214.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 272.4600 5.9200 274.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 332.4600 5.9200 334.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 392.4600 5.9200 394.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 452.4600 5.9200 454.4600 322.0350 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 512.4600 5.9200 514.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 572.4600 5.9200 574.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 632.4600 5.9200 634.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 752.4600 5.9200 754.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 812.4600 5.9200 814.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 872.4600 5.9200 874.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 932.4600 5.9200 934.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 992.4600 5.9200 994.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 1052.4600 5.9200 1054.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 1112.4600 5.9200 1114.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 1172.4600 5.9200 1174.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 1232.4600 5.9200 1234.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
deselectAll
selectWire 1292.4600 5.9200 1294.4600 321.6900 6 VSS
editResize -direction y -offset -4.565 -side high -keep_center_line auto
```
- Resizes multiple `VSS` stripes on layer 6 (bottom section) by **-4.565 units** upward.

#### **9.5 Timing Analysis**
```tcl
set_db timing_analysis_type best_case_worst_case
set_db timing_analysis_check_type setup
set_db timing_analysis_clock_propagation_mode sdc_control
report_timing
set_db timing_analysis_check_type hold
set_db timing_analysis_clock_propagation_mode sdc_control
report_timing
```
- Configures timing analysis for **best-case/worst-case** scenarios.
- Checks **setup** and **hold** timing with SDC-controlled clock propagation and generates reports.


### **10. GDSII Stream Out**
```tcl
streamOut main3.gds -mapFile /home/vlsiuser14/tapeout_components_scl180/large_mem/icc_gds_out_6LM.map -libName DesignLib -merge { /home/vlsiuser14/tapeout_components_scl180/large_mem/6M1L/gds/scl18fs120.gds /home/vlsiuser14/tapeout_components_scl180/large_mem/iopad/cio250/6M1L/gds/tsl18cio250_6lm.gds} -uniquifyCellNames -units 1000 -mode ALL
```
- Exports the design to `main3.gds` GDSII file.
- Uses mapping file `icc_gds_out_6LM.map` and library `DesignLib`.
- Merges external GDS files for standard cells (`scl18fs120.gds`) and I/O pads (`tsl18cio250_6lm.gds`).
- Sets precision to **1000 units** and includes all design elements with unique cell names.

