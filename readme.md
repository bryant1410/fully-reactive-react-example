# React + RxJS: going fully reactive

### React is _not_ reactive

Nowadays React is getting more and more popular. However, despite the name, it is **not** fully reactive solution. As Andre Staltz says - [only rendering is reacitve in React](http://staltz.com/dont-react/#/12). But can we change that? Can we make a react components _more_ reactive?

### Going reactive in the component

Imagine a simple counter written in plain, old React:
```js
import React from 'react';
import ReactDOM from 'react-dom';

class Main extends React.Component {
  constructor (props) {
    super(props);
    this.state = {count: 0};
  }

  //This is an "API" method of the component
  updateCount (by) {
    this.setState({
      count: this.state.count + by
    });
  }

  render () {
    return (
        <div>
          <button onClick={this.updateCount.bind(this, 1)}>+</button>
          <button onClick={this.updateCount.bind(this, -1)}>-</button>
          <div>count: {this.state.count}</div>
        </div>
    )
  }
}

ReactDOM.render(<Main/>, document.getElementById('app'));
```

Can you spot referencing a component's API method `updateCount(by)`? This is **proactive** approach. The method will be called by the children "components" (in this case - buttons) whenever a click action is triggered upon any of them.

Now what if we could turn the buttons' click events into observables? See the code below:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import Rx from 'rx';

class Main extends React.Component {
  constructor (props) {
    super(props);
    this.state = {count: 0};
  }

  // Click events are now observables! No more proactive approach.
  componentDidMount () {
    const plusBtn = document.getElementById('plus');
    const minusBtn = document.getElementById('minus');

    const plus$ = Rx.Observable.fromEvent(plusBtn, 'click').map(e => 1);
    const minus$ = Rx.Observable.fromEvent(minusBtn, 'click').map(e => -1);

    Rx.Observable.merge(plus$, minus$).scan((acc, n) => acc + n)
      .subscribe(value => this.setState({count: value}));
  }

  render () {
    return (
        <div>
          <button id="plus">+</button>
          <button id="minus">-</button>
          <div>count: {this.state.count}</div>
        </div>
    );
  }
}

ReactDOM.render(<Main/>, document.getElementById('app'));
```

Take a look at the `componentDidMount()`. It's a react component's standard lifecycle method called only once, just after the element is added to the DOM. Here we are creating two observable streams of click events (`plus$` and `minus$`) `map`ped into corresponding values. Then we `merge` them into single stream. Each time an event is emitted, a `scan` is called and most-up-to-date state is set upon the component.

No more passing method reference around, no more proactive approach. Here, we went _fully reactive_.

Additional reading:
- Andre Staltz on [react and why it's not fully reactive](http://staltz.com/dont-react/#/) (although the title of the presentation is.. _radical_)
- [Observables and reactive programming; proactive vs reactive explained](http://cycle.js.org/observables.html)
- [RxMarbles](http://rxmarbles.com/#scan) - `map`, `merge` and `scan` explained
