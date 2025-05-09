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

The Game Being an ***RTS*** had to have some though put into how we would handle the Units, furthermore our Game Designers wanted a ***Squad System*** which needed to be taken into account

### Basic Hierarchy

<img width="596" alt="BasicHierarchy" src="https://github.com/user-attachments/assets/906bb0f1-37e1-48b8-9cdb-e3c3b604d63f" />

Each **Squad** is made of a Squad Object that will handle, Initialisation from a DA (which includes spawning Units), Squad Wide Orders, Squad Spacing.
it contains a list of all the Unit It spawnned durring its initialisation phase.
It also contains a Command Queue and a behaviour tree to handle Squad Wide Behaviour such as Aggro of surrounding squad and coordination between all units.

Each **Unit** Is spawned by the squad and also contains a Behaviour tree and a Command Queue that is used to coordinates with Squad Wide Commands. it contains a Movement Component that handles the movement logic, for Performance concerns we use a mix of the Floating Pawn Movement Component and snapping the Units to the Navmesh since we dont need to directly simulate physics/gravity for each of our units.
The unit closest to the Squad position average is selected to become its **Sergent**, wich handles, Squad Wide Aggro Collision, Fog of War Update, it also tunes the other units velocities to be in sync with the sergent and have more coherent squad wide movements.
If a Sergent Dies the closest unit becomes sergent.

The **Units** Are spawned according to a **Formation** Struct given by the Squad Data Asset that contains all the informations to spawn the Units, including a 2D array of unit types and stats.

<img width="350" alt="grid" src="https://github.com/user-attachments/assets/126fce57-8b8d-4cb6-9acf-f1356a5fa2d7" />

Overall we have a lot of different systems that need to communicate with one another, we also have a lot of abstraction because of the Behaviour tree and Command Systems for both **Units** and **Squad**.
To help minimise the effect on performance of the behaviour trees we try to use as much Events as possible and keep the overhead to a minimum.


[![RTS Fight](https://img.youtube.com/vi/4p4HSr-GatU.jpg)](https://www.youtube.com/watch?v=4p4HSr-GatU)

https://youtu.be/4p4HSr-GatU

### Fog of War


[![Fog Of War Showcase Video](https://img.youtube.com/vi/UsWkpj-7U0M/0.jpg)](https://www.youtube.com/watch?v=UsWkpj-7U0M)

^Video Showcase^

Our game also needed a Fog of War, wich we implemented with a RenderTarget onto wich Each Squad's sergent Draws a transparent circular brush periodically.
Each sergent fires a Line trace on a Plane under the map to get the UV coordinates onto which to draw.
This render Target is then sampled by a Decal Actor that covers the whole map.

As for the Ennemy units they periodicaly query the Render Target at their corresponding UV coordinates to know if they are in the Fog of War.
This allows us to hide Unit Meshes and animation as well as interactive behaviour if they are in the Fog Of War which allows us to gain a bit of performance.
( The query is done Asyncronously as the basic Syncronous function is quite expensive )

<img width="600" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/215993c9-cbdb-4621-9292-cb76fdcfb00c" />


(In Testing this system was also able to make an accurate Minimap, but this feature was cut from the game)

## Card System

![Card](https://github.com/user-attachments/assets/5d2df46a-7566-4b99-9ff1-1abc68c6d55a)


When I designed the card system, the precise role and behaviour of the cards had yet to be fully designed.
I opted to make a the Card System very modular so that any changes could quicly be added, and to allow our GDs to churn out New Cards as fast as possible in later production and testing phases.

The card would be containers for a few List of Card Effects that would trigger on certain Events ( OnDraw(), OnPlay(), OnDiscard etc... )
By doing this our Game Designer would be able to make any effect happen at any corresponding Trigger.


<img width="600" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/e4099d20-2b52-4961-984f-d2220f7fff98" />
<img width="400" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/3b386482-9d05-424d-8e6a-165bee03e4ca" />


Each **Card** is made of OnExecution(), OnFinish() and potentially OnTick(). But it also contains a Condtition() Method that is called first, first the Card goes through every Card Effect of a SelectedList and Comfirms every conditions. if all are comfirmed it then Executes its ExecutionMethods.

![CardEffectCallOrder](https://github.com/user-attachments/assets/7822cb96-b927-42bf-8d3d-82926d88e267)

This would prove quite useful for integrating, SFX/VFX.
Combined with already made versatile utility functions it allowed out Designers and Tech Artist to experiment with the system and be more creative all while not needing to have a programmer involved since it would not affect the broader codebase.



## Optimisation 

```
void UAC_M_SquadUnitManager::StartAsyncTask()

{
	AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, [this]()
	{
		this->DoAsyncTask();

		AsyncTask(ENamedThreads::GameThread, [this]()
		{
			GetWorld()->GetTimerManager().SetTimerForNextTick([this]()
			{
				this->OnAsyncCompleted();
			});
		});
		
	});
}
```

