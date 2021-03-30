<script>
    import { gql } from "@apollo/client";
    import { query, mutation } from "svelte-apollo";
    import Hoverable from "./Hoverable.svelte";

    var count = 1;

    const projectsQuery = gql`
        query MyQuery {
            queryProject {
                id
                name
                tasks {
                    id
                    name
                    status
                    createdOn
                    subTasks {
                        id
                        name
                        status
                        createdOn
                    }
                }
            }
        }
    `;

    const deleteProjectQuery = gql`
        mutation MyMutation($id: [ID!]) {
            deleteProject(filter: { id: $id }) {
                msg
                numUids
            }
        }
    `;

    const addProjectQuery = gql`
        mutation MyMutation($name: String!) {
            addProject(input: { name: $name }) {
                numUids
            }
        }
    `;

    const addTaskQuery = gql`
        mutation MyMutation(
            $id: ID = ""
            $name: String = ""
            $status: String = ""
        ) {
            addTask(
                input: { project: { id: $id }, name: $name, status: $status }
            ) {
                numUids
            }
        }
    `;

    const addSubTaskQuery = gql`
        mutation MyMutation(
            $id: ID = ""
            $name: String = ""
            $status: String = ""
            $taskId: [ID!] = ""
            $taskStatus: String = ""
        ) {
            addSubTask(
                input: { task: { id: $id }, name: $name, status: $status }
            ) {
                numUids
            }
            updateTask(
                input: { filter: { id: $taskId }, set: { status: $taskStatus } }
            ) {
                numUids
            }
        }
    `;

    const deleteTaskQuery = gql`
        mutation MyMutation($id: [ID!] = "") {
            deleteTask(filter: { id: $id }) {
                numUids
                msg
            }
        }
    `;

    const deleteSubTaskQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $taskId: [ID!] = ""
            $taskStatus: String = ""
        ) {
            deleteSubTask(filter: { id: $id }) {
                numUids
                msg
            }
            updateTask(
                input: { filter: { id: $taskId }, set: { status: $taskStatus } }
            ) {
                numUids
            }
        }
    `;

    const updateProjectQuery = gql`
        mutation MyMutation($id: [ID!] = "", $name: String = "") {
            updateProject(
                input: { filter: { id: $id }, set: { name: $name } }
            ) {
                numUids
            }
        }
    `;

    const updateTaskQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $name: String = ""
            $status: String = ""
        ) {
            updateTask(
                input: {
                    filter: { id: $id }
                    set: { name: $name, status: $status }
                }
            ) {
                numUids
            }
        }
    `;

    const updateSubTaskQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $name: String = ""
            $status: String = ""
        ) {
            updateSubTask(
                input: {
                    filter: { id: $id }
                    set: { name: $name, status: $status }
                }
            ) {
                numUids
            }
        }
    `;

    const updateSubTaskStatusQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $status: String = ""
            $taskId: [ID!] = ""
            $taskStatus: String = ""
        ) {
            updateSubTask(
                input: { filter: { id: $id }, set: { status: $status } }
            ) {
                numUids
            }
            updateTask(
                input: { filter: { id: $taskId }, set: { status: $taskStatus } }
            ) {
                numUids
            }
        }
    `;

    const projects = query(projectsQuery, { fetchPolicy: "cache-and-network" });

    const mutateAddProject = mutation(addProjectQuery);

    const mutateAddTask = mutation(addTaskQuery);

    const mutateAddSubTask = mutation(addSubTaskQuery);

    const mutateDeleteProject = mutation(deleteProjectQuery);

    const mutateDeleteTask = mutation(deleteTaskQuery);

    const mutateDeleteSubTask = mutation(deleteSubTaskQuery);

    const mutateUpdateProject = mutation(updateProjectQuery);

    const mutateUpdateTask = mutation(updateTaskQuery);

    const mutateUpdateSubTask = mutation(updateSubTaskQuery);

    const mutateUpdateSubTaskStatus = mutation(updateSubTaskStatusQuery);

    function addProject(name) {
        mutateAddProject({ variables: { name } }).then(() => refresh());
    }

    function addTask(id, name, status) {
        mutateAddTask({ variables: { id, name, status } }).then(() =>
            refresh()
        );
    }

    function addSubTask(id, name, status) {
        const taskId = id;
        const taskStatus = "P";
        mutateAddSubTask({
            variables: { id, name, status, taskId, taskStatus },
        }).then(() => refresh());
    }

    function deleteProject(id) {
        mutateDeleteProject({ variables: { id } }).then(() => refresh());
    }

    function deleteTask(id) {
        mutateDeleteTask({ variables: { id } }).then(() => refresh());
    }

    function deleteSubTask(id, taskId) {
        const taskStatus = calcTaskStatus(taskId, id);

        mutateDeleteSubTask({
            variables: { id, taskId, taskStatus },
        }).then(() => refresh());
    }

    function updateProject(id, name) {
        mutateUpdateProject({ variables: { id, name } }).then(() => refresh());
    }

    function updateTask(id, name, status) {
        mutateUpdateTask({ variables: { id, name, status } }).then(() =>
            refresh()
        );
    }

    function updateSubTask(id, name, status) {
        mutateUpdateSubTask({ variables: { id, name, status } }).then(() =>
            refresh()
        );
    }

    function updateSubTaskStatus(id, status, taskId) {
        var taskStatus;

        if (status == "P") {
            taskStatus = "P";
        } else {
            taskStatus = calcTaskStatus(taskId, id);
        }

        mutateUpdateSubTaskStatus({
            variables: { id, status, taskId, taskStatus },
        }).then(() => refresh());
    }

    function calcTaskStatus(taskId, ignoreSubTaskId) {
        var taskStatus;

        var complete = true;

        projects.getCurrentResult().data.queryProject.forEach((project) => {
            project.tasks.forEach((task) => {
                if (task.id == taskId) {
                    task.subTasks.forEach((subtask) => {
                        if (subtask.id != ignoreSubTaskId) {
                            if (subtask.status == "P") {
                                complete = false;

                                return;
                            }
                        }
                    });

                    if (!complete) {
                        return;
                    }
                }

                if (!complete) {
                    return;
                }
            });
        });

        if (complete) {
            taskStatus = "C";
        } else {
            taskStatus = "P";
        }

        return taskStatus;
    }

    function getLabel(title) {
        count++;
        return title + " " + count;
    }

    function refresh() {
        if (!projects.loading) {
            projects.refetch();
        }
    }

    function decodeStatus(status) {
        var retVal;

        if (status == "P") {
            retVal = "Pending";
        } else if (status == "C") {
            retVal = "Completed";
        }

        return retVal;
    }
