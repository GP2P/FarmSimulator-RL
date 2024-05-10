# FarmSimulator-RL

Developed for a UC Berkeley Research Project "Simulating Group Diversity for Gate-keeping Policies using Game Engine Reinforcement Learning", migrated from the G3P [FarmSimulator](https://github.com/GP2P/FarmSimulator/) AI system template.

Developed with Unreal Engine 5. Using art assets from the Unreal Marketplace. Styled using the [Gamemakin UE5 Style Guide](https://github.com/Allar/ue5-style-guide)

**Note:** The Reinforcement Learning part of this project does not work on macOS. This is a limitation of the Learning Agents beta plugin in UE5.3

## AI System

### Features

Every NPC in the game is controlled by their own behavior tree and has an assigned home building and a workplace building. Each NPC tries to find, reserve, navigate to, and work on their own tasks from a list of available tasks around them filtered by rules defined by their workplace.

### Technical Details

This system is a personal trial of implementing complex AI systems in factory management games. Although it is not as efficient in finding and operating non-random farm tasks compared to centrally controlled AI traditionally used in management games, this system offers great scalability, modularity and customizability since all action definitions are individually separated in each behavior (work/task) and available behaviors for each worker are filtered by each workplace's rules. This creates a boilerplate system that enables mod creators to extend the AI system easily.

- The task management system in-game uses one of Unreal Engine 5's newest features in beta testing - "Smart Objects"
	- An actor that contains a Smart Object component is spawned by Game Objects like farm tiles with matured crops. The Smart Object component contains a Smart Object Definition that contains a Gameplay Behavior Config which contains a Gameplay Behavior Class that offers tasks and can be discovered by AI agents based on their workplace's given filters (e.g. agents working at a farm building can execute behaviors (work/task) that involve a nearby farm tile)
	- Available agents scan around them with a filter provided by their workplace and locate a random but closer object, and tries to reserve the slot. A slot can be claimed by multiple agents but only one reservation will be valid
	- If the reservation succeeded, the AI agent will navigate to the slot and get information from the Smart Object about how to operate it ("behavior")
	- After the AI agent completed the behavior, or during the behavior when a notify completes the behavior early, additional code is triggered to make changes  to the world as a result of the "behavior" of the AI agent
	- After a successful task search and completion, the AI agent searches again in 2 seconds to allow animations to complete etc. If the task searching or completion failed, the AI agent waits for 5 seconds to start another round of searching, to reduce the average performance impact of the search

<img width="755" alt="Screen Shot 2022-06-19 at 11 16 10 PM" src="https://user-images.githubusercontent.com/73323107/174519271-05bd28d7-b115-4186-be1d-aa59888c62be.png">

-------------------------------------------------------------------
## To run
	- First install Unreal Engine 5.3 (download via launcher https://www.unrealengine.com/en-US/download)
	- Plugins:
		- Note: If you don't have a certain plugins and there's a prompt to start anyway, click "yes" -> might change some files to disable the plugin for you.  Do not commit it.
	- Artistic Build (in "Visuals" branch)
		- Kobo_ForestVillage
	- Git clone this project
	- Navigate to the FarmSimulator-RL folder
	- Double click the FarmSimulator-RL Unreal Project File
	- Click 'Play'/'Simulate' (Green arrow/alt+s) to play the simulation
	- Notes for Unreal:
		- You cannot use git to look at the diff's of a file.  Look at the diffs through the revision control functionality in the UE5 editor.
		- You can do global search

## Overall structure
	- Search for each of these in the content manager
	- Farm Simulator-RL folder (most important)
		- Environment Folder (where the environment creation logic lives)
			- BPRoomConfig
				- This is where you can modify the environment values
				- Go to this file and go to defaults on the right panel
					- Modifying this will modify the game
				- Make sure that after you make the changes
			- SimWorld
				- Level blueprint
				- This is where the world is populated
		- Agent folder (where the agent logic lives)
			- BPAgentPathfinding
				- Responsible for pathfinding agent behavior
		- Learning folder (where the RL lives)
			- BPGateInteractor
				- Rewards for RL are populated here
			- RLTrainingManager
				- Manages the nn settings and termination conditions
					- Termination condition can be set to infinite

## Flow
	- When you click 'Play'/'Simulate' (Green arrow/alt+s)
	- 1) SimWorld
		- Spawns rooms according to room config
			- rooms construction script spawns the dimensions + gates collision box
			- room begin play function
				- spawns gates
					- Contains entry queue for agents who want to enter
					- Gates use the agentEnter() function to ask the BPInteractor which makes a decision on entry or rejection, then the admitHuman() function actually admits/rejects the agents
				- spawns agents and fuel within room
					- Make sure you don't have too many agents for each room! Will stop spawning process and display warning.
					- Pathfinding agent has an AI controller attached from the BTTPathfindingAgent in Pathfinding folder
						- Will look inside the room and reserve + navigate to resources and gates
