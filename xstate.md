# State machine

## Action is a side effect. 
There are three locations for actions in state machines:

- transition action.
- entry action. When a state is entered.
- exit action. When a state is exited.

## Statecharts 

They can be transformed to state machines.

statecharts = state-diagrams + depth ( state inside of states ) + ortogonality + broadcast-communications

extended states important concept in statecharts.
It express quantitive data. ( Potentially infinite )
Finite state is qualitative data.

## Guarded transitions

A guard prevents a transition to the next state to happen if a condition is not met.

## Actor model

The actor model is an entity that can send a message to another actor, can create new actors, or can change its behavior as a response to a message. An actor is embodied by using invoke, and can invoke a promise, a callback, an observable, or a machine.

# XState


State machine, define the blue print of the logic:

```
const machine = createMachine({
    initial: "idle",
    states: {
        idle: {
            on: {
                CLICK: "pending"
            }
        },
        "pending: {
            on: {
                COMPLETE: "compelted"
            }
        },
        completed: {
        }
    }
});
```

## Interpreter 

Use the interpreter to create and instance of a machine. The instance is called a service.

```
const service = interpreter(machine);

service.onTransition(state => {
    console.log(state);
});

service.start();

service.send("CLICK);

service.stop();
```

## Multiple actions

```
const someMachine = createMachine({
  initial: 'active',
  states: {
    active: {
      entry: ['enterActive', 'sendTelemetry'],
      on: {
        CLICK: {
          target: 'inactive',
          actions: 'clickActive'
        }
      },
      exit: 'exitActive'
    },
    inactive: {
      entry: 'enterInactive'
    }	
  }
}, {
  actions: {
    sendTelemetry: () => {/* ... */},
    enterActive: () => {
	   console.log('Entered active');
    },
    clickActive: () => {
       console.log('Clicked on active');
    },
    exitActive: () => {
      console.log('Exited active');
    },
    enterInactive: () => {	
      console.log('Entered inactive');
    }	
  }	
});
```

## Extended state / context

Extended state represented in `context` as "contextual state".

```
const lightbulbMachine = createMachine({
  initial: 'turnedOff',
  context: {
    count: 0 // # of times it was turned on
  },
  ...
```

Update the extended state with `assign` as an action ( side effect )

```
import { createMachine, assign } from 'xstate';

const assignAction = assign((context, event) => {
  return {
    message: 'hello',
    count: context.count + event.value
  };
});

console.log(assignAction);
```

## Guard transition

```
const authMachine = createMachine({	
  initial: 'idle',
  context: {
    rejections: 0
  },
  states: {	
    idle: {
      on: {
        FETCH: 'pending'
      }
    },	
    pending: {
      on: {
        REJECT: [
          {	
            target: 'failure',
            cond: (context) => { // <== guard
             return context.rejections >= 5	
            }	
          },	
          {	
            target: 'rejected',	
            actions: assign({
              retries: (context) => context.retries + 1	
            })	
          }	
        ],
        RESOLVE: {
          target: 'resolved'
        }	
      }	
    },
    resolved: {},
    rejected: {
      on: {
        FETCH: 'pending'
      }
    },
    failure: {}
  }
});
```

If having more than one target state, the first one that does pass the condition ( or does not have one ) will be entered.

## Transiet transitions / eventless transitions

They happen on null events, or happening on no events at all. 
Useful with guarded transitions to choose immediately where to go next. For example to have a dynamic initial value.

```
const appMachine = createMachine({
  initial: 'checkingAuth',
  context: {
    user: null,	
  },
  states: {	
    checkingAuth: {	
      on: {	
        '': [
          {
            target: 'dashboard',
            cond: isAuthorized
          },
          { target: 'login' }
        ]
      }
    },
    login: {},
    dashboard: {}
  }
});
```

Empty key as the event is null, and the next state is determined by the condition on the transition.

## Delayed transitions


Transitions happen in zero time. Transitions are never async.
But we can have delayed transitions.

```
...
dragging: {
      after: {
        2000: {
          target: "idle",
          actions: resetPosition

        },
      },
...
```

## Hierarchical states

A state machine within a state.

Example below. The dragging state has another state machine within to add a normal and locked mode. In locked mode you can only scroll horizontally.

```
const dragDropMachine = createMachine({
  initial: 'idle',
  context: {
    x: 0,
    y: 0,
    dx: 0,
    dy: 0,
    px: 0,
    py: 0,
  },
  states: {
    idle: {
      on: {
        mousedown: {
          actions: assignPoint,
          target: 'dragging',
        },
      },
    },
    dragging: {
      initial: "normal",
      states: {
        normal: {
          on: {
            'keydown.shift': {
              target: "locked"
            }
          }
        },
        locked: {
          on: {
            mousemove: {
              actions: assign({
                dx: (context, event) => {
                  return event.clientX - context.px;
                }
              })
            },
         
            "keyup.shift": {
              target: "normal"
            }
          }
          
        }
      },
      on: {
        mousemove: {
          actions: assignDelta,
          internal: false,
        },
        mouseup: {
          actions: [assignPosition],
          target: 'idle',
        },
        'keyup.escape': {
          target: 'idle',
          actions: resetPosition,
        },
      },
    },
  },
});
```

## History states

History state allows going back to the last visited child of a parent state.

```
const displayMachine = createMachine({
  initial: 'hidden',
  states: {
    hidden: {
      on: {
        TURN_ON: 'visible.hist',
      },
    },
    visible: {
      initial: 'light',
      states: {
        light: {
          on: {
            SWITCH: 'dark',
          },
        },
        dark: {
          on: {
            SWITCH: 'light',
          },
        },
        hist: {
          type: 'history',
        },
      },
      on: {
        TURN_OFF: 'hidden',
      },
    },
  },
});
```

## Parallel states

Parallel states can seem paradoxical to state machines because they allow being in two different states at the same time. However, the correct way of thinking about parallel states is to imagine a combination of states.

Compound states are nested states.

Regions are ortogonal states.

```

const displayMachine = createMachine(
  {
    initial: 'hidden',
    states: {
      hidden: {
        on: {
          TURN_ON: 'visible',
        },
      },
      visible: {
        type: 'parallel',
        on: {
          TURN_OFF: 'hidden',
        },
        states: {
          mode: {
            initial: 'light',
            states: {
              light: {
                on: {
                  SWITCH: 'dark',
                },
              },
              dark: {
                on: {
                  SWITCH: 'light',
                },
              },
            },
          },
          brightness: {
            initial: 'bright',
            states: {
              bright: {
                after: {
                  TIMEOUT: 'dim',
                },
              },
              dim: {
                on: {
                  SWITCH: 'bright',
                },
              },
            },
          },
        },
      },
    },
  },
  {
    delays: {
      TIMEOUT: 5000,
    },
  }
);
```
