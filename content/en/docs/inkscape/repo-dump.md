---
title: "Repo dump"
weight: 300
description: >
  Repo dump that needs reorg.
---

v1.0.2-2

Features to learn:
- Live Path Effects (LPE)
  - Perspective 
  - Pattern Along a Path
- Power Stroke 
- Gradient mesh 
- Measurement tool
- View > Split Mode 
- Outline overlay
- Page: manages page setup and the usage of multiple paged Inkscape documents
- WEBP format
- Batch export
- Shape builder
- 

What is a path?

Path > Object to path changes the object to a vector path?

Live Path Effects (LPE) make dynamic changes to any path. Use this in the following circumstances:
- Render perspective
- Align along a path 

Power Stroke 

Path menu: Take objects that you already created and turn them into new objects by how they interact with each other:
- unify
- subtract
- use the intersecting area between them 

Live Path Effects:

## Docking a toolbox

Grab the tab, then drag it until there is a big blue box.

## Fill and stroke shortcuts

Select the item, then click:
- X in bottom left to remove the fill
- Select any color in bottom to change fill 
- SHIFT + color in bottom to change stroke color 




## Creating a drop shadow with Fill and Stroke 

1. Write text on the screen. 
2. Add a linear gradient
   1. Select the bottom stop, and use a slightly darker shade of the original color. Move the A (opacity) to the right.
3. Add a thick stroke (~10px)
4. Under Stroke Style > Order, put the stroke behind the text with the second option on the top.
5. Make the stroke white.
6. Right-click and create a duplicate of the original. Do not move the dup from the original.
7. Make the dupe's fill and stroke black.
8. Lower the dupe one layer below the original.
9. Increase the blur 
10. Decrease the opacity 
11. Click the original, and move it up or down to create a slight offset.
12. Select both objects, and group them.

## Markers

Repeat an object along a path without deforming the object. Different from Pattern Along Path.

1. Convert the object to a path with select, Path > Object to path
2. Find another shape, 
   1. Object > Object to marker
      The object disappears.
3. Rm the fill and stroke from the object that you turned into a path.
4. Go Fill and Stroke > Stroke style, and select the new marker in Markers.
   The marker goes along the nodes.
5. Add new nodes to the path object:
   1. Click the Edit path by nodes under the Select tool 
   2. Drag the cursor over the path object 
   3. Click Insert New Nodes into Selected Segments above the Select tool
      Click this more times for more markers.

To change the markers:
1. Fill and Stroke > Stroke style 
2. Click a marker dropdown 
3. Click Edit on Canvas 

Make sure you scale with the same proportions with the buttons in the toolbar.

## Clipping masks 

A way of taking one object and making it fit the shape of another object. You can do this with grouped objects and images. 

Object > Clip 

1. Create a design, or any object that you want a second object to take the shape of.
2. Add the second object on top.
3. Select all both objects.
4. Object > Clip > Set Clip

If you want to undo that, Object > Clip > Release Clip. You can still grab the nodes with the Edit paths by nodes tool.

## Layer masks 

Object > Mask > Set Mask

