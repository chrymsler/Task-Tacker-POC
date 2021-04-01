<script>
    import { gql } from "@apollo/client";
    import { mutation, subscribe } from "svelte-apollo";
    import Hoverable from "./Hoverable.svelte";

    var count = 1;

    const subscribeProjectsQuery = gql`
        subscription AllDataSubscribeQuery {
            queryProject {
                id
                name
                taskCount
                tasks {
                    id
                    name
                    status
                    subTaskCount
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

    const addProjectQuery = gql`
        mutation MyMutation($name: String!) {
            addProject(input: { name: $name, taskCount: 0 }) {
                numUids
            }
        }
    `;

    const addTaskQuery = gql`
        mutation MyMutation(
            $id: ID = ""
            $name: String = ""
            $status: String = ""
            $projectid: [ID!] = ""
            $taskCount: Int = 0
        ) {
            addTask(
                input: {
                    project: { id: $id }
                    name: $name
                    status: $status
                    subTaskCount: 0
                }
            ) {
                numUids
            }
            updateProject(
                input: {
                    filter: { id: $projectid }
                    set: { taskCount: $taskCount }
                }
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
            $subTaskCount: Int = 0
        ) {
            addSubTask(
                input: { task: { id: $id }, name: $name, status: $status }
            ) {
                numUids
            }
            updateTask(
                input: {
                    filter: { id: $taskId }
                    set: { status: $taskStatus, subTaskCount: $subTaskCount }
                }
            ) {
                numUids
            }
        }
    `;

    const deleteProjectQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $taskIds: [ID!] = ""
            $subTaskIds: [ID!] = ""
        ) {
            deleteProject(filter: { id: $id }) {
                msg
                numUids
            }
            deleteTask(filter: { id: $taskIds }) {
                msg
                numUids
            }
            deleteSubTask(filter: { id: $subTaskIds }) {
                msg
                numUids
            }
        }
    `;

    const deleteTaskQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $subTaskIds: [ID!] = ""
            $projectid: [ID!] = ""
            $taskCount: Int = 0
        ) {
            deleteTask(filter: { id: $id }) {
                msg
                numUids
            }
            deleteSubTask(filter: { id: $subTaskIds }) {
                msg
                numUids
            }
            updateProject(
                input: {
                    filter: { id: $projectid }
                    set: { taskCount: $taskCount }
                }
            ) {
                numUids
            }
        }
    `;

    const deleteSubTaskQuery = gql`
        mutation MyMutation(
            $id: [ID!] = ""
            $taskId: [ID!] = ""
            $taskStatus: String = ""
            $subTaskCount: Int = 0
        ) {
            deleteSubTask(filter: { id: $id }) {
                numUids
                msg
            }
            updateTask(
                input: {
                    filter: { id: $taskId }
                    set: { status: $taskStatus, subTaskCount: $subTaskCount }
                }
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

    const projects = subscribe(subscribeProjectsQuery, {
        fetchPolicy: "cache-and-network",
    });

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
        mutateAddProject({
            variables: { name },
        });
    }

    function addTask(id, name, status) {
        const projectid = id;
        var taskCount = getTaskCount(projectid) + 1;
        mutateAddTask({
            variables: { id, name, status, projectid, taskCount },
        });
    }

    function getTaskCount(projectid) {
        var taskCount;

        $projects.data.queryProject.forEach((project) => {
            if (project.id == projectid) {
                taskCount = project.taskCount;
                return;
            }
        });

        return taskCount;
    }

    function addSubTask(id, name, status) {
        const taskId = id;
        const taskStatus = "P";
        var subTaskCount = getSubTaskCount(taskId) + 1;

        mutateAddSubTask({
            variables: { id, name, status, taskId, taskStatus, subTaskCount },
        });
    }

    function getSubTaskCount(taskid) {
        var subTaskCount = -1;

        $projects.data.queryProject.forEach((project) => {
            project.tasks.forEach((task) => {
                if (task.id == taskid) {
                    subTaskCount = task.subTaskCount;
                    return;
                }
            });

            if (subTaskCount != -1) {
                return;
            }
        });

        return subTaskCount;
    }

    function deleteProject(id) {
        var taskIds = getTaskIdsOfProject(id);
        var subTaskIds = getSubTaskIDsOfProject(id);
        mutateDeleteProject({
            variables: { id, taskIds, subTaskIds },
        });
    }

    function getTaskIdsOfProject(projectID) {
        var taskIds = [];

        $projects.data.queryProject.forEach((project) => {
            if (project.id == projectID) {
                project.tasks.forEach((task) => {
                    taskIds.push(task.id);
                });
            }
        });

        return taskIds;
    }

    function getSubTaskIDsOfProject(projectID) {
        var subTaskIds = [];

        $projects.data.queryProject.forEach((project) => {
            if (project.id == projectID) {
                project.tasks.forEach((task) => {
                    task.subTasks.forEach((subtask) => {
                        subTaskIds.push(subtask.id);
                    });
                });
            }
        });

        return subTaskIds;
    }

    function deleteTask(id, projectid) {
        var subTaskIds = getSubTaskIdsOfTask(id);
        var taskCount = getTaskCount(projectid) - 1;

        mutateDeleteTask({
            variables: { id, subTaskIds, projectid, taskCount },
        });
    }

    function getSubTaskIdsOfTask(taskId) {
        var subTaskIds = [];

        $projects.data.queryProject.forEach((project) => {
            project.tasks.forEach((task) => {
                if (task.id == taskId) {
                    task.subTasks.forEach((subtask) => {
                        subTaskIds.push(subtask.id);
                    });
                }
            });
        });

        return subTaskIds;
    }

    function deleteSubTask(id, taskId) {
        const taskStatus = calcTaskStatus(taskId, id);
        var subTaskCount = getSubTaskCount(taskId) - 1;

        mutateDeleteSubTask({
            variables: { id, taskId, taskStatus, subTaskCount },
        });
    }

    function updateProject(id, name) {
        mutateUpdateProject({
            variables: { id, name },
        });
    }

    function updateTask(id, name, status) {
        mutateUpdateTask({
            variables: { id, name, status },
        });
    }

    function updateSubTask(id, name, status) {
        mutateUpdateSubTask({
            variables: { id, name, status },
        });
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
        });
    }

    function calcTaskStatus(taskId, ignoreSubTaskId) {
        var taskStatus;

        var complete = true;

        $projects.data.queryProject.forEach((project) => {
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

<main>
    Subscribe App
    <a on:click={() => addProject(getLabel("Project"))}>[Add Project]</a>
</main>
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
                        {project.name} ({project.taskCount} tasks)
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
                                {task.name} ({task.subTaskCount} subtasks) [{decodeStatus(
                                    task.status
                                )}]
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
                                    <a
                                        on:click={() =>
                                            deleteTask(task.id, project.id)}
                                    >
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
