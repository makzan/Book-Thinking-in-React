React tasks and local storage demo

https://codepen.io/makzan/professor/39ddcd38cf563e44e503e2ae0aeda985

```js
if (localStorage["tasks"] != null) {
  var loadedTasks = JSON.parse(localStorage["tasks"]);
}

function Demo(props){
  let {existingTasks=[]} = props;
  
  const [tasks, setTasks] = React.useState(existingTasks);
  const [newTask, setNewTask] = React.useState("")
  
  return <div>
    <ul>
      {tasks.map( task => 
        <li>{task}</li>
      )}
      <input value={newTask} 
        onChange={ e => {
          setNewTask(e.target.value)
        }}
        
        />
      <button onClick={ () =>{
          // tasks.push(newTask);          
          setTasks( currentTasks => [...currentTasks, newTask] )
          setNewTask("");
          localStorage["tasks"] = JSON.stringify(tasks);
        }}>Add Task</button>
    </ul>
  </div>
}
ReactDOM.render(
  <Demo existingTasks={loadedTasks} />,
  document.getElementById("app")
);
```