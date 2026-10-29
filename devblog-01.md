# DevBlog #1 – Getting Started with AR Campus Navigation

**Project:** AR Campus Navigation Assistant  
**Author:** Alperen (344133)
**Date:** 29 October 2026

## Introduction

For our AR project, our group decided to develop an indoor campus navigation application using Unity and AR Foundation. The idea is to help new students and visitors find their way around campus by displaying virtual directions through their phone camera.

The application will allow users to scan a marker at an entrance, select a destination, and follow AR arrows to their destination. Additional checkpoint markers will help correct the user's position if tracking becomes inaccurate.

We divided the project into four main parts: localization and AR tracking, map and pathfinding, navigation logic, and AR guidance/UI.

My responsibility is **localization and AR tracking**, which means making sure the application knows where the user is in relation to our predefined campus coordinate system.

## Setting Up the AR Project

I started by creating a new Unity AR project and configuring it for Android. I used AR Foundation and ARCore for tracking the real-world environment through my phone's camera.

The first step was making sure the application could run on my Android device and detect physical surfaces. After testing the initial setup, I moved on to image tracking using `ARTrackedImageManager`.

I created a reference image library and used a physical product box as my first test marker. Initially, recognition was inconsistent, but after using a clearer image with more visual details, the tracking became more reliable.

I also created a simple cube prefab to visualize when an image was recognized. This helped me verify that the tracking system worked before implementing the actual localization logic.

## Implementing Coordinate Alignment

The next step was connecting recognized images to known positions in our campus coordinate system.

I implemented scripts including `MarkerTracker`, `MarkerMap`, and `CoordinateMapper`. The marker map stores information about each reference marker, including its name, position, and rotation.

When the application recognizes `EntranceMarker`, it uses the marker's tracked position and known coordinates to calculate the alignment between Unity's AR world and our campus coordinate system.

To visualize the result, I implemented a coordinate grid that appears in the AR environment.

I also added a debug display showing the user's current X, Y, and Z coordinates. This allowed me to test whether the coordinates changed correctly when moving around.

During testing, I measured approximately one meter of movement and compared it with the reported coordinates. The result was around 1.05 meters, which was reasonably accurate for our initial prototype.

## Challenges and Testing

One of the biggest challenges was understanding how AR tracking behaves when physical objects move.

During testing, I noticed that moving the physical marker did not automatically move the coordinate grid. At first, I thought this was an error, but I learned that the marker is used to establish a fixed coordinate system rather than continuously controlling the grid.

This is important for our application because navigation arrows should remain in the same physical locations even when the marker is no longer visible.

Another challenge was occasional tracking loss, especially when moving the phone or pointing it at areas without enough recognizable visual details.

I also added a second marker called `HallwayCheckpoint`. Both markers can now be recognized by the application, but the recalibration functionality is not fully working yet.

The application successfully establishes its initial position using `EntranceMarker`, while the next task is making sure it can update its alignment when the checkpoint is detected.

## Reflection and Next Steps

Working on localization has given me a better understanding of how AR Foundation uses image tracking and device movement to position virtual objects in the real world.

I learned that recognizing an image is only the first part of localization. The more challenging part is maintaining a consistent coordinate system and correcting positioning errors over time.

So far, I have successfully implemented the initial marker detection, coordinate alignment, a visual coordinate grid, and a coordinate debug display.

The next step is to finish checkpoint recalibration and test whether the application can correct its position when scanning a second marker.

Once localization is working reliably, it can be integrated with the other project modules to support the actual campus navigation experience.
