Try:

Implement useQuery in class component.

```jsx
class Demo extends Component {
	state = {}
	componentDidUpdate(prevProps,currentProps){
		if(prevProps.userId !== currentProps.userId){
			// and what suppose to happen is
			// this.state.user.{isLoading, data, error} to push and update
			// BUT, we didn't declare that state
			this.fetchUser(currentProps.userId)
		}
	}
	
}
```

All is about Declarative not Imperative