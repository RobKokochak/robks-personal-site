---
title: useContext Hook - Avoiding Prop Drilling in React
pubDate: Feb 16 2025
tag: Engineering
draft: false
---
# useContext Hook - Avoiding Prop Drilling in React

React components pass data in a direct-descendant uni-directional flow, meaning parent components pass data to direct child components as 'props' (i.e. properties).
This is convenient from a logical perspective, but it can create some problems for very complex trees of nested components.

One common problem with this "passing props" architecture is the phenomenon of [prop drilling](https://react.dev/learn/passing-data-deeply-with-context#the-problem-with-passing-props).
Prop drilling is a situation in React where props are passed through multiple intermediate components that don't need those props themselves, just to get the data to a deeper component in the tree.

For example, imagine this component hierarchy:
```jsx
App
└── ParentComponent (holds the data)
    └── MiddleComponent
        └── ChildComponent1
            └── ChildComponent2
                └── DeepChildComponent (needs the data)
```

If `ParentComponent` has some data that only `DeepChildComponent` needs, you would have to pass it through each the intermediate components to allow `DeepChildComponent` to access it.
This is prop drilling - `MiddleComponent`, `ChildComponent1`, and `ChildComponent2` are just passing the prop along without using it themselves.

As an app grows, this can lead to cluttered, difficult to maintain code, especially for components with a lot of nested components
which depend on data being stored at the root component.

## Enter Context
Thankfully, React provides a mechanism called [Context](https://react.dev/learn/passing-data-deeply-with-context#context-an-alternative-to-passing-props),
which allows a parent component to provide data to the
children in its component tree without passing it directly as props. In the following example, I'll show a before
and after to illustrate how using Context helps to provide more clarity and flexibility for nested components.

To illustrate this, I've provided a simple example of a card game manager, where groups of teams are assigned cards from a standard 52-card deck. Feel free to play
around with the example to get a feel for how it works. Notice that when a card is assigned to a team, it is no longer available as an option to other teams.

<iframe src="https://codesandbox.io/embed/9prjzp?view=preview&module=/src/ItemManager.tsx,/src/ParentGroup.tsx,/src/GroupTeam.tsx&fontsize=13"
  style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
  title="useContext example - after"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

There are three main components here: `ItemManager` (the root component, where the data must be stored), `ParentGroup`, and `GroupTeam`. `ItemManager` contains ParentGroups, which each
contain GroupTeams.

**Before: Prop Drilling Approach**
<iframe src="https://codesandbox.io/embed/l6h39k?view=editor&module=/src/ItemManager.tsx,/src/ParentGroup.tsx,/src/GroupTeam.tsx&fontsize=13"
  style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
  title="useContext example - before"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

Notice the long list of cluttered props being passed in each of the child components. Although `ParentGroup` isn't using `availableItems` nor `setAvailableItems`, it must
receive them and pass them along, as `GroupTeam` needs them.

Now check out how much cleaner/easier to read it becomes when `useContext` is used.

**After: using createContext and useContext**
<iframe src="https://codesandbox.io/embed/9prjzp?view=editor+%2B+preview&module=/src/ItemManager.tsx,/src/ParentGroup.tsx,/src/GroupTeam.tsx&fontsize=13"
  style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
  title="useContext example - after"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

Now, `ParentGroup` is only passed the the one prop that is most relevant to it, `thisGroup`. The `groups` and `items` data are instead abstracted
into providers which wrap the parent component, and are imported in the child components as needed.

In each of the components, the props passed in are de-cluttered significantly, and the ones which remain are only the most relevant/dependent.
This helps tremendously for readability and maintainability.

There are a lot more use-cases for Context and a lot of flexibility for how to configure it. Be sure to check out
the official React documentation/tutorial on Context [here](https://react.dev/learn/passing-data-deeply-with-context).
I hope this helps you in managing your React code, and thanks for reading!
