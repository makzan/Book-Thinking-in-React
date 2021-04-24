In the HTML, we have an element as a placeholder.

```html
<div id="app"></div>
```

Our main element will be <HSLGame /> we use RenderDOM to render this element into the #app placeholder.

```javascript
ReactDOM.render(
  <HSLGame />,
  document.getElementById('app')
);
```


## Data

In our game, there are 7 states (variables) that we need to keep track to. They are divided into 3 groups: the question of HSL, the current HSL, and the scores.

In particular, they are:

- questionH
- questionS
- questionL
- hue
- saturation
- lightness
- scores

The HSLGame function is mainly divided into the following parts.

```js
function HSLGame(){
	// setStates for the 7 variables mentioned above.
  
  function checkColorScore() {
    // A function to check color score once the button is clicked.
  }
  function generateNewQuestion() {
		// Generate new question HSL once the check color button is clicked.
  }
  
  return <div>
    The UI goes here, including 
		1. a ScoreBoard, 
		2. two ColorBox components, 
		3. three sliders for HSL, and a check color button
  </div>;
}
```

The setState for the 7 variables.

```js
const [questionH, setQuestionH] = React.useState(0);  
const [questionS, setQuestionS] = React.useState(50);  
const [questionL, setQuestionL] = React.useState(50);  

const [hue, setHue] = React.useState(180);
const [saturation, setSaturation] = React.useState(50);
const [lightness, setLightness] = React.useState(50);
  
const [scores, setScores] = React.useState([]);
```

The color checking function.

```js
function checkColorScore() {
  let diffH = Math.min(Math.abs(questionH-hue), Math.abs(questionH+360-hue));    
  let diff = Math.abs(diffH + (questionS-saturation) + (questionL-lightness));
  let score = Math.max(100-diff, 0);    
  setScores(scores => [...scores, score]); 
}
```

The `...` is called spread operator. It spread the given array into each array items. Think of `[...scores, score]` as creating a new array with the original items in scores array and also the new score item. It generates a new array.

Every time when we call the set state function, we are given the current value and we set a new value to it. They are meant to be immutable. That’s why we want to set a new array to the scores instead of modifying the existing scores array.

More on spread operation: https://javascript.plainenglish.io/how-three-dots-manipulate-arrays-in-javascript-c664fea01bd8

The question generation function.

```js
function generateNewQuestion() {
  let newHue = Math.round(Math.random()*360);    
  setQuestionH(newHue);  
  
  let newS = Math.round(Math.random()*50+25);
  setQuestionS(newS)
  
  let newL = Math.round(Math.random()*50+25);
  setQuestionS(newL)
}
```

Here is the full HSLGame function

```js
function HSLGame(){
  const [questionH, setQuestionH] = React.useState(0);  
  const [questionS, setQuestionS] = React.useState(50);  
  const [questionL, setQuestionL] = React.useState(50);  
  
  const [hue, setHue] = React.useState(180);
  const [saturation, setSaturation] = React.useState(50);
  const [lightness, setLightness] = React.useState(50);
  
  const [scores, setScores] = React.useState([]);
  
  function checkColorScore() {
    let diffH = Math.min(Math.abs(questionH-hue), Math.abs(questionH+360-hue));    
    let diff = Math.abs(diffH + (questionS-saturation) + (questionL-lightness));
    let score = Math.max(100-diff, 0);    
    setScores(scores => [...scores, score]); 
  }
  function generateNewQuestion() {
    let newHue = Math.round(Math.random()*360);    
    setQuestionH(newHue);  
    
    let newS = Math.round(Math.random()*50+25);
    setQuestionS(newS)
    
    let newL = Math.round(Math.random()*50+25);
    setQuestionS(newL)
  }
  
  
  return <div>
    <ScoreBoard scores={scores} />
    <ColorBox h={questionH} s={questionS} l={questionL} />
    <ColorBox h={hue} s={saturation} l={lightness} />
    
    <input className="rainbow" type="range" min="0" max="360"
       value={hue}
       onChange={(e)=>{
        setHue(e.target.value*1)
      }}
      />
    <input type="range" min="0" max="100"
       value={saturation}
       onChange={(e)=>{
        setSaturation(e.target.value*1)
      }}
      />
    <input type="range" min="0" max="100"
       value={lightness}
       onChange={(e)=>{
        setLightness(e.target.value*1)        
      }}
      />
    <button onClick={(e)=>{
        checkColorScore();
        generateNewQuestion();
      }}
      >Check Color</button>
  </div>;
}
```

The ColorBox function component.

```js
function ColorBox(props) {
  var bgColor = `hsl(${props.h}, ${props.s}%, ${props.l}%)`;
  var fgColor = (props.l < 50) ? "white" : "black";
  var style = {
    background: bgColor,
    color:fgColor
  };
  return <output style={style}>    
  </output>
}
```

The ScoreBoard function component.

```js
function ScoreBoard(props) {
  let sum = 0;
  let average = 0;
  for (let score of props.scores) {
    sum += score
  }
  if (props.scores.length > 0) {
    average = sum / props.scores.length
  }
  return <p>
      Score: {props.scores.join(", ") }<br/>
      Average: { average }
    </p>
}
```