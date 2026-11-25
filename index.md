---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---

# FPGA VGA Driver Project

The aim of this project was to use Verilog and the Basys 3 FPGA development board to generate graphics on a VGA monitor. I started with the provided VGA colour cycle and colour stripes designs, experimented with different settings, and then created my own animated Pac-Man design.

## Template VGA Design

### Project Set-Up

The project was created in Xilinx Vivado using the Basys 3 FPGA development board. The Verilog modules were added to Design Sources, the testbench was added to Simulation Sources, and the Basys 3 constraints file was added to Constraints.

The Clocking Wizard converts the board's 100 MHz clock into a 25 MHz pixel clock. The VGA design uses a resolution of 640 x 480 pixels, even though the monitor used in the lab has a native resolution of 1920 x 1080.

<!-- Add Project Summary screenshot if available. -->

### Template Code

| File or module | Function |
|---|---|
| `VGATop.v` | Connects the clock, VGA timing and image generation modules. |
| `VGASync.v` | Generates horizontal and vertical timing and pixel coordinates. |
| `VGAColourCycle.v` | Cycles through eight colours using a state machine and counter. |
| `VGAColourStripes.v` | Generates eight vertical colour stripes based on column coordinates. |
| `clk_wiz_0` | Generates the 25 MHz pixel clock. |
| `Basys3_Master.xdc` | Assigns the FPGA pins. |
| `Testbench.v` | Provides stimulus for behavioural simulation. |

Each RGB channel is four bits wide, giving values from 0 to 15 and a total of 4096 possible colours. The original colour stripes module divides the visible display into eight 80-pixel-wide stripes. The colour cycle module uses a counter and a state machine to change the RGB values.

### Simulation

Behavioural simulation can be used to examine the VGA timing signals before programming the FPGA. The horizontal counter is expected to count from 0 to 799 and the vertical counter from 0 to 524. The `vid_on` signal indicates the visible 640 x 480 area. The horizontal and vertical synchronisation pulses are active low.

<!-- Add template simulation waveform screenshot if available. -->

### Synthesis

Synthesis converts the Verilog design into digital logic. Implementation places and routes that logic on the FPGA. A bitstream can then be generated and programmed onto the Basys 3 using Vivado Hardware Manager.

<!-- Add template synthesis utilisation report if available. -->

### Demonstration

I tested the colour-pattern designs on the physical VGA monitor using different counter settings. The pixel clock was configured to 25 MHz for the VGA timing. Changing the counter setting changes how often the colour pattern updates, rather than changing the VGA resolution or the monitor's refresh rate.

#### Setting 1,500,000

<img width="3000" height="4000" alt="1000025261" src="https://github.com/user-attachments/assets/dd688ec4-2fed-4b90-955c-e8262247b3f8" />



*Figure 1: VGA colour pattern at a setting of 1,500,000.*

At this setting, the monitor displayed eight wide vertical colour stripes with a visible change in colour across the middle of the screen. The top and bottom parts of the image show different stages of the colour sequence. This is consistent with the colour pattern changing while a frame is being drawn, rather than waiting until the next frame begins.

#### Setting 5,000

<img width="3000" height="4000" alt="1000025260" src="https://github.com/user-attachments/assets/8aba7fc5-c56b-4168-8690-8868c29e8bca" />

*Figure 2: VGA colour pattern at a setting of 5,000.*

At this setting, the monitor displayed a repeating pattern of coloured rectangular blocks. The colours appeared at different horizontal and vertical positions across the screen.

#### Setting 2


<img width="3000" height="4000" alt="1000025259" src="https://github.com/user-attachments/assets/88696c62-cb21-44c7-b10e-105673aee338" />


*Figure 3: VGA colour pattern at a setting of 2.*

At this setting, the display showed a much finer pattern consisting of narrow vertical colour stripes. Comparing the three results demonstrated how changing the counter setting affected the pattern produced on the monitor. These photographs show the actual display results from the lab.

### Clock Speed and VGA Refresh Rate Limitations

The Basys 3 has a 100 MHz system clock, which is reduced to 25 MHz by the Clocking Wizard for the VGA pixel timing. At 25 MHz, a new pixel is generated every 40 ns. However, the display cannot update a complete image at that frequency because each frame contains many pixels and blanking periods.

The visible resolution is 640 x 480 pixels, but VGA timing also includes horizontal and vertical blanking intervals for synchronisation. The total timing is 800 pixel clocks per line and 525 lines per frame.

