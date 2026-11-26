# DevBlog #3: Map and Pathfinding

**Project:** AR Campus Navigation Assistant

## Introduction

In the first devblog we looked at localization, so the app knows where the phone is in our campus coordinate system. The second one was about the AR guidance and the UI, which is the arrows and screens the user actually sees. This post is about the part that sits in between them, which is figuring out where the hallways are and which way the user should walk to get to their destination.

My part of the project is the map and the pathfinding. Simply put, my code gets a start position and a destination, and gives back a route as a list of points in campus coordinates. The guidance part then uses those points to place the arrows on the floor.

I decided early on that I wanted the pathfinding to be plain C# without anything from AR Foundation or the scene. That meant I could work on it and test it on my laptop without having a phone connected, and the others in the group could start using it before we had a real map of the hallway.

## The campus as a graph

I chose to describe the campus as a graph. Every important point is a node, so that could be the entrance, a corner, a place where two hallways meet, a marker or a room you can navigate to. The hallways between them are edges, and every edge has a length in meters.

For this I made three small classes. `CampusNode` holds an id, a name that is shown to the user, a position, whether it can be picked as a destination, and optionally the name of a marker placed there. `CampusEdge` connects two nodes, and `CampusGraph` builds the actual graph and keeps track of which nodes are neighbours.

All edges go both ways since you can obviously walk both directions in a hallway. If I don't give an edge a distance, it just uses the direct distance between the two nodes. You can also set the distance manually, which is meant for cases where the real walk is longer, for example if you have to go around a door.

The graph also checks its own data when it is built. If two nodes have the same id, if an id is empty or if an edge points to a node that doesn't exist, it throws an error straight away. I added this after thinking about how annoying it would be to have a small typo in an id and then only find out much later when a route just doesn't work.

To make it possible to edit the map without changing any code, I made `CampusMapData`, which is a ScriptableObject where you can add nodes and edges in the Inspector in Unity.

## Finding the shortest route

For the search I used Dijkstra's algorithm, which I implemented in `PathfindingService`. Our map is really small, so I went with the simple version without a priority queue, where it just goes through the unvisited nodes and picks the closest one each time. With so few nodes it is more than fast enough, and the code is a lot easier to read.

One thing I did was to run the search from the destination instead of from the start. Because the edges go both ways, every node then ends up knowing which neighbour is one step closer to the destination. So with one search I can read the route from any starting point, and that turned out to be really useful later when the user is not standing on a node.

When something goes wrong, the service doesn't just return null. It returns a `PathResult` with a status that says whether it worked, whether the start or the destination wasn't found, or whether the destination can't be reached at all. When it works, it contains the nodes on the route, the waypoints and the total distance. That way the UI can actually tell the user why there is no route.

Before we had a real map, I made a small mock map with a hallway, a couple of junctions and three rooms, so the others had something to build against. I also put in a storage room that isn't connected to anything, just so I could test that the code says the room can't be reached instead of crashing.

## Routing from where the user actually is

My first version could only find a route from one node to another. But in reality the user is almost never standing exactly on a node. They could be in the middle of a hallway, or they might have walked past the turn they were supposed to take.

So I added `FindPathFromPosition`. It finds the closest point on the nearest hallway and then checks which end of that hallway gives the shortest way to the destination. If the user has walked past a turn, it sends them back to the turn instead of further down the hallway. That closest point is also added as the first waypoint, so the arrows start where the user is and not at some node a few meters away.

I only measure distances on the floor here and ignore the height. The phone is held about 1.5 meters above the floor, and I didn't want that to have any effect on which hallway the user gets matched to.

I also wrote `RouteSampler`, which splits a route up into points with an even distance between them. A route might only have a few corners, but the guidance part needs an arrow every meter or so. The sampler keeps all the corners so the route keeps its shape and just adds extra points in between.

Last, there is `PathfindingManager`, which is the component you put in the scene. It loads a map and lets the other parts use the pathfinding through an interface. If no map has been assigned it uses the mock map and writes a warning in the console, which made it easy to test the UI in a scene without any map data.

## The proof of concept map

When the localization part was ready, I made `PocCampusMap`, which is built around the real positions of the two markers, `EntranceMarker` and `HallwayCheckpoint`. The idea is that the marker nodes are the spot on the floor right below the physical markers, so my part and the localization part use the exact same coordinates.

The map has three destinations, which are the XR Lab, a classroom and the reception. In my first version they were spread out over about ten meters, but when we put everything together the map was made smaller, so the whole route fits inside the area where we can actually test. That is why the final map is only a couple of meters across.

I also wrote tests for the map itself. They check that it can be built, that the marker nodes are in the same place as the markers, that every destination can be reached from both the entrance and the checkpoint, and that the route to the XR Lab goes past the checkpoint. So if someone changes the map in the Inspector and accidentally breaks a connection, the tests will catch it.

## Testing

Because my code doesn't depend on the scene or AR, nearly all of it could be tested with EditMode tests in Unity's Test Runner. In the end I had around 50 test cases covering the graph, the pathfinding, routing from a position, the sampler, the manager and the map.

Some of the ones I found most useful were:

* It picks the shortest route, even if that route has more steps
* The route from A to B is the same as B to A, just reversed
* Standing past a turn sends you back to the turn
* Unknown or unreachable rooms give the right status

Having the tests early also helped a lot when the coordinate system got changed later on, because I could quickly see that the routing still worked the same way.

The tests only show that the logic works on its own though. To see everything working together we also tested the app on an Android phone, where the route from my part was shown as arrows on the floor after scanning the entrance marker.

## Challenges

The biggest challenge was making sure we all used the same coordinates. The localization part decides the coordinate system, and my map has to use the exact same starting point, directions and units. If the map is rotated or moved just a bit compared to the markers, it doesn't matter how correct it is on its own. Using the marker names in the nodes and testing that the positions match made it a lot easier to keep things in sync.

Another challenge was making my part usable for the others before it was done. The interface, the mock map and the fallback in the manager meant that the guidance and the UI could be built at the same time without having to wait for me.

## Reflection and next steps

I learned that the algorithm itself was actually the easy part. Dijkstra is well known, and on a map this small speed is not a problem. The harder part was all the practical stuff around it, like what to do when the user is between two nodes, how to report errors in a useful way and how to keep the map matching the real world.

I think keeping the pathfinding separate from the scene and AR was a good choice. It made it easier to test and easier for the others to build on top of.

The next step would be to measure a real hallway on campus and replace the proof of concept map with real positions, including where the markers are and which way they face. So far we have only tested on the phone with the small map, so a longer route in a real hallway would show if the arrows stay lined up over a longer distance.

The map is only an example and not measured from a real building, and the app is a small demo and not something you should rely on for real navigation.