Way to create areas of selective transparency on an object. Basically, you create an object, then put another object over it (square or rectangle) that you can manipulate to control how much of the image comes through the mask. Use gradients, and black and white:
`
black: allows the underlying object to be transparent 
white: keeps anything underneath it opaque.


## Aligning and distributing objects 

Object > Align and Distribute (CTRL + SHIFT + A)

Align menu:
- Alignment handles with third click: When you select a group of objects, click 3 times to access alignment handles
- Move/align selection as group instead of grouping
- Relative to:
  - Page
  - Last selected lets you align relative to selection order. The last is the order (vice versa for first selected)

Centering text:
Use Align baseline of text button
- Can't use the axis bc some letters dip below the baseline

Rearrange
- Swap object locations

Remove overlaps

## Aranging Objects 

Grid: Arrange objects in a grid formation
- Fit into selection box: arrange within the area you get from the selection tool
  - Select how they are spaced apart with the diagram
  - Default rotate using the center. You can change that with the Objects' bounding boxes selection
  - First/Last selected... is when you have more than one circle and you need to decide which circle you will wrap the objects around.


Circular: Requires an ellipse 
- Wraps objects around the ellipse

## Transform Tool 

Object > Tranform 

Manipulate the object with precision. 

Move: Move the object using precise units. 
- Relative button makes changes relative to the document coordinates.

Scale: Scale based on numerical inputs.

Rotate: Rotate precisely. Rotate as a group or use Apply to each object separately.

Skew: Skew an object on the horizontal and vertical axis.


## Path functions 

View > Display Mode > Outline to see all the paths.

Every vector object is a series of X and Y coordinates (nodes). Use Edit paths as nodes tool to see the nodes. The lines that connect them are called paths. When you have a path, you have an element that you can apply a fill and stroke.

Rectangles, ellipses, and other shapes are not paths--they are objects. To change that:

Select the object, Path > Object to Path

Text is a text object, not a path. You can change them to Paths like objects.

You can change the stroke to its own object with Path > Stroke to Path. You probably have to ungroup the original object and the path after you do this.

## Path operations 

Path operations are boolean operations--you create shapes based on how shapes interact with each other.

Path > Union combines obects into one.
Path > Difference takes the object from top and subtracts it from the object on the bottom.
Path > Intersection deletes everything except the intersecting area
Path > Exclusion creates a shape from the areas that do not intersect. It creates negative space.
Path > Division cuts the object on the bottom using the shape of the object on top.
Path > Cut Path relates to strokes. Object on top breaks the paths on the image below. Only works for paths with strokes applied.

## More path operations 

Path > Combine vs Union? Combine is temporary and Union is permanent. Use Path > Break Apart when you want to undo a Combine  

Path > Break Apart does not preserve negative space 
Path > Split path preserves the negative space

## Offsetting Path 

Increase the size of an object by adding to its surface area rather than L x W.

Path > Dynamic offset, then click nodes tool and select the node to change the size.
Path > Inset | Offset increase or decrease the offset one step

Path > Linked offset. This seems like a shortcut that means you don't have to make copies of images to create outlines. You have to change it to a path to get more nodes, though, bc it is an offset.

## Tracing bitmaps 

Make vector tracing of a bitmap with an algorithm, instead of tracing it manually. Good for a quick, sometimes sloppy tracing of your bitmap.

Path > Trace Bitmap 
- Single scan creates a single vector object 
  - cycle through the Detection mode dropdown to see what gives you the best outcome
- Multiple scans are better for color 
  - Scans is the number of colors in the output.
  - Remove background only works on images where the bg is really defined 

## Trace pixel art 

Path > Trace bitmap 
This uses a lot of computing power. Do not use this for images that are greater than 100px by 100px.

Open the pixel art tab.
- Voroni: creates individual, grouped boxes 
- B-splines:

# Path effects 

Advanced tranformations to objects that are paths.

Turn the object into a path with Path > Object to Path, then select the new path and enter Path > Path Effects. Then it is a path effect object.

When you are done adding path effects, you have to select Path > Object to path again.

Click + icon at the bottom of the side panel too add a path effect. Click the - icon to remove an existing, highlighted path effect.

You can turn the path effect on or off by selecting the eye icon in the Path Effects menu.

## Attach path 

Attach 2 different paths together.

1. Create two paths.
2. Copy one to the clipboard
3. Select the other path
4. Click the + icon in the Path effects
5. Select Attach 
6. Under Start Path, select Link to path in clipboard

## Bend path 

This tool adds a path in the middle of an object, and you can bend an object along the path with Edit on-canvas button.

## BSpline 

Create curved lines without drawing the curve. Select the bezier pen and then select the BSpline mode from the top toolbar.

## Construct grid 

Creates a perspective grid from any three-point object. Create the object, apply the path effect, then change the number grid boxes along the X and Y axises.

## Corners (Fillet & Chamfer)

Allows you to take an object and give it rounded or square corners.
- Fillet: rounded corner
- Chamfer: squared off corners

Method options update the way the corners behave.

## Dashed stroke 

Turn a stroke into a dashed line. Gives you more granular control than the Fill and Stroke options.
- Hole factor: changes space between the dashes. 0 - 1 value.
- Use segments: changes the number of dashes for the entire shape.
- Half start/end: makes any dashes between nodes twice the size of the dashes on the nodes.
- Equalize dashes: makes dashes equal length

Use Fill and Stroke to change the dash style (butt cap, join style, etc.)

## Envelope deformation 

Change the object to path before you start. Helpful if you want to form text around a shape, like a circle.

Select X bend path to access a line that lets you mold the shape around the path.

## Hatches 

Applies a pen effect to the image. After you apply the Path effect, select the image and then select the nodes to change direction, density, style of the pen markings.

# Text 

## Creating text 

Click the T icon in the toolbar, or CTRL + SHIFT + T.
When you are actively editing the text, a shortcut toolbar displays at the top.

Spacing button: Kerning is the space between words or letters.

The node in the bottom right controls how far the text can go.

To change default font text:
1. Open the Text and Font pane
2. On the Font tab, select a new font 
3. Select Set as default at the bottom of the pane.

## Flowing text into a frame 

Click and drag a box to create a boundary for your text. Grab the bottom right node to edit the boundary.

1. Write some text 
2. Create an object 
3. Select the text, hold shift, select the object 
4. Text > Flow into frame. 

To undo this, select the text and Text > Unflow

## Putting text on a path

1. Create a path.
2. Write some text.
3. Select the text, hold shift, select the path.
4. Text > Put on path 

To change the location of the text on the path, you have to change it manually using kerning (in the Spacing menu)


### Wrapping text around a circle 

You cannot have multiple text items on a single path, or they will stack on top of each other. So, you have to create multiple circles

To change the direction of the path, select the Path, then Path > Reverse. 

After you put the text on the circle path, you can rotate just the circle to get the text on top (or wherever)

Put text on bottom of circle

1. Create text 
2. Create circle 
3. Select text, hold shift, select circle 
4. Text > Put on path 
5. Select just the circle 
6. Path > Object to path 
7. Path > Reverse 

After you wrap text around a circle, convert the text to the path with Path > Object to path.

# Filter effects 

Filters are pixel-based effects that you can apply to vector objects. When you apply a filter, the object is no longer a path.

Filters > * 

Filters are Inkscape-specific feature, so it won't work in any other image editor.

# Inkscape tools

## Edit paths by nodes 

Press n to edit the structural properties of a selected vector path.
You can grab the nodes and the edges of the new path to edit them.

Select an object and Path > object to path.

There is a toolbar that displays at the top, below the standard toolbar:
- Insert new nodes into selected segments: Select two nodes and it adds a new node exactly half way btwn the nodes. You can also just double-click the path, but this is exact 
- Insert new nodes into selected segments with a dropdown:
- Delete selected nodes: select and delete. You can also select and delete it.
- Join selected nodes: Join the nodes, create a new node 1/2 way btwn the two nodes. Good for joining two nodes when you are drawing shapes with bezier pen. 
- Break path at selected nodes: creates two separate nodes from one 
- Join selected end nodes with a new segment: creates new segment to join two nodes 
- Delete segment btwn two non-endpoint nodes: deletes segment btwn two nodes
- Make selected nodes corner: 
- Make selected nodes smooth: Handles attached to the node stay in place
- Make selected nodes auto smooth: Handles attached to the node move around and adjust themselves as needed 
- Make selected segment straight: 
- Make selected segment curved: Adds handles so you can shape the segment
...
- Show transformation handles for selected nodes: Path > object to path, then select some nodes, click this button. You can edit the nodes like they are an object. Click a node again and you get the rotate handles.
- Show path outline: helpful if you are working on an object that is similar color to the background.

## Squares and rectangles 

CTRL + SHIFT lets you draw the square from the center.
Circle nodes in the top-right corner let you change the shape of the edges
Use Make corners sharp to quickly correct corners.

## Circles and ellipses

Hold CTRL to make a perfect circle.
CTRL + SHIFT to make a perfect circle from the center.

- Square nodes change width and height 
- Round node lets you make a partial circle 
  - Cursor ouside the circle, you make a pie chart 
  - Cursor inside, you get a semi circle
  - There is an option in the ellipse toolbar that lets you make the circle whole again.

Slice, arch, closed shape icons in the ellipse toolbar

## Stars and polygons 

Star toolbar has regular polygon or star shape option at the very top-left. 
- Add corners

Nodes let you change the shape
- CTRL like other tools.

Click <= looking icon at the right to change it back to a normal star.

Polygon 
- Add corners, round the edges 

## 3D boxes

Press 'x' to access the box tool. 

X axis: red lines
Y axis: blue lines 
Z axis: yellow lines 

By default, X and Z axis have a vanishing point (the nodes at the far left and right, respectively). You can enable a vanishing point for any axis with the || icon beside the axis controls in the box toolbar.

After you finalize the box, you can deconstruct the box by changing it to a path:
1. Select the box 
2. Path > Object to path 
3. Select the box with the select tool, then click ungroup.

## Creating spirals 

Press 'i'

Each spiral is a single stroke. Change colors using the stroke style menus.

Outer handle increases, decreases outer turns to the spiral
Inner handle does the same from the inside.

You can change all this with the spiral toolbar. 
Click the <= looking icon to change it back to the default.

After you finalize the shape, you can change make it a path with Path > Object to path.

## Bezier pen 

Press 'b'
Lets you draw your own paths, however you like 

There are Modes in the top-left when you open the bezier tool. Create regular bezier path is the pen in its most raw form.

Each click is a node. Hold CTRL to better manage the angle of the line 
Click Enter when you are done drawing paths. 
- The path is a stroke. Use the stroke menu to edit it.
- Create a closed path by connecting the first node with the last. The node turns red when you can connect it. Then you can add fill and treat it as any other closed path.

If you click and drag at a point, you get a curved line. Hard to manage these curved lines, so use a different Mode. 
- Create spiro path: everything that you draw has a nice fluid curve.
  - It is not a pure path--when you move a node, it moves other segments/curves. Go to Path > Object to path 
- Create BSpline path: Creates more accurate curves. Makes it easier to trace over objects than the spiro mode.
  - Hold SHIFT and click and you get a corner node.
- Create a sequence of straight line segments: Only lets you draw straight lines. Use to trace over something that has only straight lines to prevent you from creating a curved line. 
  - This creates a pure path, you don't need to convert it.
- Create a sequence of paraxial line segments: Every line that you draw runs adjacent to the previous line. useful if you are drawing a floor plan or something like that.
- Shape: Draw paths using specific shapes. None is the default.
  - Path takes the shape that you select. You can edit the paths to change the shape.
  - After you are done, Path > object to path.
  - From clipboard: draws whatever shape you have in your clipboard. Use to create custom brush strokes.
  - Bend from clipboard: works with grouped objects. This is pretty cool.
    - Path > Object to path, then Object > ungroup 

## Tweak objects 

Press 'w'. press spacebar to toggle btwn tweak and seelct tool 

Make small changes to paths, objects, and colors 

In the toolbar, width is the size of the cursor circle, and Force is how it applies the change. The lower the Force, the less visible the change. 

Available modes are self-explanatory.
Fidelity: Recommend 50. When it is high, it is gonna create a lot of new nodes and increase the file size dramatically.

## Zoom 

Press 'z' 

Click to zoom in, SHIFT + click to zoom out.

Toolbar is useful for quick zooms.

## Measurements

Press 'm'. Enable snapping to help you get an accurate measurement.

Click and drag to measure distance btwn points. Can tweak precision or scale.

Helpful for text, provides Font sizing.

Make sure Measure all layers is on in the toolbar.
Convert to item: Turns measurements into an object--good for diagrams.

## Drawing freehand lines

Press 'p'

BSpline is a way to create curved lines with corners.
Change the line to path to further edit it. Path > object to path

Put smoothing at 50 to create good lines.
Can create lines with Shapes.
- From clipboard lets you stretch a path in your clipboard across a line.

## Caligraphy pen 

Press 'c'

Mass is helpful to draw good clean lines because it slows the cursor down.

## Spraying objects (airbrush)

Select an object, select the spray tool, then use the toolbar to:
- spray copies
- clones 
- a single path (single path is cpu intensive)


## Eraser 

SHIFT + E
Use Cut out from paths and shapes Mode 

After you erase a vector object, you can move the nodes around to change the shape of the erased part.
- To split it into separate parts, select the Break Apart Cut Items icon in the far right of the toolbar.
- Or you can select and then Path > Break apart 
If you use the eraser on a text item, it becomes a path--you cannot erase it.

## Bucket Fill 

Fills negative space with a color and creates a new path from it.

Requires a closed-in area--you can't just use it on the workspace.
- If there are tiny gaps, use the Close gaps: setting in the toolbar.

Threshold might leave gaps in the fill area.

## Linear and radial gradients 

Press 'g'

Use fill and stroke menu, click on one of the gradient tools in the Fill tab.

Press g, then select a stop (node point where there is color information) and then click a new color if you want. 

When you select a stop to change gradient direction, hold CTRL to help with making it level

Double click on the gradient line and add a new stop to add a new color. You can click and drag to move it after you create it.
- Click on it and delete it 

Radial gradients run in a circle rather than in a direction.

Menu provides options to create gradient on fill or stroke.

There is a drop down that contains all the gradients that you have made so you can easily apply them to other objects. 
- Lock the gradient to apply updates to each object that uses that gradient (except the position of the handles)

Repeat: Anything after the gradient ends is solid. THis dropdown changes that.

Offset: Changes the posotion of the selected stop on the gradient line.

## Mesh and conical gradients 

Select from toolbar
Creates objects with various different colors.

Different options are in the toolbar.

Select the mesh tool, then drag the cursor across an object. Select a node or a side and change the color.

Change the rows and columns to add more boxes and nodes to change colors.

Conical gradient is similar to the mesh, but it creates a gradient in a circle.

If you move the lines too much, you can straighten them or make them smooth curves.

Scale mesh to fit in binding box: When you grab the nodes and change the shape, this button takes that shape and makes it fit within the original confines of the object.

## Color picker

Press 'd'
Select an object and assign its color using another object. So, you select an object, open the color picker, then click on a different object and the original object that you selected becomes that color.

Good to use for photos. If you select an entire area, then the color picker uses the average of that selection.

Pick and assign modes

Pick AND assign: preserves original color (like transparency)
Pick: Uses original color, no transparency 
Assign: Creates new color from appearance. If there is transparency, it has the same color but not the transparency.

## Diagram connectors

Press 'o'
Creates bezier paths that connect objects together. Does not work on text objects. 
- 1 px size stroke works well bc thats the size of the connectors.

toolbar options let you avoid or ignore objects.
Create right-angle lines  
Curvature rounds the joints 
Spacing separates lines when an object is connected to multiple lines. Spacing makes sure they don't connect at the same point.

Add arrows with Fill and Stroke > Stroke style tab