| VGA timing property | Value |
|---|---:|
| Pixel clock | 25 MHz |
| Time per pixel clock | 40 ns |
| Visible resolution | 640 x 480 |
| Total timing | 800 x 525 |
| Clock cycles per line | 800 |
| Time per line | 32 microseconds |
| Clock cycles per frame | 420,000 |
| Time per frame | 16.8 ms |
| Refresh rate | Approximately 59.52 Hz |

The refresh rate can be calculated using:

```text
Frame clocks = 800 x 525 = 420,000
Frame time   = 420,000 / 25,000,000 = 0.0168 seconds
Refresh rate = 25,000,000 / 420,000 = 59.52 Hz
```

This means the VGA output can present approximately 59.5 complete frames per second, even though the FPGA is processing 25 million pixel clocks per second. The monitor's native resolution of 1920 x 1080 does not increase the frame rate of the VGA signal generated by the FPGA.

#### Effect of Changing the Colour Counter

For a counter running at 25 MHz and updating every N clock cycles, the interval between colour changes is N / 25,000,000 seconds. The following table compares the three experimental settings using that timing model.

| Counter setting | Time between updates | Colour updates per second | Average updates per VGA frame |
|---:|---:|---:|---:|
| 1,500,000 | 60 ms | 16.67 | 0.28 |
| 5,000 | 0.2 ms (200 microseconds) | 5,000 | 84 |
| 2 | 80 ns | 12,500,000 | 210,000 |

These are calculated rates, not measured frequencies. The combined cycling-stripes design uses the 25 MHz pixel clock and a counter that updates at N - 1. The original supplied `VGAColourCycle.v` instead uses a `COUNT_TO` comparison and is connected to the 100 MHz board clock in its original `VGATop.v`. If that unmodified counter was used, the interval would be (N + 1) / 100,000,000 seconds rather than N / 25,000,000. The exact edited colour-pattern code used for the photographs was not retained with these results, so the table describes the 25 MHz timing model rather than claiming to be a measurement of the photographs.

At 1,500,000, a colour phase lasts approximately 3.57 frames under the 25 MHz model. Most frames can show the same colour pattern, but because the counter is not synchronised with the start of each frame, a colour change can still occur halfway through a frame. This can produce the horizontal colour boundary visible in Figure 1.

At 5,000, the counter updates approximately every 6.25 scan lines. The colour sequence can therefore change several times while the monitor is drawing the screen, producing the repeated blocks seen in Figure 2.

At 2, the counter updates every two pixel clocks under this model. The colour changes are far too fast to be displayed as separate complete images. Instead, different pixels within each scan line receive different colours, which is consistent with the narrow vertical stripes seen in Figure 3.

The monitor is not necessarily lagging behind or queuing images. The VGA raster is continuously scanning from left to right and top to bottom, while the FPGA's colour state may change during that scan. This creates a mixture of colour states within a single frame, which can appear as tearing, flickering or fine repeating patterns. The photographs show the resulting patterns but do not measure the monitor's actual input lag or response time.

#### Maximum Useful Image Update Rate

For one new complete image per VGA frame, the useful update limit is approximately 59.52 updates per second. This corresponds to 420,000 pixel-clock cycles per frame. A free-running counter set to 420,000 is not sufficient on its own to prevent tearing unless its update is also aligned with the VGA frame boundary.

A better approach is to update the colour phase or animation position only once per frame during the vertical blanking period, when the monitor is not drawing visible pixels. Slower animation can then be produced by counting multiple frames between updates. This keeps each visible frame consistent without changing the 25 MHz pixel clock or the VGA resolution.

## My VGA Design Edit

For my own design, I created a yellow Pac-Man that automatically moves clockwise along the edges of a solid bright blue rectangle on a white background. The character changes direction at each corner and has an animated mouth and a black eye. The animation does not require buttons for movement.

### Code Adaptation

I added a module called `PacManImage.v` and connected it to the existing VGA top-level design. The VGA synchronisation module and clock generator provide the pixel coordinates and clock needed to draw and animate the character.

#### Background and Blue Rectangle

The background is white because all three four-bit RGB channels are set to their maximum value.

```verilog
red   = 4'hF;
green = 4'hF;
blue  = 4'hF;
```

The solid blue rectangle is drawn using the current column and row coordinates.

```verilog
if (col >= 100 && col <= 340 &&
    row >= 220 && row <= 420) begin
    red   = 4'h0;
    green = 4'h0;
    blue  = 4'hF;
end
```

The rectangle covers 241 columns and 201 rows, including the boundary pixels. I adjusted its coordinates during testing until it appeared in the desired position on the physical monitor.

