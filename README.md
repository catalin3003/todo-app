# Todo App: Selectors (Integration)
This project integrates a [todo view layer](https://github.com/thinkloop/todo-react-components) and a [todo state container](https://github.com/thinkloop/todo-redux-state) to create a functional todo app. Its primary task is to take flat, normalized, shallow state provided by a redux state container, and transform it into nested, denormalized, hierarchical structures that the view demands. The mechanism by which it does this is called: **selectors**. Selectors are functions that take state (nothing else), and return view-specific structures. For example, the following selector builds a label out of `activePage` state:

```javascript
import todoReduxState from 'todo-redux-state';

export default function () {

	// get relevant state
	const { activePage } = todoReduxState.state;
	
	// generate and return label
	return `${activePage} is active`;
}
```

Whenever that label is needed, the selector is called and the value is returned. If underlying state changes, the selector will return an updated value. Given the same state, the selector will always return the same value.

Sometimes selectors become popular and get called many times between state changes, by ui elements, or other selectors. Each time they are called, the computations are re-executed, even if the state has not changed and the returned values are the same. This does not matter much with our trivial example, but it can matter a lot with a heavier use-case, like say, building, filtering, and sorting the app's primary array: 

```javascript
/*
* BAD - this selector is expensive and re-computes every time it is called
*/

import todoReduxState from 'todo-redux-state';

export default function () {
	const { todos, searchPhrase } = todoReduxState.state;
	
	return Object.keys(todos)
				
		// convert objects to array
		.map(key => {
			return { ...todos[key], id: key };
		})
		
		// filter out those that do noot match search phrase
		.filter(todo => todo.description.indexOf(searchPhrase))
		
		// sort
		.sort((a, b) => a.createdDate < b.createdDate ? -1 : 1);
}
```

The solution is to use [memoization](https://github.com/thinkloop/memoizerific): returning cached values if functions are called with the same params. The previous example can be rewritten to use memoization like this:

```javascript
/*
* GOOD - computations only happen when state changes
*/
import memoizerific from 'memoizerific';
import todoReduxState from 'todo-redux-state';

// default entry point, gathers state and runs memoized function
export default function () {
	
	// get relevant state
	const { todos, searchPhrase } = todoReduxState.state;
	
	// run memoized function
	return selectTodos(todos, searchPhrase);
}

// memoized function
export const selectTodos = memoizerific(1)((todos, searchPhrase) => {
	
	// run expensive computation
	return Object.keys(todos)
      .map(key => {
      	return {
      		...todos[key],
      		id: key
      	};
      })
      .filter(todo => todo.description.indexOf(searchPhrase))
      .sort((a, b) => a.createdDate < b.createdDate ? -1 : 1);  
});
```

In this optimized example, the expensive computations are moved to a separate memoized function that only runs when `todos` or `searchPhrase` have changed, otherwise it returns a cached result. Additionally, since cached results can be compared using strict equals, they can also be used to minimize unnecessary re-renders in the react ui.

### Run

To run it, clone the repo and start the webserver:

```
> git clone https://github.com/thinkloop/todo-app
> npm start

// open localhost url
```

### License

Released under an MIT license.

### Related
1. [todo-react-components](https://github.com/thinkloop/todo-react-components) (view-layer)
2. [todo-redux-state](https://github.com/thinkloop/todo-redux-state) (data-layer)
3. [todo-app](https://github.com/thinkloop/todo-app) (integration)

### Like it? Star It
