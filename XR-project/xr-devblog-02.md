# DevBlog #2: Scenario Flow and Assessment

**Project:** VR Fire Safety Training Simulator

## Introduction

The first devblog was about the office, the hazards and the fire effects. Those things give the user something to look at and interact with, but on their own they don't make a training session. Something has to decide what happens first, when the fire starts, what choices the user has and what feedback they get at the end.

That is my part of the project. I made the state machine that controls the training, the rules of the scenario, how the safe and unsafe choices are handled, the log of everything that happens during a session and the results at the end. I didn't make the VR controls, the environment or the UI. Those parts talk to my part and react to what it says.

## Planning the flow

Before I wrote any code I split the session up into phases. It starts with a short introduction, and then the user walks around the office looking for hazards. When the inspection is done, the fire starts. From there the user can go straight to evacuating, or if the scenario allows it, practise with the fire extinguisher first and then evacuate. No matter which way they go, the session ends when they reach the assembly area outside.

Something that was important to me was that you should never be able to get stuck. People do things in strange orders in VR. They might pick up the extinguisher while they are still looking for hazards, hit the same hazard twice, or pause right in the middle of the fire. All of those things needed to have a clear and predictable result.

## The state machine

The main part of my code is `TrainingStateMachine`. The other parts of the project never change the phase directly. Instead they send commands like start training, confirm a hazard, choose to evacuate, choose to practise or tell it that the user reached the assembly area. The state machine then decides if that command is allowed right now.

Every command gets written to a log together with what happened to it. It could be accepted, or it could be rejected because it was a duplicate, came in the wrong phase, came while paused, used an unknown id or wasn't allowed. If a command is rejected, nothing changes. So doing things in the wrong order or twice can't break the scenario, and when something didn't happen during testing I could just look in the log to see why.

Along the way I ended up adding a bunch of rules. If the user finds all three hazards the fire starts by itself, but they can also choose to finish early, and the hazards they didn't find are then shown as missed. Picking up the extinguisher during the fire counts as choosing to practise, so you don't have to press a button first. If practising isn't allowed in the scenario and the user tries anyway, it is logged as an unsafe choice and they get sent to evacuation. Using the extinguisher outside of the practice while the fire is going always counts as unsafe. If the user just walks to the assembly area without choosing anything, that counts as evacuating. And you can always stop practising and evacuate instead, because evacuating should never be the wrong answer.

There are no timers inside the state machine, so the same commands in the same order always give the same result. Time is only used to note down when things happened, and the clock is passed in from outside so I could control it in the tests.

I also kept all of this in its own assembly that doesn't use anything from Unity. That way the rules can be tested without a headset and without even opening a scene.

## Connecting it to the rest of the project

To use the state machine in Unity I made `ScenarioCoordinator`, which is a component in the `OfficeTraining` scene. It connects my part to the environment from the first devblog. While the user is inspecting, the hazards can be clicked and there is no fire. When the fire starts, hazard inspection is turned off and the fire is active. During evacuation the fire is shown as put out if the practice went well, and otherwise it keeps going and the user has to get out. This fits with how the environment was designed, where the scenario decides when the fire starts and stops and not the environment itself. Pausing and resetting are also passed on to the environment, so the whole scene resets together.

The interaction part tells my part what the user does, like confirming a hazard, picking up or using the extinguisher, finishing the practice and walking into the assembly area. The UI uses my commands for its buttons and listens for events, for example when the phase changes, when the game is paused or when the final report is ready. Everything can also be hooked up with UnityEvents in the Inspector, which made it easier for the others to connect things without writing extra code.

Settings like whether practising is allowed and how many hazards you need to find are stored in a `ScenarioConfig` asset, so they can be changed without touching the code.

## Assessment and feedback

I decided not to give the user a score. If you get told you got 75 points, that doesn't really tell you what you did well or what you should do differently. So instead the report goes through four goals. Did you find the hazards, did you make a safe choice when the fire started, did you finish the practice if you chose it, and did you reach the assembly area. For each goal it shows what was needed and a short message about how it went.

The report also shows which hazards you found and missed, which path you took, how the practice went, how many unsafe actions there were, how long the session took and the full log.

All the text the user sees is in one file, `FeedbackText.cs`. I did that so all the wording can be checked in one place against official fire safety guidelines. I tried to keep the text neutral, so it just describes what happened in the simulation and doesn't pretend to be professional emergency advice.

## Testing

Since the core doesn't use Unity, I could test it properly with EditMode tests. There are around 50 test cases split over three test classes. The first one tests the state machine itself, so things like a normal run, evacuating straight away, unsafe actions, the practice path, wrong order, duplicates, pausing, ending early and resetting. The second tests the rules and when each goal counts as achieved. The third connects the coordinator to the real hazard and fire scripts from the environment and checks that hazards can only be clicked at the right time, that the fire follows the phase and that pause and reset reach the environment.

I also made a menu in the editor called Run Mock Simulations, which runs six sessions that are scripted in advance:

* A normal run where everything goes as planned
* The user evacuates right away
* Unsafe use of the extinguisher
* Wrong order, duplicates and pausing
* A reset in the middle of the session
* A session that is ended early

Each of them prints the full report and log in the console. It was a quick way to show the others what the reports look like before the UI was done.

When all the parts were put together, we tested the whole scenario on a VR headset. That was the first time we could try the full flow the way a user would, from looking for hazards to the fire starting, choosing between evacuating and practising with the extinguisher, and ending at the assembly area.

## Challenges

The hardest part was deciding what should happen in the weird situations. What if the user picks up the extinguisher in a scenario where practising isn't allowed? What if they walk to the assembly area while they are still supposed to be choosing? Those are just as much design questions as code questions, so I tried to write each decision down in the README so the group could agree on it.

Another challenge was that the interaction part and the UI were being built at the same time as mine, so we had to agree on names early. The extinguisher id is one example. If there is no hazard registry in the scene, my coordinator just uses the three default hazards, so the UI could be built in a test scene before everything was connected.

## Reflection and next steps

This part showed me how useful a state machine is when a lot of different systems need to agree on what is going on. Having one place that decides the phase and logs every request made putting it all together a lot smoother than if every part had kept track of the progress by itself.

I also found out that the assessment was more about design than code. Writing clear goals and feedback took more thinking than actually writing the checks.

We have tested it on a headset ourselves, but the next step would be to let people outside the group try it, to see if the flow feels natural and if the feedback makes sense to someone who didn't build it. Whether practising should be allowed and all the feedback text also need to be checked against proper local fire safety guidelines.

This is a student prototype and not certified fire safety training.
