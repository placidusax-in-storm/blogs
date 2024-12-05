I've also noticed that there are too many Vuex states (as seen in the `mapState` usage). I prefer to store fetched data within the component's scope (i.e., `this.data`) rather than in the global Vuex store.

Storing fetched data in Vuex introduces at least four problems:

1. Without a `queryKey`, different components may request the same API with different parameters, which can cause a component to receive dirty data.
    
2. In JavaScript, Vuex breaks the reference safety rule, as all data and actions are bound in a dynamic way, making it difficult to track which API is called behind a specific action.
    
3. The global context has a tendency to become mixed. If those states are inter-referenced, they will ultimately form a reference graph between the states, meaning they are all coupled.
    
4. Tracking the initialization and mutation of global states can be challenging. While Vuex makes this easier, we don't need to solve those problems if we define those states as narrowly as possible within the component's scope.
    
