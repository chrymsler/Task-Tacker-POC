<script>
    import { gql } from "@apollo/client";
    import { subscribe } from "svelte-apollo";
    import { dataset_dev } from "svelte/internal";
    import App from "../App.svelte";
    import Hoverable from "./Hoverable.svelte";

    const sProjectsQuery = gql`
        subscription AllSDataQuery {
            querySProject {
                id
                name
                stasks {
                    id
                    name
                    status
                    ssubTasks {
                        id
                        name
                        status
                    }
                }
            }
        }
    `;

    const projects = subscribe(sProjectsQuery, { fetchPolicy: "network-only" });
</script>

<main>Subscribe App</main>
{#if $projects.loading}
    Loading...
{:else if $projects.error}
    Oh no! {$projects.error.message}
{:else if $projects.data}
    <ol>
        {#each $projects.data.querySProject as project}
            <li>{project.name}</li>
        {/each}
    </ol>
{/if}
