# **Cube3D v4.0.0 \- Function Logic & Breakdown**

This document provides a detailed, step-by-step explanation of the User Interface, the constant lookup tables, and every JavaScript function inside the Cube3D application.

## **Table of Contents**

1. [UI Button Descriptions](https://www.google.com/search?q=%23ui-button-descriptions)  
2. [3D Geometry & Rendering Functions](https://www.google.com/search?q=%231-3d-geometry--rendering-functions)  
3. [State Management & Utility Functions](https://www.google.com/search?q=%232-state-management--utility-functions)  
4. [Algorithm / Solve Functions](https://www.google.com/search?q=%233-algorithm--solve-functions)  
5. [Movement & Animation Processors](https://www.google.com/search?q=%234-movement--animation-processors)  
6. [Input Bindings & Mouse/Touch Controls](https://www.google.com/search?q=%235-input-bindings--mousetouch-controls)  
7. [Automated Testing Module](https://www.google.com/search?q=%236-automated-testing-module)  
8. [Constant Lookup Tables & Notations](https://www.google.com/search?q=%238-constant-lookup-tables--notations)  
9. [Global Event Listeners & Chat Parser](https://www.google.com/search?q=%239-global-event-listeners--chat-parser)  
10. [Full Table Contents](https://www.google.com/search?q=%2310-full-table-contents)

## **UI Button Descriptions**

The application features a control panel on the left side and a step-by-step solve tracker on the right side.

* **Shuffle:** Applies 20 random 90-degree slice rotations to scramble the cube completely.  
* **Reset:** Instantly resets the cube to its pristine, solved canonical state (White top, Blue front, etc.) and clears progress.  
* **Solve:** Automatically triggers the step-by-step solving algorithm from the current scrambled state. It utilizes the cube\_solve() function to progress through Steps 1 to 6\. It stops and alerts you if it encounters an unmapped configuration.  
* **Toggle Corners (Hidden by Default):** Toggles the visual Arabic/Roman numeral overlays on target pieces. This button only appears once the solver algorithms have identified pieces.  
* **Save:** Serializes the exact current 3D cube state into a text string and automatically downloads it to your computer as a .txt file.  
* **Load:** Opens an OS file browser allowing you to select and load a previously saved .txt cube state into the canvas.  
* **Record:** Toggles macro recording mode. When active, every manual slice rotation you make is logged. When you click it again to stop, it allows you to assign those recorded moves to a missing lookup table entry via the terminal.  
* **Test Loop:** Initiates an infinite, automated testing cycle. It shuffles the cube, completely randomizes the 3D camera angle, and attempts to solve it autonomously. It tracks successful consecutive passes and stops immediately if a failure occurs.  
* **Get View:** Prints the current raw X and Y camera angles to the terminal.  
* **Dump Tables:** Exports all currently loaded algorithm sequence lookup tables (corners, edges, etc.) into a downloadable .txt file for backup.  
* **Back Move (Bottom Right):** Reverses the last executed mathematical slice rotation. If recording mode is active, it dynamically prunes the recorded macro string.

## **1\. 3D Geometry & Rendering Functions**

### **getSliceIndices(f, r, c)**

1. Takes a 2D face index f (0 to 5\) and grid row/col r and c.  
2. Evaluates the face index to determine which 3D axis is fixed (e.g., Face 0 fixes y at 0).  
3. Maps the 2D row/col to the dynamic x and z axes respectively.  
4. **Returns:** An object {x, y, z} representing the specific cubie's index in the 3x3x3 array.

### **createFaceGrid(faceIdx, fixedAxis, fixedVal, axis1, axis2)**

1. Loops 3 times for the row and 3 times for the column.  
2. Initializes 4 vertex coordinates (corners of a square) for a single facelet in 3D space.  
3. Sets the static coordinate for the fixed axis (e.g., z \= 1 for the Top face).  
4. Assigns the spatial offsets (-1, \-1/3, 1/3, 1\) to axis1 and axis2 to position the vertices physically in the 3D grid.  
5. Calls getSliceIndices() to map this physical square to the logical 3x3x3 data array.  
6. Pushes the combined data (vertices, index, logical map) to the global faces array.

### **rotate3D(x, y, z, ax, ay)**

1. Receives a 3D coordinate and camera angles (ax for pitch, ay for yaw).  
2. Applies a standard 2D rotation matrix around the Y-axis using ay (Yaw).  
3. Applies a second rotation matrix around the X-axis using ax (Pitch) on the new coordinates.  
4. Swaps/inverts the resulting axes specifically for proper isometric canvas projection.  
5. **Returns:** \[x2, \-z2, y2\].

### **project3D(x, y, z)**

1. Passes the raw 3D coordinate to rotate3D() to factor in the camera angles.  
2. Calculates a perspective scaling factor based on the resulting Z-depth and the global fov.  
3. Multiplies the rotated X and Y by the scaling factor and adds the canvas center point (cx, cy) to center the projection.  
4. **Returns:** The final 2D \[px, py\] canvas coordinates.

### **rotateSlice3D(x, y, z, axis, angle)**

1. Takes a 3D coordinate, the axis to rotate around (x, y, or z), and the angle in radians.  
2. Checks which axis is being rotated.  
3. Applies a 2D rotation matrix to the *other* two coordinate axes using standard Math.cos and Math.sin formulas.  
4. **Returns:** The new rotated 3D coordinate.

### **rotateSliceArray(axis, index, dir)**

1. Converts the directional move (1 or \-1) into a 90-degree radian angle (PI/2).  
2. Creates a completely deep clone of the 3x3x3 cube3D data structure so mutations don't overlap during execution.  
3. Iterates over every single coordinate (x, y, z) from 0 to 2\.  
4. Checks if the current coordinate belongs to the slice moving on the chosen axis.  
5. If in the slice, passes the coordinate to rotateSlice3D() to find out where this cubie physically ends up.  
6. Rounds the result to snap it to the new 0, 1, or 2 integer array index.  
7. Iterates over the cubie's color facings. If a color exists, applies rotateSlice3D to the face's normal vector to determine where that color is pointing *after* the rotation.  
8. Writes the reorganized cubie into the newly cloned cube array at its new coordinates.  
9. Overwrites the global cube3D array with the updated clone.

### **draw(activeAnim)**

1. Clears the HTML5 canvas.  
2. Creates an empty projectedFaces array.  
3. Iterates over all 54 items in the global faces array.  
4. Checks if an animation is provided (activeAnim). If so, checks if the current face belongs to the moving slice and applies rotation data.  
5. Loops over the 4 vertices of the face, rotating them in 3D space if animating.  
6. Projects the 3D vertex to 2D coordinates using project3D() and calculates the average Z-depth of the 4 points.  
7. Looks up the current color of the facelet from the global cube3D array using the stored array mappings.  
8. Pushes the 2D polygon, color, and average depth to projectedFaces.  
9. Sorts projectedFaces based on depth (Painter's Algorithm: back to front).  
10. Caches the array into lastProjectedFaces for mouse picking later.  
11. Iterates over sorted faces, drawing the filled polygon path and stroking the black borders.  
12. If corner indications are enabled, calculates the 2D center point of the facelet.  
13. If the facelet belongs to an identified target corner or side, draws its respective number or Roman numeral directly onto the polygon using canvas text. Overlays markers like o for orientations.  
14. Projects, sorts, and draws the small Red/Green/Blue 3D axis indicator in the top right corner.

## **2\. State Management & Utility Functions**

### **updatePassedCountUI()**

1. Grabs the passedTestCount HTML element.  
2. Overwrites its text content with the current value of the global passedTestCount variable.

### **updateStepUI(maxStep)**

1. Loops through step identifiers (1 to 6).  
2. If the iterator is less than or equal to maxStep, removes the disabled CSS class from the right-panel buttons.  
3. If greater, adds the disabled CSS class.

### **resetStepProgress()**

1. Resets currentMaxStep to 1 and calls updateStepUI(1).  
2. Hides the HTML elements that display the target face and side face color palettes.  
3. Clears internal identification arrays and resets state flags.  
4. Hides the "Toggle Corners" button and strips gradient background progress visually from all step buttons.

### **checkSolved()**

1. Iterates over the top layer and verifies every cubie's 'top' color matches the top center piece.  
2. Repeats this exhaustive matching iteration for the bottom, front, back, left, and right faces against their respective center pieces.  
3. **Returns:** true if every single facelet aligns; otherwise returns false.

### **updateSolvedStatus()**

1. Evaluates the boolean output of checkSolved().  
2. If true, adds the active CSS class to the solved indicator, empties the move history, calls resetStepProgress(), and forces max step to 0 to disable buttons.  
3. If false, removes the active class and ensures Step 1 is activated if progress was reset.

### **resetCube()**

1. Guards against firing while animations are currently running.  
2. Re-initializes the global cube3D array from scratch, manually assigning default colors (Blue front, White top, etc.).  
3. Resets camera angles to standard isometric defaults.  
4. Clears history, resets steps, runs updateSolvedStatus(), and triggers draw() to re-render.

### **logMoveToTerminal(axis, index, dir, steps)**

1. Maps semantic axes (x, y, z) to numeric values (0, 1, 2\) and rotation directions (1, \-1) to bit strings (0, 1).  
2. Concatenates the components into standard shorthand format (e.g., "02:11").  
3. Calls addLog() to print the move into the visible UI terminal.  
4. Checks if the isRecording flag is active. If so, pushes the shorthand string into the recordedMoves array.

### **shuffleCube()**

1. Clears the solved indicator and resets the step progress pipeline.  
2. Runs a loop 20 times.  
3. For each iteration, randomly selects an axis, a slice index (0, 1, 2), and a direction.  
4. Constructs an animation object for the move, sets a rapid speed, pushes it to animationQueue, logs to terminal, and kicks off requestAnimationFrame(animateTurn).

### **serializeCube()**

1. Extracts the 6 core center colors from the 3D data structure to generate a shorthand layout prefix (e.g., "WBYROG").  
2. Iterates over every x, y, z coordinate in the cube3D array.  
3. Converts existing color facelets to character codes (T, O, F, B, R, L) and pairs them with their coordinate block.  
4. **Returns:** A 55-chunk, space-separated string containing the entire exact physical state.

### **deserializeCube(str)**

1. Splits the incoming string argument by whitespace and validates chunk length.  
2. Generates a blank, 3x3x3 cube3D null array.  
3. Loops over the parts of the string in pairs (XYZ chunk and Colors chunk).  
4. Parses the characters, maps the letters back to numerical color IDs, and writes them directly into the specific properties of the targeted cubie.  
5. Overwrites the global cube3D array and **returns** true.

### **saveCube(), loadCube(), dumpTablesToFile()**

* File I/O handlers for converting internal string/dictionary state to dummy Blob URLs for automatic .txt file downloading, and \<input type="file"\> reading.

## **3\. Algorithm / Solve Functions**

### **alignCubeToSelection()**

1. Runs matrix calculations simulating where the camera is looking based on angleX and angleY.  
2. Computes the mathematical dot product of the camera vectors against standard face normal vectors to deduce which face is pointing most "UP" and "FRONT".  
3. Executes sequence of applyWholeCube logic statements to physically rotate the entire 3D array data so the viewed orientation aligns with the data index axes.  
4. Resets camera angles back to default seamlessly.

### **executeStep1()**

1. Bails out if an animation is currently executing.  
2. Runs alignCubeToSelection().  
3. Looks up the canonical front color corresponding to the current top center color.  
4. Rotates the entire cube along the Z-axis until the front center matches the canonical requirement.  
5. Updates UI color squares, upgrades to Step 2, snaps camera to 0.625 angle, and renders.

### **top\_corner\_identify(printToTerminal)**

1. Retrieves center colors and builds target objects for the 4 top corners.  
2. Exhaustively loops through the 8 physical corners of the array.  
3. Extracts present colors. If a cubie matches a target, computes orientation and 'solved' status.  
4. Pushes tracking object to topCornersData, recalculates visual progress bar percentage.  
5. If requested, logs unsolved edge paths to terminal. Updates canvas overlay numerals.

### **top\_corner\_solve(corner)**

1. Constructs a string key (e.g., c1\_200:1) representing the corner's exact location and twisting orientation.  
2. Looks up the sequence from CORNER\_SOLUTIONS.  
3. Dispatches to executeShorthandSequence() if found, logs failure if missing.

### **executeStep2()**

1. Locks isStep2Running.  
2. **First Click:** Runs identifying logic silently, turns on UI overlays.  
3. **Subsequent Clicks:** Re-identifies state silently, grabs first unsolved corner, and triggers top\_corner\_solve().  
4. Hooks an anonymous callback to check if all corners are finally solved post-animation, and upgrades to Step 3\.

### **top\_side\_identify, top\_side\_solve, top\_side\_check, executeStep3()**

* Mirrors the logic of the Step 2 corners, but analyzes the 2-color edge pieces across the TOP\_SIDE\_SOLUTIONS table to assemble the top cross.

### **edge\_side\_identify, edge\_side\_solve, mid\_edge\_check, executeStep4()**

* Mirrors Step 3, but targets the 4 middle-layer vertical edge pieces using the EDGE\_SIDE\_SOLUTION table. Ensures alignment strictly to the canonical Front/Back axes.

### **bottom\_corner\_identify**

* Measures the 4 bottom-layer corners. Critically, it tracks isLocationSolved (XYZ match) vs isOrientSolved (Twist match) independently rather than one binary solved value.

### **bottom\_corner\_solve\_location(wrongLocations)**

1. If 3 or 4 corners are out of place, applies a baseline bottom layer rotation ("02:01") to alter positions geometrically.  
2. If 1 or 2 corners are out of place, computes a location-based key (e.g., bcl1\_000) and executes sequence from BOTTOM\_CORNER\_LOC\_SOLUTION.

### **bottom\_corner\_solve\_orientation(wrongOrientation)**

1. Determines orientation pattern by grouping corners by the horizontal face their 'bottom' color points towards.  
2. **Priority 1:** Finds if two corners face the identical side (triggering BCO2).  
3. **Priority 2:** Locates one solved corner and an adjacent corner pointing toward an interconnected shared face (triggering BCO1).  
4. Executes sequence from BOTTOM\_CORNER\_ORIENT\_SOLUTION.

### **executeStep5()**

1. Enforces Step 4 completion. Changes view to the bottom face.  
2. Evaluates location array first. If unlocated, fires bottom\_corner\_solve\_location().  
3. If locations are perfect but twisted, fires bottom\_corner\_solve\_orientation().  
4. Hooks recursive callback to evaluate completion and advance to Step 6\.

### **bottom\_side\_identify(printToTerminal)**

* Isolates the 4 bottom edges and records location/orientation properties.

### **bottom\_side\_solve\_location()**

1. Checks if exactly 1 bottom side is correctly located. If so, retrieves the side piece resting in its relative left-adjacent slot to compute an origin-destination key (e.g., bsl\_1\_4).  
2. If 0 edges are correctly located, it picks a "BSL ANY" sequence out of BOTTOM\_SIDE\_LOC\_SOLUTION array zero-index to safely force a permutation cycle shift.  
3. Executes the retrieved sequence or traps a missing CLI command.

### **bottom\_side\_solve\_orientation(wrongOrientation)**

1. Collects the identifiers of pieces resting correctly but flipped.  
2. Sorts identifiers into a concatenated string key (e.g., bso\_23).  
3. Evaluates BOTTOM\_SIDE\_ORIENT\_SOLUTION and dispatches shorthand rotation array.

### **executeStep6()**

* Locks pipeline, alters camera angle, evaluates location followed by orientation for the final 4 edges. On final successful pass, forces updateSolvedStatus() to fully complete the app solve.

## **4\. Movement & Animation Processors**

### **rotateLayer(axis, layer, dirString, steps)**

1. Validates axis (x/y/z) and layer index.  
2. Maps semantic directions to integers (1 or \-1).  
3. Loops steps times, pushing a move instruction object to animationQueue.  
4. Kicks off requestAnimationFrame(animateTurn) if not animating.

### **executeShorthandSequence(val)**

1. Uses regular expression /^\[012\]\[012\]:\[01\]\[12\]$/ to validate formatting.  
2. Splits strings into sequences.  
3. Iterates over valid moves, parses the regex matches into arguments, and routes them to rotateLayer().

### **animateTurn()**

1. If queue is empty, flips isAnimating false, triggers draw(), fires onAnimationComplete closures, and halts.  
2. Extracts first move object and adds speed to progress.  
3. If progress \> 90-degrees (Math.PI / 2), finalizes rotation in data structure via rotateSliceArray(), shifts object off queue, and recurvsively loops to next turn.  
4. If progress \< 90-degrees, calls draw(anim) to render the transitional twist and requests next animation frame.

## **5\. Input Bindings & Mouse/Touch Controls**

### **handleInteractionStart(x, y)**

1. Blocks input if animating. Stores starting coordinates.  
2. Iterates backwards through lastProjectedFaces (front to back). Uses pointInPolygon to see if user clicked inside a rendered face.  
3. Caches target to dragStartFace or enables isGlobalDragging if missed.

### **handleInteractionMove(x, y)**

1. If isGlobalDragging, increments global angleX/angleY camera values and calls draw().  
2. If dragging a face:  
   * Computes drag distance.  
   * Looks up valid 3D axes a specific face is legally allowed to turn on.  
   * Simulates micro-rotations for each axis/direction and computes mathematical dot product comparing user's swipe vector against the simulated 2D line.  
   * If the best dot product \> 0.5, packages the resulting optimal move into animationQueue and fires animateTurn().

## **6\. Automated Testing Module**

### **waitForAnimation()**

* Initiates an asynchronous while-loop. Forces the thread to sleep for 100ms chunks using JavaScript Promises if animations are currently processing, blocking the surrounding test routine until the physical UI completely settles.

### **cube\_solve()**

1. Externalized master solve loop.  
2. Sequentially executes executeStep1() through executeStep6().  
3. Between every step execution, it yields to waitForAnimation().  
4. Utilizes for loops to limit the max number of clicks/attempts allowed per step (e.g. up to 16 times for Step 5).  
5. If the executeStepX module returns false (indicating a missing entry in a lookup table), or if the loop caps out without incrementing currentMaxStep, it immediately returns false to abort the solve gracefully.  
6. **Returns:** true if checkSolved() verifies absolute completion.

### **runAutomatedTest()**

1. Infinite while-loop contingent on the isTesting flag.  
2. Triggers shuffleCube() and yields.  
3. Completely randomizes floating-point angleX and angleY coordinate views, redraws, and yields.  
4. Awaits the boolean result of cube\_solve().  
5. If true, increments global passedTestCount, updates the UI, yields, and restarts the shuffle loop.  
6. If false, traps the error, breaks out of the while loop, and disables the testing UI flags.

## **8\. Constant Lookup Tables & Notations**

The algorithm uses specific naming conventions and macro string syntaxes to record and execute solutions to individual puzzles dynamically.

### **Rotation Syntax (02:11)**

The solver uses a 5-character shorthand separated by a colon to instruct the 3D engine how to twist slices.

* **Character 1 (0):** The layer slice index (0 \= Top/Left/Front, 1 \= Mid, 2 \= Bottom/Right/Back).  
* **Character 2 (2):** The 3D Axis (0 \= X-axis, 1 \= Y-axis, 2 \= Z-axis).  
* **Character 3 (:):** Delimiter.  
* **Character 4 (1):** Direction of rotation (0 \= Clockwise, 1 \= Counter-Clockwise).  
* **Character 5 (1):** The number of 90-degree steps to execute.

*(Example: 02:11 means "Rotate Layer 0 along the Z-axis, counter-clockwise, 1 time").*

### **CORNER\_SOLUTIONS (Step 2\)**

Handles Top Layer Corners.

* **Key Format:** c\[ID\]\_\[X\]\[Y\]\[Z\]:\[FacingAxis\]  
* **Explanation:** c1\_200:1 targets Corner \#1 (Front-Right). It is currently located at \[x:2, y:0, z:0\] and its primary top color is currently facing outwards on Axis 1 (the Y-axis).

### **TOP\_SIDE\_SOLUTIONS (Step 3\)**

Handles Top Layer Edges.

* **Key Format:** s\[ID\]\_\[X\]\[Y\]\[Z\]:\[FacingAxis\]  
* **Explanation:** s2\_012:0 targets Side \#2 (Right-Top). It is currently located at \[x:0, y:1, z:2\] and its primary top color is currently facing out on Axis 0 (the X-axis).

### **EDGE\_SIDE\_SOLUTION (Step 4\)**

Handles Mid Layer Edges.

* **Key Format:** m\[ID\]\_\[X\]\[Y\]\[Z\]:\[FacingAxis\]  
* **Explanation:** m1\_010:2 targets Mid Edge \#1 (Front-Right). It is currently located at \[x:0, y:1, z:0\] and its primary color is facing out on Axis 2 (the Z-axis).

### **BOTTOM\_CORNER\_LOC\_SOLUTION (Step 5\)**

Handles Bottom Corner Positioning.

* **Key Format:** bcl\[ID\]\_\[X\]\[Y\]\[Z\]  
* **Explanation:** bcl1\_000 targets Bottom Corner \#1 (Front-Right-Bottom). It specifically addresses when that piece is in the incorrect slot located at \[x:0, y:0, z:0\].

### **BOTTOM\_CORNER\_ORIENT\_SOLUTION (Step 5\)**

Handles Bottom Corner Twisting once in the correct slot.

* **Key Format 1:** bco1\_S\[SolvedID\] \-\> Triggered when exactly one corner is solved. E.g., bco1\_S3 (Corner 3 is correct, the rest must rotate based on an adjacent facing condition).  
* **Key Format 2:** bco2\_\[Face\]\_\[ID1\]\[ID2\] \-\> Triggered when two corners have colors facing the identical side. E.g., bco2\_L\_34 (Corners 3 and 4 have bottom colors facing the Left side).

### **BOTTOM\_SIDE\_LOC\_SOLUTION (Step 6\)**

Handles Bottom Edge Positioning.

* **Key Format:** bsl\_\[CorrectID\]\_\[LeftAdjacentID\]  
* **Explanation:** bsl\_1\_4 triggers when Edge 1 is in the right spot, and Edge 4 is currently sitting physically in Edge 1's left-adjacent slot (which belongs to Side 4).

### **BOTTOM\_SIDE\_ORIENT\_SOLUTION (Step 6\)**

Handles Bottom Edge Twisting/Flipping once in the correct slot.

* **Key Format:** bso\_\[UnsolvedIDs\]  
* **Explanation:** bso\_24 triggers when Edge 2 and Edge 4 are in their proper slots, but their colors are flipped. bso\_1234 triggers when all 4 are flipped.

## **9\. Global Event Listeners & Chat Parser**

### **addLog(message, sender)**

* Creates a new DOM div block. Applies styling prefixes (\[C3D\] for system, \> for user). Injects the message, appends the node to the \#chatLog container, and auto-scrolls to the bottom.

### **chatInput.addEventListener('keydown')**

1. Triggers exclusively when 'Enter' is pressed inside the terminal.  
2. Checks if any pending tracker flags (pendingMissingSideKey, pendingMissingBSOKey, etc.) are active waiting for macro definition:  
   * If 'yes'/'y', binds the recorded macro string to the appropriate global lookup table dictionary.  
3. Assesses input against strict Regex to see if it represents raw move inputs (e.g., "02:01") and pipes them to executeShorthandSequence().  
4. Evaluates hardcoded text commands (shuffle, reset, view x y z, layer, updatesidetable).

## **10\. Full Table Contents**

This section contains the exact JSON data sets defining every known solution path currently active in the solver.

### **Explanation of Table Elements**

* **Keys (Left Side):** These represent the *identified error state* of a piece or group of pieces. They are generated dynamically by the solver's identification algorithms. They encapsulate information such as the piece's designated ID number, its current \[X, Y, Z\] coordinate string, and which axis its primary color is currently facing.  
* **Values (Right Side):** These represent the sequence of moves required to solve that exact configuration. Sequences are formatted as comma-separated 5-character shorthand strings (e.g. 02:11, 20:01), which map perfectly to \[Layer\]\[Axis\]:\[Direction\]\[Steps\].

### **CORNER\_SOLUTIONS (Top Corners)**

{  
  "c1\_200:1": "02:11, 20:01, 02:01, 20:11",   
  "c2\_002:0": "00:01, 02:01, 21:01, 02:01, 21:11",   
  "c3\_020:1": "01:11, 02:01, 01:01",  
  "c1\_000:0": "02:01, 01:01, 02:01, 01:11",  
  "c1\_000:1": "02:01, 20:01, 02:11, 20:11",  
  "c1\_000:2": "02:01, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_002:0": "00:01, 02:01, 00:11, 01:01, 02:01, 01:11",  
  "c1\_002:1": "00:01, 02:01, 00:11, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_002:2": "00:01, 02:01, 00:11, 20:01, 02:11, 20:11",  
  "c1\_020:0": "02:02, 20:01, 02:11, 20:11",  
  "c1\_020:1": "02:02, 01:01, 02:01, 01:11",  
  "c1\_020:2": "02:02, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_022:0": "00:11, 02:11, 00:01, 02:02, 02:01, 20:01, 02:11, 20:11",  
  "c1\_022:1": "00:11, 02:11, 00:01, 02:02, 02:01, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_022:2": "00:11, 02:11, 00:01, 02:02, 02:01, 01:01, 02:01, 01:11",  
  "c1\_200:0": "20:01, 02:11, 20:11",  
  "c1\_200:2": "20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_202:0": "20:01, 02:11, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_202:1": "20:01, 02:11, 20:11, 02:01, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_220:0": "02:02, 02:01, 01:01, 02:01, 01:11",  
  "c1\_220:1": "02:02, 02:01, 20:01, 02:11, 20:11",  
  "c1\_220:2": "02:02, 02:01, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_222:0": "20:11, 02:01, 20:01, 02:02, 01:01, 02:01, 01:11",  
  "c1\_222:1": "20:11, 02:01, 20:01, 02:02, 20:01, 02:02, 20:11, 02:01, 20:01, 02:11, 20:11",  
  "c1\_222:2": "20:11, 02:01, 20:01, 02:02, 20:01, 02:11, 20:11",  
  "c2\_000:0": "02:02, 20:11, 02:01, 20:01",  
  "c2\_000:1": "02:02, 21:01, 02:11, 21:11",  
  "c2\_000:2": "02:02, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_002:1": "00:01, 02:01, 00:11, 02:01, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_002:2": "00:01, 02:01, 00:11, 02:01, 21:01, 02:11, 21:11",  
  "c2\_020:0": "02:02, 02:01, 21:01, 02:11, 21:11",  
  "c2\_020:1": "02:02, 02:01, 20:11, 02:01, 20:01",  
  "c2\_020:2": "02:02, 02:01, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_022:0": "00:11, 02:11, 00:01, 21:01, 02:11, 21:11",  
  "c2\_022:1": "00:11, 02:11, 00:01, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_022:2": "00:11, 02:11, 00:01, 20:11, 02:01, 20:01",  
  "c2\_200:0": "02:01, 21:01, 02:11, 21:11",  
  "c2\_200:1": "02:01, 20:11, 02:01, 20:01",  
  "c2\_200:2": "02:01, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_202:0": "20:01, 02:11, 20:11, 02:02, 21:01, 02:11, 21:11",  
  "c2\_202:1": "20:01, 02:11, 20:11, 02:02, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_202:2": "20:01, 02:11, 20:11, 02:02, 20:11, 02:01, 20:01",  
  "c2\_220:0": "20:11, 02:01, 20:01",  
  "c2\_220:1": "21:01, 02:11, 21:11",  
  "c2\_220:2": "20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c2\_222:0": "20:11, 02:01, 20:01, 02:02, 02:01, 20:11, 02:01, 20:01",  
  "c2\_222:1": "20:11, 02:01, 20:01, 02:02, 02:01, 20:11, 02:02, 20:01, 02:11, 20:11, 02:01, 20:01",  
  "c3\_000:0": "00:01, 02:01, 00:11",  
  "c3\_000:1": "01:11, 02:11, 01:01",  
  "c3\_000:2": "00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_002:0": "00:01, 02:01, 00:11, 02:02, 02:01, 00:01, 02:01, 00:11",  
  "c3\_002:1": "00:01, 02:01, 00:11, 02:02, 02:01, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_020:0": "02:01, 01:11, 02:11, 01:01",  
  "c3\_020:2": "02:01, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_022:0": "00:11, 02:11, 00:01, 02:02, 01:11, 02:11, 01:01",  
  "c3\_022:1": "00:11, 02:11, 00:01, 02:02, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_022:2": "00:11, 02:11, 00:01, 02:02, 00:01, 02:01, 00:11",  
  "c3\_200:0": "02:02, 02:01, 01:11, 02:11, 01:01",  
  "c3\_200:1": "02:02, 02:01, 00:01, 02:01, 00:11",  
  "c3\_200:2": "02:02, 02:01, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_202:0": "20:01, 02:11, 20:11, 01:11, 02:11, 01:01",  
  "c3\_202:1": "20:01, 02:11, 20:11, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_202:2": "20:01, 02:11, 20:11, 00:01, 02:01, 00:11",  
  "c3\_220:0": "02:02, 00:01, 02:01, 00:11",  
  "c3\_220:1": "02:02, 01:11, 02:11, 01:01",  
  "c3\_220:2": "02:02, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_222:0": "20:11, 02:01, 20:01, 02:01, 00:01, 02:01, 00:11",  
  "c3\_222:1": "20:11, 02:01, 20:01, 02:01, 00:01, 02:02, 00:11, 02:11, 00:01, 02:01, 00:11",  
  "c3\_222:2": "20:11, 02:01, 20:01, 02:01, 01:11, 02:11, 01:01",  
  "c4\_000:0": "02:02, 02:01, 21:11, 02:01, 21:01",  
  "c4\_000:1": "02:02, 02:01, 00:11, 02:11, 00:01",  
  "c4\_000:2": "02:02, 02:01, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_002:0": "00:01, 02:01, 00:11, 02:02, 21:11, 02:01, 21:01",  
  "c4\_002:1": "00:01, 02:01, 00:11, 02:02, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_002:2": "00:01, 02:01, 00:11, 02:02, 00:11, 02:11, 00:01",  
  "c4\_020:0": "00:11, 02:11, 00:01",  
  "c4\_020:1": "21:11, 02:01, 21:01",  
  "c4\_020:2": "00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_022:0": "00:11, 02:11, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_022:1": "00:11, 02:11, 00:01, 02:01, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_200:0": "02:02, 00:11, 02:11, 00:01",  
  "c4\_200:1": "02:02, 21:11, 02:01, 21:01",  
  "c4\_200:2": "02:02, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_202:0": "20:01, 02:11, 20:11, 02:02, 02:01, 00:11, 02:11, 00:01",  
  "c4\_202:1": "20:01, 02:11, 20:11, 02:02, 02:01, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_202:2": "20:01, 02:11, 20:11, 02:02, 02:01, 21:11, 02:01, 21:01",  
  "c4\_220:0": "02:01, 21:11, 02:01, 21:01",  
  "c4\_220:1": "02:01, 00:11, 02:11, 00:01",  
  "c4\_220:2": "02:01, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_222:0": "20:11, 02:01, 20:01, 21:11, 02:01, 21:01",  
  "c4\_222:1": "20:11, 02:01, 20:01, 00:11, 02:02, 00:01, 02:01, 00:11, 02:11, 00:01",  
  "c4\_222:2": "20:11, 02:01, 20:01, 00:11, 02:11, 00:01"  
}

### **TOP\_SIDE\_SOLUTIONS (Top Edges)**

{  
  "s1\_001:0": "12:11, 01:11, 12:01, 01:01",  
  "s1\_001:1": "12:01, 12:01, 01:01, 12:11, 01:11, 12:11",  
  "s1\_010:0": "10:01, 02:01, 10:11",  
  "s1\_010:2": "02:01, 10:01, 02:11, 02:11, 10:11",  
  "s1\_012:0": "00:01, 12:11, 12:11, 00:11, 01:11, 12:01, 12:01, 01:01",  
  "s1\_012:2": "00:01, 12:01, 00:11, 12:01, 01:01, 12:11, 01:11, 12:11",  
  "s1\_021:0": "01:01, 12:11, 12:11, 01:11, 12:11, 12:11",  
  "s1\_021:1": "01:11, 12:01, 01:01, 12:01, 12:01, 12:01",  
  "s1\_100:1": "02:01, 10:01, 02:11, 10:11",  
  "s1\_100:2": "10:01, 02:11, 02:11, 10:11",  
  "s1\_102:1": "01:11, 12:11, 01:01, 01:01, 12:11, 12:11, 01:11, 12:11",  
  "s1\_120:1": "02:01, 10:01, 02:01, 10:11",  
  "s1\_120:2": "10:11, 02:01, 10:01, 10:01, 02:11, 10:11",  
  "s1\_122:1": "21:01, 12:11, 21:11, 12:01, 01:01, 12:11, 01:11, 12:01",  
  "s1\_122:2": "10:11, 02:01, 10:01, 10:01, 02:01, 10:11",  
  "s1\_201:0": "12:01, 01:01, 12:11, 01:11",  
  "s1\_201:1": "12:01, 01:11, 12:11, 12:11, 01:01, 12:01",  
  "s1\_210:0": "10:01, 02:11, 10:11",  
  "s1\_210:2": "02:11, 10:01, 02:01, 02:01, 10:11",  
  "s1\_212:0": "20:01, 12:11, 20:11, 12:01, 12:01, 01:01, 12:11, 01:11",  
  "s1\_212:2": "11:01, 02:01, 11:11, 02:01, 10:01, 02:01, 10:11",  
  "s1\_221:0": "01:11, 12:01, 12:01, 01:01, 12:01, 12:01",  
  "s1\_221:1": "01:01, 12:11, 01:11, 12:01",  
  "s2\_001:0": "20:01, 12:01, 20:11, 12:11",  
  "s2\_001:1": "20:11, 12:11, 12:11, 20:01, 12:11, 12:11",  
  "s2\_010:0": "02:01, 11:01, 02:01, 11:11",  
  "s2\_010:2": "02:01, 02:01, 11:01, 02:01, 02:01, 11:11",  
  "s2\_012:0": "00:01, 12:01, 00:11, 12:11, 20:01, 12:01, 20:11, 12:11",  
  "s2\_012:2": "00:11, 12:11, 00:01, 12:01, 20:01, 12:11, 12:11, 20:11, 12:01, 12:01",  
  "s2\_021:0": "20:11, 12:11, 20:01, 12:01",  
  "s2\_021:1": "20:01, 12:01, 12:01, 20:11, 12:01, 12:01",  
  "s2\_100:1": "11:01, 02:01, 11:11",  
  "s2\_100:2": "02:01, 11:01, 02:11, 02:11, 11:11",  
  "s2\_120:1": "11:01, 02:11, 11:11",  
  "s2\_120:2": "02:11, 11:01, 02:11, 02:11, 11:11",  
  "s2\_122:1": "21:01, 12:01, 12:01, 21:11, 20:11, 12:11, 12:11, 20:01",  
  "s2\_122:2": "10:11, 02:11, 10:01, 02:01, 11:01, 02:11, 11:11",  
  "s2\_201:0": "12:01, 12:01, 20:11, 12:11, 20:01, 12:11",  
  "s2\_201:1": "12:11, 20:01, 12:01, 20:11",  
  "s2\_210:0": "02:11, 11:01, 02:01, 11:11",  
  "s2\_210:2": "11:01, 02:11, 02:11, 11:11",  
  "s2\_212:0": "20:01, 12:11, 20:11, 20:11, 12:11, 12:11, 20:01, 12:11",  
  "s2\_221:0": "12:11, 12:11, 20:01, 12:01, 20:11, 12:01",  
  "s2\_221:1": "12:01, 20:11, 12:11, 20:01",  
  "s3\_001:0": "21:01, 12:01, 12:01, 21:11, 12:01, 12:01",  
  "s3\_001:1": "21:11, 12:11, 21:01, 12:01",  
  "s3\_010:0": "10:11, 02:11, 10:01",  
  "s3\_010:2": "02:11, 10:11, 02:11, 02:11, 10:01",  
  "s3\_012:0": "00:11, 12:11, 00:01, 12:01, 12:01, 21:11, 12:11, 21:01",  
  "s3\_012:2": "11:11, 02:11, 11:01, 02:11, 10:11, 02:01, 10:01",  
  "s3\_021:0": "12:01, 21:11, 12:11, 21:01",  
  "s3\_021:1": "12:01, 21:01, 12:01, 12:01, 21:11, 12:11, 12:11, 12:11",  
  "s3\_100:1": "02:11, 10:11, 02:11, 10:01",  
  "s3\_100:2": "02:01, 02:01, 10:11, 02:01, 02:01, 10:01",  
  "s3\_120:1": "02:01, 10:11, 02:11, 10:01",  
  "s3\_120:2": "10:11, 02:11, 02:11, 10:01",  
  "s3\_122:1": "21:11, 12:01, 21:01, 21:01, 12:01, 12:01, 21:11, 12:01",  
  "s3\_201:0": "21:11, 12:11, 12:11, 21:01, 12:11, 12:11",  
  "s3\_201:1": "21:01, 12:01, 21:11, 12:11",  
  "s3\_210:0": "10:11, 02:01, 10:01",  
  "s3\_210:2": "02:01, 10:11, 02:11, 02:11, 10:01",  
  "s3\_221:0": "12:11, 21:01, 12:01, 21:11",  
  "s3\_221:1": "12:01, 12:01, 21:11, 12:11, 21:01, 12:11",  
  "s4\_001:0": "12:11, 12:11, 00:11, 12:01, 00:01, 12:01",  
  "s4\_001:1": "12:01, 00:01, 12:11, 00:11",  
  "s4\_010:0": "02:01, 11:11, 02:11, 11:01",  
  "s4\_010:2": "11:11, 02:01, 02:01, 11:01",  
  "s4\_012:0": "00:01, 12:01, 00:11, 00:11, 12:01, 12:01, 00:01, 12:01",  
  "s4\_021:0": "12:11, 00:01, 12:11, 12:11, 00:11, 12:11",  
  "s4\_021:1": "12:11, 00:11, 12:01, 00:01",  
  "s4\_100:1": "11:11, 02:11, 11:01",  
  "s4\_100:2": "02:11, 11:11, 02:01, 02:01, 11:01",  
  "s4\_120:1": "11:11, 02:01, 11:01",  
  "s4\_120:2": "02:01, 11:11, 02:01, 02:01, 11:01",  
  "s4\_122:1": "02:01, 11:11, 02:11, 02:11, 11:01",  
  "s4\_201:0": "00:01, 12:11, 00:11, 12:01",  
  "s4\_201:1": "00:11, 12:01, 12:01, 00:01, 12:01, 12:01",  
  "s4\_210:0": "02:11, 11:11, 02:11, 11:01",  
  "s4\_210:2": "02:11, 02:11, 11:11, 02:11, 02:11, 11:01",  
  "s4\_221:0": "00:11, 12:01, 00:01, 12:11",  
  "s4\_221:1": "00:01, 12:11, 12:11, 00:11, 12:01, 12:01"  
}

### **EDGE\_SIDE\_SOLUTION (Mid Edges)**

{  
  "m1\_001:0": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:01, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_001:1": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:11, 02:11, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_010:0": "20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_010:2": "02:11, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_021:0": "21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01, 02:01, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_021:1": "21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01, 02:11, 02:11, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_100:1": "02:11, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_100:2": "02:01, 02:01, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_120:1": "02:01, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_120:2": "01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_201:0": "20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11, 02:01, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_210:0": "02:11, 02:11, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m1\_210:2": "02:01, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_221:0": "21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01, 02:11, 01:01, 02:11, 01:11, 02:11, 20:01, 02:01, 20:11",  
  "m1\_221:1": "21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01, 20:01, 02:01, 20:11, 02:01, 01:01, 02:11, 01:11",  
  "m2\_001:0": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:11, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_001:1": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:01, 02:01, 20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m2\_010:0": "20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m2\_010:2": "02:01, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_021:0": "21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01, 02:11, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_021:1": "00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_100:1": "02:11, 20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m2\_100:2": "21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_120:1": "02:01, 20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m2\_120:2": "02:11, 02:11, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_210:0": "02:01, 02:01, 20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m2\_210:2": "02:11, 21:01, 02:01, 21:11, 02:01, 20:11, 02:11, 20:01",  
  "m2\_221:0": "20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11, 02:01, 02:01, 02:01, 20:11, 02:11, 20:01, 02:11, 21:01, 02:01, 21:11",  
  "m3\_001:0": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:11, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m3\_001:1": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01",  
  "m3\_010:0": "02:11, 02:11, 00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01",  
  "m3\_010:2": "02:01, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m3\_021:0": "21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01, 02:11, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m3\_100:1": "02:01, 00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01",  
  "m3\_100:2": "02:01, 02:01, 02:01, 02:01, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m3\_120:1": "02:11, 00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01",  
  "m3\_120:2": "02:01, 02:01, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m3\_210:0": "00:11, 02:01, 00:01, 02:01, 21:11, 02:11, 21:01",  
  "m3\_210:2": "02:01, 02:01, 02:01, 21:11, 02:11, 21:01, 02:11, 00:11, 02:01, 00:01",  
  "m4\_001:0": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11, 02:01, 01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11",  
  "m4\_010:0": "02:01, 02:01, 00:01, 02:11, 00:11, 02:11, 01:11, 02:01, 01:01",  
  "m4\_010:2": "02:11, 01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11",  
  "m4\_100:1": "02:01, 00:01, 02:11, 00:11, 02:11, 01:11, 02:01, 01:01",  
  "m4\_100:2": "02:11, 02:11, 01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11",  
  "m4\_120:1": "02:11, 00:01, 02:11, 00:11, 02:11, 01:11, 02:01, 01:01",  
  "m4\_120:2": "01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11",  
  "m4\_210:0": "00:01, 02:11, 00:11, 02:11, 01:11, 02:01, 01:01",  
  "m4\_210:2": "02:01, 01:11, 02:01, 01:01, 02:01, 00:01, 02:11, 00:11"  
}

### **BOTTOM\_CORNER\_LOC\_SOLUTION (Bottom Corners \- Location)**

{  
  "bcl1\_000": "20:01, 02:11, 20:11, 01:01, 02:01, 01:11, 20:01, 02:01, 20:11, 02:11, 02:11",  
  "bcl1\_020": "00:11, 02:11, 00:01, 21:11, 02:01, 02:01, 21:01, 00:11, 02:01, 00:01, 02:11",  
  "bcl1\_220": "21:01, 02:11, 21:11, 20:11, 02:01, 20:01, 21:01, 02:01, 21:11, 02:11, 02:11",  
  "bcl2\_000": "21:01, 02:11, 21:11, 20:11, 02:01, 02:01, 20:01, 21:01, 02:01, 21:11, 02:11",  
  "bcl2\_020": "00:11, 02:11, 00:01, 21:11, 02:01, 21:01, 00:11, 02:01, 00:01, 02:11, 02:11",  
  "bcl3\_020": "01:11, 02:11, 01:01, 00:01, 02:01, 00:11, 01:11, 02:01, 01:01, 02:11, 02:11"  
}

### **BOTTOM\_CORNER\_ORIENT\_SOLUTION (Bottom Corners \- Orientation)**

{  
  "bco1\_S1": "21:01, 02:11, 21:11, 02:11, 21:01, 02:11, 02:11, 21:11, 02:11, 02:11",  
  "bco1\_S2": "00:11, 02:11, 00:01, 02:11, 00:11, 02:11, 02:11, 00:01, 02:11, 02:11",  
  "bco1\_S3": "20:01, 02:11, 20:11, 02:11, 20:01, 02:11, 02:11, 20:11, 02:11, 02:11",  
  "bco1\_S4": "01:11, 02:11, 01:01, 02:11, 01:11, 02:11, 02:11, 01:01, 02:11, 02:11",  
  "bco2\_B\_24": "01:11, 02:11, 01:01, 02:11, 01:11, 02:11, 02:11, 01:01, 02:11, 02:11",  
  "bco2\_F\_13": "21:01, 02:11, 21:11, 02:11, 21:01, 02:11, 02:11, 21:11, 02:11, 02:11",  
  "bco2\_L\_34": "20:01, 02:11, 20:11, 02:11, 20:01, 02:11, 02:11, 20:11, 02:11, 02:11",  
  "bco2\_R\_12": "00:11, 02:11, 00:01, 02:11, 00:11, 02:11, 02:11, 00:01, 02:11, 02:11"  
}

### **BOTTOM\_SIDE\_LOC\_SOLUTION (Bottom Edges \- Location)**

{  
  "bsl\_1\_2": "10:01, 02:01, 10:11, 02:01, 02:01, 10:01, 02:01, 10:11",  
  "bsl\_1\_3": "10:01, 02:11, 10:11, 02:11, 02:11, 10:01, 02:11, 10:11",  
  "bsl\_2\_3": "11:01, 02:01, 11:11, 02:01, 02:01, 11:01, 02:01, 11:11",  
  "bsl\_2\_4": "11:01, 02:11, 11:11, 02:11, 02:11, 11:01, 02:11, 11:11",  
  "bsl\_3\_1": "10:11, 02:11, 10:01, 02:11, 02:11, 10:11, 02:11, 10:01",  
  "bsl\_3\_4": "10:11, 02:01, 10:01, 02:01, 02:01, 10:11, 02:01, 10:01",  
  "bsl\_4\_1": "11:11, 02:01, 11:01, 02:01, 02:01, 11:11, 02:01, 11:01",  
  "bsl\_4\_2": "11:11, 02:11, 11:01, 02:11, 02:11, 11:11, 02:11, 11:01"  
}

### **BOTTOM\_SIDE\_ORIENT\_SOLUTION (Bottom Edges \- Orientation)**

{  
  "bso\_12": "20:01, 12:01, 20:11, 20:11, 12:01, 12:01, 20:01, 02:01, 20:11, 12:11, 12:11, 20:11, 20:11, 12:11, 20:11, 02:11",  
  "bso\_1234": "01:11, 12:01, 01:01, 01:01, 12:01, 12:01, 01:11, 02:01, 01:01, 12:11, 12:11, 01:11, 01:11, 12:11, 01:01, 02:11",  
  "bso\_13": "01:11, 12:01, 01:01, 01:01, 12:01, 12:01, 01:11, 02:01, 02:01, 01:01, 12:11, 12:11, 01:01, 01:01, 12:11, 01:01, 02:11, 02:11",  
  "bso\_14": "01:11, 12:01, 01:01, 01:01, 12:01, 12:01, 01:11, 02:01, 01:01, 12:11, 12:11, 01:01, 01:01, 12:11, 01:01, 02:11",  
  "bso\_23": "21:01, 12:01, 21:11, 21:11, 12:01, 12:01, 21:01, 02:01, 21:11, 12:11, 12:11, 21:11, 21:11, 12:11, 21:11, 02:11",  
  "bso\_24": "00:11, 12:01, 00:01, 00:01, 12:01, 12:01, 00:11, 02:01, 02:01, 00:01, 12:11, 12:11, 00:01, 00:01, 12:11, 00:01, 02:11, 02:11",  
  "bso\_34": "00:11, 12:01, 00:01, 00:01, 12:01, 12:01, 00:11, 02:01, 00:01, 12:11, 12:11, 00:01, 00:01, 12:11, 00:01, 02:11"  
}  
