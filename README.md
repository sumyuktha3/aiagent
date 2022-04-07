# Developing AI Agent with PEAS Description

## AIM

To find the PEAS description for the given AI problem and develop an AI agent.

## THEORY
Considering Vacuum cleaner as agent and locations of 9 different rooms as environment we are going to develop an agent based on its cleanliness.
If the room is clean it shifts to the next adjacent room, if it is dirty it clean the room and continue its activity.

## PEAS DESCRIPTION

![description](https://user-images.githubusercontent.com/75235818/162262176-79eb978b-c4a3-407f-9d61-8c319b4026d7.jpeg

## DESIGN STEPS
### STEP 1:
Identifying the input: location of agent, status of location.

### STEP 2:
Identifying the output: Action(NE,West1,West2,South1,South2,East1,East2,North,Initial)

### STEP 3:

Agent: Vacuum cleaner
Performance Measure: 
Moving locations and cleanliness
Environment:
Locations
T,A,B,C,D,P,Q,R,S
Actuators:
Brushes and Vacuum Extractor
Sensors:
Camera, Dirt detection sensor

### STEP 4:

The agent should detect the dirt and clean-if the location it is dirty,else it should move to the adjacent location.

### STEP 5:

The performance is measured with the dirt detection and cleaning.

## PROGRAM

import random
import time
class Thing:
    """
        This represents any physical object that can appear in an Environment.
    """

    def is_alive(self):
        """Things that are 'alive' should return true."""
        return hasattr(self, 'alive') and self.alive

    def show_state(self):
        """Display the agent's internal state. Subclasses should override."""
        print("I don't know how to show_state.")


class Agent(Thing):
    """
        An Agent is a subclass of Thing
    """

    def __init__(self, program=None):
        self.alive = True
        self.performance = 0
        self.program = program

    def can_grab(self, thing):
        """Return True if this agent can grab this thing.
        Override for appropriate subclasses of Agent and Thing."""
        return False

def TableDrivenAgentProgram(table):
    """
    This agent selects an action based on the percept sequence.
    It is practical only for tiny domains.
    To customize it, provide as table a dictionary of all
    {percept_sequence:action} pairs.
    """
    percepts = []

    def program(percept):
        percepts.append(percept)
        action=table.get(tuple(percept))
        return action

    return program

le_T, le_A, le_B, le_C, le_D, le_P, le_Q, le_R, le_S = (1,1), (2,2), (1,2), (0,2), (0,1), (0,0), (1,0), (2,0), (2,1) 

def TableDrivenVacuumAgent():
    """
    Tabular approach towards vacuum world
    """
    table = {(le_T, 'Clean'): 'NE',
             (le_T, 'Dirty'): 'Suck',
             (le_A, 'Clean'): 'West1',
             (le_A, 'Dirty'): 'Suck',
             (le_B, 'Clean'): 'West2',
             (le_B, 'Dirty'): 'Suck',
             (le_C, 'Clean'): 'South1',
             (le_C, 'Dirty'): 'Suck',
             (le_D, 'Clean'): 'South2',
             (le_D, 'Dirty'): 'Suck',
             (le_P, 'Clean'): 'East1',
             (le_P, 'Dirty'): 'Suck',
             (le_Q, 'Clean'): 'East2',
             (le_Q, 'Dirty'): 'Suck',
             (le_R, 'Clean'): 'North',
             (le_R, 'Dirty'): 'Suck',
             (le_S, 'Clean'): 'Initial',
             (le_S, 'Dirty'): 'Suck',
    }
    return Agent(TableDrivenAgentProgram(table))


class Environment:
    """Abstract class representing an Environment. 'Real' Environment classes
    inherit from this. Your Environment will typically need to implement:
        percept:           Define the percept that an agent sees.
        execute_action:    Define the effects of executing an action.
                           Also update the agent.performance slot.
    The environment keeps a list of .things and .agents (which is a subset
    of .things). Each agent has a .performance slot, initialized to 0.
    Each thing has a .location slot, even though some environments may not
    need this."""

    def __init__(self):
        self.things = []
        self.agents = []

    def percept(self, agent):
        """Return the percept that the agent sees at this point. (Implement this.)"""
        raise NotImplementedError

    def execute_action(self, agent, action):
        """Change the world to reflect this action. (Implement this.)"""
        raise NotImplementedError

    def default_location(self, thing):
        """Default location to place a new thing with unspecified location."""
        return None

    def is_done(self):
        """By default, we're done when we can't find a live agent."""
        return not any(agent.is_alive() for agent in self.agents)

    def step(self):
        """Run the environment for one time step. If the
        actions and exogenous changes are independent, this method will
        do. If there are interactions between them, you'll need to
        override this method."""
        if not self.is_done():
            actions = []
            for agent in self.agents:
                if agent.alive:
                    actions.append(agent.program(self.percept(agent)))
                else:
                    actions.append("")
            for (agent, action) in zip(self.agents, actions):
                self.execute_action(agent, action)

    def run(self, steps=1000):
        """Run the Environment for given number of time steps."""
        for step in range(steps):
            if self.is_done():
                return
            self.step()

    def add_thing(self, thing, location=None):
        """Add a thing to the environment, setting its location. For
        convenience, if thing is an agent program we make a new agent
        for it. (Shouldn't need to override this.)"""
        if not isinstance(thing, Thing):
            thing = Agent(thing)
        if thing in self.things:
            print("Can't add the same thing twice")
        else:
            thing.location = location if location is not None else self.default_location(thing)
            self.things.append(thing)
            if isinstance(thing, Agent):
                thing.performance = 0
                self.agents.append(thing)

    def delete_thing(self, thing):
        """Remove a thing from the environment."""
        try:
            self.things.remove(thing)
        except ValueError as e:
            print(e)
            print("  in Environment delete_thing")
            print("  Thing to be removed: {} at {}".format(thing, thing.location))
            print("  from list: {}".format([(thing, thing.location) for thing in self.things]))
        if thing in self.agents:
            self.agents.remove(thing)


class TrivialVacuumEnvironment(Environment):
    """This environment has two locations, A and B. Each can be Dirty
    or Clean. The agent perceives its location and the location's
    status. This serves as an example of how to implement a simple
    Environment."""

    def __init__(self):
        super().__init__()
        self.status = {le_T: random.choice(['Clean', 'Dirty']),
                       le_A: random.choice(['Clean', 'Dirty']),
                       le_B: random.choice(['Clean', 'Dirty']),
                       le_C: random.choice(['Clean', 'Dirty']),
                       le_D: random.choice(['Clean', 'Dirty']),
                       le_P: random.choice(['Clean', 'Dirty']),
                       le_Q: random.choice(['Clean', 'Dirty']),
                       le_R: random.choice(['Clean', 'Dirty']),
                       le_S: random.choice(['Clean', 'Dirty']),}

    def thing_classes(self):
        return [ TableDrivenVacuumAgent]

    def percept(self, agent):
        """Returns the agent's location, and the location status (Dirty/Clean)."""
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        """Change agent's location and/or location's status; track performance.
        Score 10 for each dirt cleaned; -1 for each move."""
        if action=='NE':
            agent.location = le_A
            agent.performance -=1
        elif action=='West1':
            agent.location = le_B
            agent.performance -=1
        elif action=='West2':
            agent.location = le_C
            agent.performance -=1
        elif action=='South1':
            agent.location = le_D
            agent.performance -=1
        elif action=='South2':
            agent.location = le_P
            agent.performance -=1
        elif action=='East1':
            agent.location = le_Q
            agent.performance -=1
        elif action=='East2':
            agent.location = le_R
            agent.performance -=1
        elif action=='North':
            agent.location = le_S
            agent.performance -=1
        elif action=='Initial':
            agent.location = le_T
            agent.performance -=1
        elif action=='Suck':
            if self.status[agent.location]=='Dirty':
                agent.performance+=10
            self.status[agent.location]='Clean'

    def default_location(self, thing):
        """Agents start in either location at random."""
        return random.choice([le_T, le_A, le_B, le_C, le_D, le_P, le_Q, le_R, le_S])


if __name__ == "__main__":
    agent = TableDrivenVacuumAgent()
    environment = TrivialVacuumEnvironment()
    environment.add_thing(agent)
    print('Starting Action\n',environment.status)
    print('Agent Location\n',agent.location)
    print('Agent Performance\n',agent.performance)
    time.sleep(3)
    for i in range(3):
        print(environment.run(steps=1))
        print('Ending Action\n',environment.status)
        print('Agent Location\n',agent.location)
        print('Agent Performance\n',agent.performance)
        time.sleep(3)



## OUTPUT

![output5](https://user-images.githubusercontent.com/75235818/162262461-2213d5c3-2810-42e3-9279-e8936047cc55.jpg)

## RESULT

Thus, a vacuum cleaner is developed as an agent and PEAS description is mentioned.
