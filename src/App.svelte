<script>
	import { ApolloClient, InMemoryCache } from "@apollo/client";
	import { setClient } from "svelte-apollo";

	const client = new ApolloClient({
		uri: "https://billowing-bird.ap-south-1.aws.cloud.dgraph.io/graphql",
		cache: new InMemoryCache(),
		onError: ({ networkError, graphQLErrors }) => {
			console.log("graphQLErrors", graphQLErrors);
			console.log("networkError", networkError);
		},
		defaultOptions: {
			query: {
				errorPolicy: "all",
			},
		},
		fetchOptions: {
			mode: "no-cors",
		},
	});

	setClient(client);

	import { Router, Route, Link } from "svelte-routing";
	import Home from "./components/Home.svelte";

	export let url = "";
</script>

<Router {url}>
	<nav><Link to="/">Home</Link></nav>

	<div><Route path="/"><Home /></Route></div>
</Router>