</script>

<main>Projects</main>
<a on:click={() => addProject(getLabel("Project"))}>[Add Project]</a>
<a on:click={() => projects.refetch()}>[Refresh]</a>
<div>
    {#if $projects.loading}
        Loading...
    {:else if $projects.error}
        Oh no! {$projects.error.message}
    {:else}
        <ol>
            {#each $projects.data.queryProject as project}
                <li>
                    <Hoverable let:hovering>
                        {project.name}
                        {#if hovering}
                            --&gt;
                            <a
                                on:click={() =>
                                    addTask(project.id, getLabel("Task"), "P")}
                            >
                                [Add Task]
                            </a>
                            <a on:click={() => deleteProject(project.id)}>
                                [Delete Project]
                            </a>
                            <a
                                on:click={() =>
                                    updateProject(
                                        project.id,
                                        getLabel("Project")
                                    )}
                            >
                                [Update Project]
                            </a>
                        {/if}
                    </Hoverable>
                </li>
                <ol>
                    {#each project.tasks as task}
                        <li>
                            <Hoverable let:hovering>
                                {task.name} [{decodeStatus(task.status)}]
                                {#if hovering}
                                    --&gt;
                                    <a
                                        on:click={() =>
                                            addSubTask(
                                                task.id,
                                                getLabel("SubTask"),
                                                "P"
                                            )}
                                    >
                                        [Add SubTask]
                                    </a>
                                    <a on:click={() => deleteTask(task.id)}>
                                        [Delete Task]
                                    </a>
                                    <a
                                        on:click={() =>
                                            updateTask(
                                                task.id,
                                                getLabel("Task"),
                                                task.status
                                            )}
                                    >
                                        [Update Task]
                                    </a>
                                {/if}
                            </Hoverable>
                        </li>
                        <ol>
                            {#each task.subTasks as subtask}
                                <li>
                                    <Hoverable let:hovering>
                                        {subtask.name} [{decodeStatus(
                                            subtask.status
                                        )}]
                                        {#if hovering}
                                            --&gt;
                                            {#if subtask.status == "P"}
                                                <a
                                                    on:click={() =>
                                                        updateSubTaskStatus(
                                                            subtask.id,
                                                            "C",
                                                            task.id
                                                        )}
                                                >
                                                    [Done]
                                                </a>
                                            {:else if subtask.status == "C"}
                                                <a
                                                    on:click={() =>
                                                        updateSubTaskStatus(
                                                            subtask.id,
                                                            "P",
                                                            task.id
                                                        )}
                                                >
                                                    [Not Done]
                                                </a>
                                            {/if}
                                            <a
                                                on:click={() =>
                                                    deleteSubTask(
                                                        subtask.id,
                                                        task.id
                                                    )}
                                            >
                                                [Delete SubTask]
                                            </a>
                                            <a
                                                on:click={() =>
                                                    updateSubTask(
                                                        subtask.id,
                                                        getLabel("SubTask"),
                                                        subtask.status
                                                    )}
                                            >
                                                [Update SubTask]
                                            </a>
                                        {/if}
                                    </Hoverable>
                                </li>
                            {/each}
                        </ol>
                    {/each}
                </ol>
            {/each}
        </ol>
    {/if}
</div>

<style>
    li {
        padding-bottom: 10px;
    }

    a:hover {
        cursor: pointer;
    }
</style>
