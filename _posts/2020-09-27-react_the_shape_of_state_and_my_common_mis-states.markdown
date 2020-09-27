---
layout: post
title:      "React: The Shape of State and My Common Mis-states"
date:       2020-09-27 15:08:58 -0400
permalink:  react_the_shape_of_state_and_my_common_mis-states
---


When I started my React project I found it amazing how quickly I had a basic structure of an application setup. A lot of the other projects up until now really required line-by-line attention, and only returning one <div> at a time really made things seem a bit easier when using React and Redux.

However, I struggled the most figuring out the return shape of state and how to handle it, and I hope to leave this here to remind myself how to deal with some of the common issues I had and hopefully shed some light for others. 

**Basic Shape**

When I started the project, my state looked like this:

```
export default function deathReducer(state = { deaths: [ ], loading: false }, action)
       case 'LOADING_DEATHS':
			      return {...state}
			 case 'FETCH_DEATHS':
			      return {...state, deaths: action.payload}
		
        default: 
            return state
    } 
```
As you can see, my funeral app is a bit morbid, but the state shape was what I was familiar with working with from previous labs -- state in this case returns an object, with an array of deaths, and a loading key/value pair. In the application, calling state.deaths would return something shaped like this:

```
  ARRAY
	[ {death1 with other objects inside}, {death2}, {death3}]  OR
	deaths = [{...}, {...}, {...}, {...}]
```

This was easy to deal with when passing it down as props (this.props.deaths), as I could iterate over it using map or find and pull out the attributes I needed to display. For example:
```
<div>
     {this.props.deaths.map(death=> <div>
		      <p>{death.person}</p>
		      <p>{death.date}</p>
		 </div>
		 )}
</div>
```

Adding actions were relatively easy as well, for instance ADD and DELETE:
```
	(in deathReducer.js)
	
	     case 'ADD_DEATH'
                return {...state,
                deaths: [...state.deaths, action.death.data]
                 }
        case 'DELETE_DEATH':
            return {
                ...state, 
                deaths: [...state.deaths.filter(death => death.id !== action.id)]
            }
```

Note to self: Remember that filter returns an array here -- so the delete method is basically saying "return a NEW array with all the deaths that DO NOT match the one returned here from my json (aka action.id). With this method I also needed to send a response back after death.destroy from the backend.

```
(backend Rails API, deathscontroller)

def destroy
     death = Death.find_by_id(params[:id])
		 death.destroy
		 render json: DeathSerializer.new(death)
end		 
```

**Belongs To" State Shape**

My project include a couple of different 'belongs_to' relationships, which caused me to find some different solutions to make sure that state always returned properly. It is important make sure state is returned the same way from different actions so you can pass it down to all the children that are stateless and still render correctly. 

Relationships:
Deaths (has_one)> Ceremony
Ceremony belongs to Death

Dealing with the ceremony belongs_to state shape was an interesting problem to solve.

From the backend, we are first creating a ceremony with strong params. The params I setup already made the association -- which meant that the Ceremony had to send over a death_id or it would throw an error upon creation. I was able to set this up in state, when a ceremony was created on the front end.

```
(Backend -- in ceremony_controller)
def create
     ceremony = Ceremony.new(ceremony_params)
		 death = Death.find_by_id(params[:ceremony][:death_id])
		 
		 if ceremony.save 
		     render json: DeathSerializer.new(death)
		 else
		      render json: {error: 'Ceremony not created'}
			end
end
```
Keep in mind this is inside the CEREMONY controller, which would normally send back a ceremony. However, because we establish the relationship on the backend when creating the ceremony, it has a death_id, we can send back the entire death object to make state easier to deal with when it comes to the front end, as the death will have that ceremony inside of it's shape. 



```
(in deathReducer.js)

case 'CREATE_CEREMONY':
            let deaths = state.deaths.map(death=> {
                if (death.id === action.death.data.id) {
                    return action.death
                } else {
                    return death
                }
            })
       return {...state, deaths: deaths}

```

