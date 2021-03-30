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
	import BasicApp from "./components/BasicApp.svelte";
	import SubscribeApp from "./components/SubscribeApp.svelte";

	export let url = "";
</script>

<Router {url}>
	<nav>
		<Link to="/">[Basic App]</Link>
		<Link to="/subscribe">[Subscribe App]</Link>
	</nav>
	<hr />

	<div><Route path="/"><BasicApp /></Route></div>
	<div><Route path="/subscribe"><SubscribeApp /></Route></div>
</Router>
