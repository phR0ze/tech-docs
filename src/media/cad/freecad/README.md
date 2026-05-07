# FreeCAD <img style="margin: 6px 13px 0px 0px" align="left" src="../../../data/images/logo_36x36.png" />

Documenting my research on FreeCAD

### Quick links
* [.. up dir](README.md)
* [FreeCAD Overview](#freecad-overview)
  * [FreeCAD competitors](#freecad-competitors)
  * [Install and configure FreeCAD](#install-and-configure-freecad)
  * [Crete new project](#create-new-project)
  * [Workbenches](#workbenches)
* [General object manipulation](#general-object-manipulation)
  * [Mouse Controls](#mouse-controls)
  * [Draw Rectangle](#draw-rectangle)
  * [Rotate object around axis](#rotate-object-around-axis)
* [BIM Workbench](#bim-workbench)
  * [Measure Area](#measure-area)
  * [Move with host](#move-with-host)
  * [Selection planes](#selection-planes)
* [Parts Design Workbench](#parts-design-workbench)
  * [Create a truss](#create-a-truss)
* [Techdraw Workbench](#techdraw-workbench)

## FreeCAD Overview
General purpose CAD. With the BIM plugin you can get a lot of architecture support similar to Revit 
or Sketchup.

**References**
* BIM - Building Information Modeling
* [BIM with FreeCAD playlist](https://www.youtube.com/playlist?list=PLmKdGVtV5Vnt2cj4IZIv9FM39QHaE1ZaU)
* [BIM with FreeCAD - Introduction](https://www.youtube.com/watch?v=rkWOFQ2fGZQ)
* [House using ARCH workbench](https://www.youtube.com/watch?v=RduDsY_8kJ8)
* [FreeCAD Tuts](https://www.youtube.com/playlist?list=PLDd21g-eSHwkkxVOfVmR8ObpPN5QbL7ye)

### FreeCAD competitors
* Revit
* Sketchup
* Rymn

### Install and configure FreeCAD
1. Install FreeCAD
   ```bash
   $ sudo pacman -S freecad
   ```
2. Install addons, navigate to `Tools > Addon manager`
   * `A2plus` workbench for combining parts into assemblies
   * `Render` allows you to use external renders to produce nice images
   * `FreeCAD-ArchTextures` provides ability to texture models in FreeCAD directly
   * `dxf-library` to export dxf file format
   * `flamingo` collection of tools to work with metalic structures
   * `parts_library` collection of pre-made objects
3. Restart FreeCAD
4. BIM Welcome screen you can revist it at `Help > BIM Welcome screen`
   1. Navigate to the `BIM` workbench from the workbench drop down widget
   2. Work through the BIM setup wizard
   3. Choose `Fill with default values` as `US/Imperial`
   4. Choose `Preferred working units` as `feet`
5. Configure Settings, navigate to `Edit > Preferences > General > Units`
   1. Choose `Unit System` as `Building US (ft-in/sqft/cft)`
6. Switch to `OpenbSCAD` mouse controls
   1. Bottom right where you see `CAD` change that to `OpenSCAD`
   2. Hover over your selection and it will show you how to use the mouse
   3. Also click the same menu and select `Settings >Orbin Style >Turntable`
   4. Press Shift and hold middle mouse button to pan
   5. Hit `Home` to return to home view

### Create new project
Example of starting a new project

1. Click the `Create new...`
2. Switch to the `BIM` workbench
3. Switch to top down view clicking `TOP` in top right view control
4. Choose the working plan as the `Top`
5. Select the rectangle tool
 
### Workbenches
Workbenches in FreeCAD are a collection of tools that can be selected to focus on different aspects 
of CAD work. You can customize workbenches and move tools from one to another.

* `Arch` - contains a lot of tools for working with Architecture
* `BIM` - addon for Building Information Modeling tools similar to Revit, AchiCAD, Tkla, AllPlan or BricsCAD

## General object manipulation

### Mouse Controls
At the bottom right hand of the screen you can change the mouse controls to `OpenSCAD` which feels
the most intuative to me.

* Left mouse button to orbit
* Right mous button to pan
* Scroll to zoom

### Draw rectangle
1. Select the rectangle tool

### Rotate object around axis
1. Select your object
2. Navigate to `Edit >Placement...`
3. In the `Rotation` section at the bottom click drop down and select `Euler angles (zy'x")`
4. Change `Pitch (around y-axis)` for roof angles

## BIM Workbench
The `BIM` workbench is essentially the Arch workbench with additional tooling and settings to help with 
BIM. Select it from the workbench drop down widget.

**References**
* [BIM roof](https://www.youtube.com/watch?v=XCPUkXVqNQ8&list=PLU9HicgJ9hhJvYJso3a6w2TlSLfQvFhzW&index=1)

### Configure BIM

#### Setup snap tools
Once in the BIM workbench you'll see a number of icon tools along the top. The `Teal` ones are the
snap widgets. Drag this to the right hand side for easier navigation.

### Measure Area
The easiest way to do this is to draw a temporary retangle over the area and measure that.

1. Choose the line tool and correct snap tools to draw a rectangle to measure
2. Split and Join down to just the shape you want
3. Set it to `Make Face`
4. If the face doesn't fill then set `Closed` to `true`
5. In the same property data window you can see the area now

### Switch to Wireframe
If you have a filled area already you can change it to a `Wireframe` by:
1. Selecting it
2. Switching to `View` in the `Model Properties` on the left
3. Scrolling to `Display Options` section
4. Changing `Display Mode` to `Wireframe`

### Split line
1. Select the intersection you want to split at
2. Use the split tool from the left hand `Tasks` pane or the icon tool

### Join lines
Simpley select the contiguous lines in the model and click the `Join` tool

### Move with host
BIM components that are built off a base rectangle like walls and slabs have a property called `Move 
With Host` that can be set to true to keep them together when you move the base rectangle

### Selection planes
Create selection planes for technical views that will show up in the `Techdraw Workbench`

1. Select all the components you'd like in the View
2. Hide any sub-components you don't want to show up
3. Click the `Selection Plane` button
4. Switch over to the `TechDraw` workbench


## Parts Design Workbench

References:
* [Framing a Smart Rafter](https://www.youtube.com/watch?v=vw7sY3GEBvQ)

### Create a truss
1. Click `Create body`
2. Click `Create sketch`
3. Select the `XZ_Plane (Base plane)` for intuitive building orientation
4. Click `OK`
5. Configure grid view
   1. On left expand the `Edit controls` section
   2. Check `Show grid` and set `Grid Size` to `1'`
6. Zoom out until you can see the grid
7. Select `Create polyline` and raw a rough outline of a rafter
8. Clean up with constraints for proper dimensions
   1. Select the places it should be horizontal and click `Constrain horizontally`
   2. Select the two rafter length lines and click the `Constrain parallel`
   3. Select the endpoints on one end and then the Z axis and click `Constrain point on object`
   4. Select the endpoints on the other end and then the X axis and click `Constrain point on object`
9. Click the `Toggle construction geometry` tool button
   1. Draw a line between to parallel lines and add `Constrain perpendicular`
   2. Select that line and set a `Constrain distance` of `3.5"` for the rafter width
   3. Drag the dimension labeling out away so you can see
   4. Add `Constrain distance` for `16"` overhang
   5. Add `Constrain distance` for run of `13' 10+3/4"` and rise of `3.5'` based on `3/12` pitch
10. Complete out the part
   1. Click `Close` on the left
   2. With the `Sketch` selected in the tree click the `Pad` button and enter `1.5"`
11. Zoom out until you see your padded shape

Note: my lines were all blue originally which indicates they are construction geometry only and when 
I went to pad it got and error. I then selected my shape lines and clicked the geometry button to 
turn them all green and then the pad worked.

### Add annotation sketch to truss sketch
1. First remove the `Pad`
   1. Switch to `Part Design Workbench`
   2. In the tree view on left select the `Pad` and right click and select `Delete`
2. Create a new sketch
   1. Switch to `Sketcher Workbench`
   2. In the tree view on left select your root document
   3. Click the `Create sketch` button to add a new sketch outside the body of the first
   4. Choose the same `XZ-Plane` as before and then set `Offset` to `1.5`
3. Copy the geometry of the first sketch
   1. Double click the new sketch to open it
   2. Click the `Carbon copy` button and select one of the lines from the original sketch. 
   * Note: despite the carbon copy terminology this sketch is tied to the original

## A2plus Workbench

### How to combine parts
This is how I added my truss part to my BIM structure

* [How to combine parts](https://www.youtube.com/watch?v=d68J-JMOYNQ)

1. Install the `A2plus` addon and restart
2. Create a new FreeCAD document and switch to the `A2plus Workbench`
3. Click the `Add a part from an external file` and choose your other file e.g. `truss.FCstd`
4. Place the part then rotate and move as necessary

## TechDraw Workbench
The TechDraw workbench is used to make a 2D technical drawing of your work based on the selection 
planes you created with BIM.

* [Techdraw 15 min](https://www.youtube.com/watch?v=02Gv4gN117M)

### Change TechDraw defaults
1. Navigate to `Edit >Preferences...`
2. Scroll down on the left hand side to `TechDraw`
3. On the `General` tab
   1. Set the `Label Size` to something larger `1/4"`
   2. Set the `Default Template` to `USLetter_Landscape_blank.svg`
4. On the `Scale` tab
   1. Change the `View Scale Type` to `Custom`
   3. Change the `Vertex Scale` to `0`
5. On the `Colors` tab
   1. Change `Dimension` to a blue to distinguish it from normal lines  
   Note: only applies to new dimensions

### Techdraw page setup
1. Switch to the `TechDraw workbench`
2. Click the `Insert Default Page` button top left
3. Double click the `Page > Template` object on left in the tree view
4. Under the templates properties below select the `Template` file path property and choose `USLetter_Landscape_blank.svg`
   * Optionally if you want the title block you'll need to use the `Template` called `Arch_D_Landscape_cust.svg`

### Insert technical views
1. Click drawing tab to bring up your drawing and position it as desired
2. Now select in the tree on the left the components you'd like to add to the TechDraw
3. Click the `Insert View` button to insert the selection
4. Change the view property `Scale Type` to `Custom`
5. Change the view property `Scale` to `0.005`

### Add dimensions to views
1. Select the edge in your TechDraw view
2. Click the dimension buttons as desired

### Edit drawing title block
Note for title block you'll need to change the template to `Arch_D_Landscap_cust.svg`

1. Right click on drawing and choose `Toggle Frames` to ensure they are visible
2. Now you can double click the green blocks in the title block to edit them

### Change drawing logo
[Tutorial on modifying drawing logo in SVG](https://www.youtube.com/watch?v=usA1erNCQOY)
1. Navigate to `/usr/share/freecad/Mod/TechDraw/Templates`
2. Right click on your target template e.g. `Arch_D_Portrait.svg` and choose `Open with >Inkscape`


### Insert dimensions