To break this down: I am mapping the state.deaths, which is an array of objects [{...}, {...}, {...}] and asking EACH invidual object if it's ID matches the ONE death that was sent back with it's new ceremony. If it matches, we are returning that one UPDATED death object, if not, we return the death object that was there before. 

Then the returned array from map is saved to the deaths variable, and state is set to that new array (last line above). 

If we returned the ceremony from the backend, we would have to do a lot more work to find that specific ceremony inside of state and replace it inside of the one death, and then replace that one death inside of state. 

**Splitting One Reducer Into Two**

This was by far the most difficult part. I had a big problem though - I got to a point in my project where it was too difficult to render a super nested element. Inside of a ceremony, the user can add contacts to notify upon their death.

Relationship:
Death has one Ceremony
Ceremony belongs to a Death
Ceremony has many Contacts
Contacts belong to a Ceremony

Here are the steps and explanations behind splitting the reducer. Documentation is also available here: https://redux.js.org/api/combinereducers

```
in index.js

Change all reducers to one:

import rootReducer from './reducers/rootReducer'

Make sure it is connected to the store:

const store = createStore(rootReducer, composeEnhancers(applyMiddleware(thunk)))

```

In my case, I now wanted a deathReducer and a ceremonyReducer.

```
ceremonyReducer.js

export default function ceremonyReducer(state = [], action) {
    switch(action.type){
        case 'CREATE_CEREMONY':
            return [...state, action.payload.data]
	}
```

```
deathReducer.js

export default function deathReducer(state = [], action) {
    switch(action.type){
        case 'FETCH_DEATHS':
            return state = action.payload.data
}
```

Make sure both of those reducers are combined/imported:

```
in reducers/rootReducer.js

import { combineReducers } from 'redux';
import deathReducer from './deathReducer';
import ceremonyReducer from './ceremonyReducer';

const rootReducer = combineReducers({
   deaths: deathReducer, 
   ceremonies: ceremonyReducer})

export default rootReducer;

```

I had to experiment a lot with this to see how state was shaped. However, the best way to see state was to do this:

```
in index.js

const store = createStore(rootReducer...etc)

console.log(store.getState())
```

I was very confused that BOTH reducers used the word state = [ ], it muddied the waters for me on how state was shaped for each of them.  Here is what state looks like now:

```
{
 deaths: [{...}, {...}, {...}, etc],
 ceremonies: [{...}, {...}, {...}, etc]
}

OR from the Chrome Dev Tools:
{deaths: Array(0), ceremonies: Array(0)}
ceremonies: [ ]
deaths: [  ]
__proto__: Object
```

To access these things when I connected to the store now looked like so:

```
in DeathsContainer

const mapStateToProps = state => {
     return {
		      deaths: state.deaths
			}
}

in CeremoniesContainer

const mapStateToProps = state => {
     return {
		      ceremonies: state.ceremonies
			}
}
```

Make sure you know what TYPE and SHAPE you are returning. I got stuck for several hours on the following mistake.

In my reducer, I originally had state returning like so:

```
in ceremonyReducer

case 'FETCH_CEREMONIES':
            return [...state, action.payload.data]
```

Looks pretty normal, right?!

However, what this actually RETURNED, was the weirdest error. It kept telling me I could not MAP over this object, and I kept thinking it was returning an array, so how is that possible? I debugged for hours!

The issue -- I was returning a DOUBLE NESTED ARRAY.  Action.payload.data was actually THIS:

```
action.payload.data

[{...}, {...}, {...}]

```

So by writing the above code of [...state, action.payload.data], my return value was this:

```
this.props.deaths = 
[ [ {...}, {...}, {...} ] ]
```

So in order to iterate over this, I actually needed to call this.props.deaths[0].map(death => death), which I never would've guessed in a million years! Thank you to Emery for helping me debug this issue.

To fix this issue, I returned to my reducer, which is now:

```
case 'FETCH_CEREMONIES':
            return state = action.payload.data
```

Now I am setting state to hold the array itself. It is mutating state, but only on the first fetch / original call. 

**Conclusion**

In conclusion, I still have a lot to learn and a long way to go, but understanding through mistakes and understanding the importance of return values was a valuable lesson in React/Redux state and state of mind in general. I hope this helps!  