#### Drawing Pac-Man

The position of each pixel relative to Pac-Man's centre is calculated using signed arithmetic.

```verilog
dx = $signed({1'b0,col}) -
     $signed({1'b0,pac_x});

dy = $signed({1'b0,row}) -
     $signed({1'b0,pac_y});
```

The circle equation is used to draw Pac-Man with a radius of 22 pixels. Since 22 squared is 484, pixels inside this distance are coloured yellow unless they form part of the mouth.

```verilog
if ((x*x + y*y <= 484) && !mouth) begin
    red   = 4'hF;
    green = 4'hF;
    blue  = 4'h0;
end
```

A small black eye is added inside the yellow body using coordinate comparisons.

```verilog
if (x >= 5 && x <= 8 &&
    y >= -12 && y <= -9) begin
    red   = 4'h0;
    green = 4'h0;
    blue  = 4'h0;
end
```

#### Movement Speed

```verilog
localparam SPEED = 10_000;
reg [17:0] counter = 0;
```

The movement counter runs from the 25 MHz clock. Every 10,000 clock cycles, Pac-Man's position or direction is updated. This gives a calculated update interval of 0.4 ms and a nominal movement speed of 2500 pixels per second along a straight side. Reducing the SPEED value makes the movement faster, while increasing it slows the character down.

This is considerably faster than the 59.52 Hz VGA refresh rate. At this speed, approximately 42 movement updates occur during each 16.8 ms frame. The monitor cannot show all 42 intermediate positions as separate images, and the position can change while the character is being drawn, potentially making the animation appear to jump or tear. The logic is still updating at the programmed speed; this is a limitation of the raster refresh rate rather than evidence of a processing backlog.

A future improvement would be to update Pac-Man's displayed position once during vertical blanking. Moving one pixel per frame would give approximately 59.5 pixels per second, or two pixels per frame would give approximately 119 pixels per second, with a consistent position throughout each visible frame.

#### Starting Position and Direction

```verilog
reg [2:0] direction = 1;
reg [10:0] pac_x = 100;
reg [10:0] pac_y = 220;
```

Pac-Man starts at the top-left corner of the rectangle. When the reset input is activated, his position and direction return to their initial values.

| Direction value | Movement |
|---|---|
| 1 | Right |
| 2 | Left |
| 3 | Up |
| 4 | Down |

#### Automatic Clockwise Movement

A case statement controls the movement. Pac-Man moves one pixel at a time until he reaches a corner, where the direction changes.

```verilog
case (direction)
    1: begin // Right
        if (pac_x < 340)
            pac_x <= pac_x + 1;
        else
            direction <= 4;
    end

    4: begin // Down
        if (pac_y < 420)
            pac_y <= pac_y + 1;
        else
            direction <= 2;
    end

    2: begin // Left
        if (pac_x > 100)
            pac_x <= pac_x - 1;
        else
            direction <= 3;
    end

    3: begin // Up
        if (pac_y > 220)
            pac_y <= pac_y - 1;
        else
            direction <= 1;
    end

    default: direction <= 1;
endcase
```

| Corner | X coordinate | Y coordinate |
|---|---:|---:|
| Top-left | 100 | 220 |
| Top-right | 340 | 220 |
| Bottom-right | 340 | 420 |
| Bottom-left | 100 | 420 |

Pac-Man's centre follows the rectangle edges, meaning part of his body overlaps the blue rectangle as he moves. The route repeats continuously.

#### Image Rotation

A second case statement rotates the pixel coordinates so Pac-Man faces the direction of travel. This avoids having to create a separate image for each direction.

```verilog
case (direction)
    1: begin x = dx;  y = dy;  end
    2: begin x = -dx; y = -dy; end
    3: begin x = -dy; y = dx;  end
    4: begin x = dy;  y = -dx; end
    default: begin x = dx; y = dy; end
endcase
```

#### Mouth Animation

A separate counter toggles the mouth state after ten movement updates.

```verilog
if (mouth_count == 9) begin
    mouth_count <= 0;
    mouth_open <= ~mouth_open;
end
else
    mouth_count <= mouth_count + 1;
```

The mouth is generated by removing a wedge from the circle. The size of the wedge depends on `mouth_open`.

```verilog
ay = (y < 0) ? -y : y;

mouth = (x > 0) &&
        (mouth_open ? (2*ay < x)
                    : (10*ay < x));
```

