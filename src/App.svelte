<script>
	import {
		ApolloClient,
		InMemoryCache,
		HttpLink,
		split,
	} from "@apollo/client";
	import { getMainDefinition } from "@apollo/client/utilities";
	import { WebSocketLink } from "@apollo/link-ws";
	import { setClient } from "svelte-apollo";

	const httpLink = new HttpLink({
		uri: "https://billowing-bird.ap-south-1.aws.cloud.dgraph.io/graphql",
	});

	const wsLink = new WebSocketLink({
		uri: "wss://billowing-bird.ap-south-1.aws.cloud.dgraph.io/graphql",
		options: {
			reconnect: true,
		},
	});

	const splitLink = split(
		({ query }) => {
			const definition = getMainDefinition(query);
			return (
				definition.kind === "OperationDefinition" &&
				definition.operation === "subscription"
			);
		},
		wsLink,
		httpLink
	);

	const client = new ApolloClient({
		link: splitLink,
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
