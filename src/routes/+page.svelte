<!-- Main Svelte file for our app -->
<script>
  // Import relevant functions for communicating with Firebase database
  import { initializeApp, getApps, getApp } from "firebase/app";
  // Import relevant functions for adding/updating/deleting tasks with Firebase database
  import {
    getFirestore,
    collection,
    onSnapshot,
    doc,
    addDoc,
    updateDoc,
    deleteDoc,
  } from "firebase/firestore";
  import { firebaseConfig } from "$lib/firebaseConfig";
  import { browser } from "$app/environment";

  // Initialize connection to Firebase database
  browser && (getApps().length == 0 ? initializeApp(firebaseConfig) : getApp());
  // db = database
  const db = browser && getFirestore();
  // Get the specific collection within the Firebase database of "todos" (which is the collection we use)
  const colRef = browser && collection(db, "todos");

  // List of todos that we get from Firebase database, which each are objects
  // with fields completed (boolean), createdAt (timestamp), and task ("string")
  let todos = [];
  let task = "";

  // Get all stored tasks from Firebase database
  browser &&
    onSnapshot(colRef, (querySnapshot) => {
      let fbTodos = [];
      querySnapshot.forEach((doc) => {
        // Convert between Firebase style format and our code format for a task
        let todo = { ...doc._document.data.value.mapValue.fields, id: doc.id };
        // Buffer to temporarily store the todo tasks fetched
        fbTodos = [todo, ...fbTodos];
      });
      // Store the todo tasks fetched in the todos variable above
      todos = fbTodos;
    });

  // Helper function to add a todo task to Firebase database
  const addTodo = async () => {
    // If the task is nonempty, add it; otherwise, do nothing!
    if (task != "") {
      // Add a "document" to Firebase database, in the "todos" collection
      await addDoc(collection(db, "todos"), {
        // The task has the user entered title name
        task: task,
        // By default, tasks aren't completed
        completed: false,
        // The task has just been created!
        createdAt: new Date(),
      });
    }
    // Clear the user input box for inputting the task name
    task = "";
  };

  // Helper function to check off a task as done
  const markTodoDone = async (id) => {
    // Update the "document" in Firebase database in the "todos" collection
    await updateDoc(doc(db, "todos", id), {
      // Set task to be completed
      completed: true,
    });
  };

  // Helper function to delete a todo task
  const deleteTodo = async (id) => {
    // Delete from the "todos" collection in the Firebase database
    await deleteDoc(doc(db, "todos", id));
  };
</script>

<!-- Div that contains the task name input box and button -->
<div class="inputs">
  <!-- Input box -->
  <input type="text" placeholder="Add a task" bind:value={task} />
  <!-- Input button labelled "Add", which when clicked calls the helper function to add a todo task -->
  <button class="addButton" on:click={addTodo}> Add </button>
</div>

<!-- Div that contains the main todo task list -->
<div class="container">
  <!-- For each todo task in the todos list above -->
  {#each todos as todo}
    <!-- If the task is completed, the HTML div has class complete; otherwise, it has class incomplete -->
    <div
      class="task"
      class:incomplete={!todo.completed.booleanValue}
      class:complete={todo.completed.booleanValue}
    >
      <!-- Display the task name that was stored in the database -->
      <span class="todoVal">
        {todo.task.stringValue}
      </span>
      <!-- Display the "mark as done" and "delete" buttons -->
      <span>
        <button on:click={() => markTodoDone(todo.id)}>✓</button>
        <button on:click={() => deleteTodo(todo.id)}>✗</button>
      </span>
    </div>
  {/each}
</div>

<!-- CSS manual styling for the webpage -->
<style>
  /* Style the input text box */
  input {
    width: 50%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
    font-size: 30px;
  }

  /* Style tasks that are completed */
  .complete {
    background-color:lightgreen;
  }
  /* Style tasks that are incomplete */
  .incomplete {
    background-color:lightpink;
  }
  /* Style the total list of todos, which is contained in a container */
  .container {
    position: relative;
    width: 50%;
    height: 100%;
    display: flex;
    text-align: center;
    flex-direction: column;
  }

  /* Style each individual task */
  .task {
    padding-left: 100px;
    padding-right: 100px;
    display: flex;
    justify-content: space-between;
    margin: 10px;
  }

  /* Style the text label for each individual task */
  .todoVal {
    padding-top: 15px;
  }

  /* Style the "Add" button in the input form */
  .addButton {
    border: 5px solid lightskyblue;
    margin: 5px;
  }

  /* Style the "mark as done" and "delete" buttons for each task */
  button {
    border: none;
    color: black;
    padding: 15px 32px;
    text-align: center;
    background-color: transparent;
    text-decoration: none;
    display: inline-block;
    font-size: 30px;
  }

  span {
    font-size: 30px;
  }
</style>