This changes the mouth between a wider and a narrower opening while Pac-Man moves. At the final SPEED setting, the mouth state changes every 4 ms, which is faster than the monitor's frame interval. A separate slower animation counter could be used in a future improvement to make the opening and closing more visible.

#### RTL Schematic

<img width="1221" height="668" alt="pacman-rtl-schematic" src="https://github.com/user-attachments/assets/93f69ace-2b62-4b6e-82ed-79693a5f3f41" />


*Figure 4: Top-level RTL schematic of the Pac-Man VGA design.*

The Clocking Wizard supplies the pixel clock to the `VGASync` and `PacManImage` modules. The `VGASync` module generates the horizontal and vertical synchronisation signals and passes the row and column coordinates to `PacManImage`. Three multiplexers controlled by `vid_on` set the RGB outputs to zero outside the visible display area.

| RTL schematic property | Value |
|---|---:|
| Cells | 6 |
| I/O ports | 16 |
| Nets | 53 |

These values are from the top-level RTL schematic in Vivado and are not the final FPGA resource utilisation figures.

#### Expanded RTL Schematic

<img width="1198" height="159" alt="image" src="https://github.com/user-attachments/assets/10e360f2-39dd-43ee-9b1f-a25f72b9a7f2" />


*Figure 5: Expanded schematic showing the internal digital logic.*

The expanded schematic provides a more detailed view of the interconnected hardware logic generated from the Verilog design. It illustrates how operations such as counting, comparison, selection and arithmetic can be represented as digital hardware. The individual component labels are not legible at this scale.

### Simulation

The following signals can be examined in a behavioural simulation of the modified design.

| Signal | Function |
|---|---|
| `pac_x` | Horizontal position |
| `pac_y` | Vertical position |
| `direction` | Movement and facing direction |
| `counter` | Movement timing |
| `mouth_count` | Mouth timing |
| `mouth_open` | Mouth state |
| `red`, `green`, `blue` | Pixel colour |

The expected behaviour is that Pac-Man starts at (100, 220), moves towards the right, changes direction at X = 340 and continues downwards. The remaining sides repeat in the same way. Simulation can also be used to check the reset and mouth animation.

<!-- Add Pac-Man simulation waveform screenshot if available. -->

### Synthesis

The modified design introduces additional registers, counters, multiplexers, arithmetic and comparison logic compared with the original colour-pattern design. The position registers store Pac-Man's coordinates, the direction register controls his movement, and the arithmetic determines whether a pixel belongs to the circular character.

The RTL screenshots show the structure of the design. A synthesis utilisation report is needed to determine the actual number of LUTs, flip-flops and DSP resources used.

<!-- Add modified design synthesis utilisation report if available. -->

### Demonstration

<img width="3000" height="4000" alt="1000025258" src="https://github.com/user-attachments/assets/aadb2d3c-8af6-4a07-af5d-c2b3eccf4f75" />

*Figure 6: Final Pac-Man image displayed on the physical VGA monitor.*

The final design was displayed on the HP monitor connected to the Basys 3. The photograph shows the white background, solid bright blue rectangle and yellow Pac-Man with a black eye positioned along the top edge of the rectangle.

The Verilog code controls the automatic clockwise movement, direction changes and mouth animation. The final movement setting is 10,000 clock cycles per update. The photograph demonstrates the graphical output; it is a still image rather than a recording of the full animation.

## Problems Encountered

During development, I had to correct the direction of Pac-Man's mouth and use signed coordinate calculations so the character could face all four directions correctly. I also adjusted the rectangle's position several times to obtain the desired placement on the physical monitor.

Another issue was the RGB colour range. Initially, values such as 255 and 256 were used, but each colour output is only four bits wide. I corrected the colours to use values from 0 to 15. The movement speed was adjusted using the SPEED parameter. The colour-pattern experiments also showed the importance of synchronising image updates with the VGA frame rate to avoid mixed colour states within a frame.

## Conclusion

This project helped me understand how an FPGA generates VGA graphics using Verilog. I learned how to use pixel coordinates to draw shapes, counters to control movement, and case statements to change the direction of an animated character. I also gained experience using Vivado to examine RTL schematics and display a design on the Basys 3. The final project demonstrates how a basic VGA colour template can be developed into a custom animated image using digital logic. The clock-speed experiments also showed the difference between the FPGA pixel clock and the much slower full-screen refresh rate, and why frame-synchronised updates are important for consistent animation.
<img width="3000" height="4000" alt="1000025260" src="https://github.com/user-attachments/assets/68c09546-7d4a-455f-a4d7-b64bc8655f68" />
