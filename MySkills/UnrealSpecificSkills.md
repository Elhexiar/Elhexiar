# My Unreal Engine skills

I first started to experiment with Unreal Engine as a Level Editor for Unreal Tournament 4 with the goal of making maps to play with friends and teachers.
I then started to experiment with its programming aspect in my free time during my studies as most of our first years was done on either Unity or WebEngines.

## Final Year Project

During the concept selection my concept was selected on the condition that I fuse it with another student's concept.
After our Concept was properly selected as one of the Final Year Project Concept, i handed over the Game design aspect to a colleague that could handle this aspect full time for the rest of developement.
Meanwhile I focused on the Tech aspect of the project and importantly the R&D.
We eventualy chose to go for Unreal, One of the reason was a desire from myself and a lot of member of the team to branch out of unity learn how to make games no matter the engine.

Our game Being an ***RTS with Card Game Element*** is System heavy, which needed a well planned architecture phase and a lot of re-evaluation phase.
I decided to focus on being a ***Systems Programmer*** and ***Gameplay Programmer***, delegating The UI Programming and some of the Tooling.


I will do my best to try and explain some of the systems I come up with for our game !

# RTS

The Game Being an ***RTS*** had to have some though put into how we would handle the Units, furthermore our Game Designer Wanted a ***Squad System*** which needed to be taken into account

### Basic Hierarchy

<img width="596" alt="BasicHierarchy" src="https://github.com/user-attachments/assets/906bb0f1-37e1-48b8-9cdb-e3c3b604d63f" />

Each **Squad** is made of a Squad Object that will handle, Initialisation from a DA (which includes spawning Units), Squad Wide Orders, Squad Spacing.
it contains a list of all the Unit It spawnned durring its initialisation phase.
It also contains a Command Queue and a behaviour tree to handle Squad Wide Behaviour such as Aggro of surrounding squad and coordination between all units.

Each **Unit** Is spawned by the squad and also contains a Behaviour tree and a Command Queue that is used to coordinates with Squad Wide Commands. it contains a Movement Component that handles the movement logic, for Performance concerns we use a mix of the Floating Pawn Movement Component and snapping the Units to the Navmesh since we dont need to directly simulate physics/gravity for each of our units.
The unit closest to the Squad position average is selected to become its **Sergent**, wich handles, Squad Wide Aggro Collision, Fog of War Update, it also tunes the other units velocities to be in sync with the sergent and have more coherent squad wide movements.
If a Sergent Dies the closest unit becomes sergent.

The **Units** Are spawned according to a **Formation** Struct given by the Squad Data Asset that contains all the informations to spawn the Units, including a 2D array of unit types and stats.

<img width="291" alt="grid" src="https://github.com/user-attachments/assets/126fce57-8b8d-4cb6-9acf-f1356a5fa2d7" />
