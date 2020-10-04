---
title: My first hook
date: "2020-10-06"
spoiler: My first hook
cta: "none"
---

I didn't like React initially primarily due to jsx. I always liked clear separation of html and js and hence I chose to learn Angular and used it on many projects. Probably I was hoping that jsx will go away one day but that never happened. Eventually I realized that Angular has its own pitfalls and there is too much ceremony in the form of modules and components. Comparing that to functional components in React, I decided to ignore my hate towards jsx and bundled down to finally learn React.

Hooks are one of the cornerstones of React functional components. As I am still learning, I still need to grasp many aspects of writing hooks. On one of my side projects, I came across a scenario where I need to use different GraphQL queries condionally and it would have been a mess to include all this login in main component (even if is was possible at all). So I sat down to write a custom hook which I will explain below.

##Problem##

There are three GraphQL queries each returning a maximum number (e.g., a primary key) from a table. We need to execute one of these three queries conditionally. The user selects the level (1 - 3) from a UI component that dictates which of the queries should be executed to get the next number.

So we start with importing required hooks from react and apollo. We are using useQuery hook from Apollo library to execute GraphQL queries. We are using a couple of useState calls to save nextNumber and currentLevel.

```jsx
import { useState, useEffect } from "react";
import { useQuery } from "@apollo/react-hooks";
import { getNextLevel1, getNextLevel2, getNextLevel3 } from "../queries";

export const useNextNumber = () => {
  const [nextNumber, setNextNumber] = useState(0);
  const [currentLevel, setCurrentLevel] = useState(0);
};
```

Next, we will add calls to useQuery for each of three queries. This is where I spent a lot of time trying different parameters and most of the time ending with all queries being fired at once. Note that we are using <code>skip</code> parameter to mention that the query should be skipped if currentLevel doesn't match the given level (1-3).

```jsx
const level1 = useQuery(getNextLevel1, { skip: currentLevel != 1 });
const level2 = useQuery(getNextLevel2, { skip: currentLevel != 2 });
const level3 = useQuery(getNextLevel3, { skip: currentLevel != 3 });
```

## Different syntax for useQuery

Usually you will see useQuery hook is used as<br/>

```jsx
const { loading, data, error } = useQuery(query);
```

But in our case, we need to use the skip flag and we are not interested in loading and error states, so we are not destructuing the result from useQuery calls.

### useEffect

Next we are going to use the useEffect hook to set the nextNumber state. In the dependency array for this hook we have listed level1.data, level2.data and level3.data. This ensures that the function inside useEffect is only executed when any of these value changes which will happen when any of the respective query is called.

```jsx
useEffect(() => {
  if (currentLevel === 0) return;
  if (level1.data) setNextNumber(+level1.data.result[0].max);
  else if (level2.data) setNextNumber(+level2.data.result[0].max);
  else if (level3.data) setNextNumber(+level3.data.result[0].max);
}, [level1.data, level2.data, level3.data]);
```

### Hook API

Finally, we need to write the function that will be called from the client code. This function takes one parameter. The level selected by the user. It then validates the level and resets the data property of each of three queries. It then calls the refetch function on query that is matching the level and sets the currentLevel. The public API of this hook is nextNumber value and the fetchNextNumber function.

```jsx
const fetchNextNumber = (level: number): void => {
  if (level === currentLevel) return;
  level1.data = level2.data = level3.data = undefined;

  if (level === 1) level1.refetch();
  else if (level === 2) level2.refetch();
  else if (level === 3) level3.refetch();
  setCurrentLevel(level);
};

return { nextNumber, fetchNextNumber };
```

Complete code for useNextNumber hook is listed below.

```jsx
import { useState, useEffect } from "react";
import { useQuery } from "@apollo/react-hooks";
import { getNextLevel1, getNextLevel2, getNextLevel3 } from "../queries";

export const useNextNumber = () => {
  const [nextNumber, setNextNumber] = useState(0);
  const [currentLevel, setCurrentLevel] = useState(0);

  const level1 = useQuery(getNextLevel1, { skip: currentLevel != 1 });
  const level2 = useQuery(getNextLevel2, { skip: currentLevel != 2 });
  const level3 = useQuery(getNextLevel3, { skip: currentLevel != 3 });

  useEffect(() => {
    if (currentLevel === 0) return;
    if (level1.data) setNextNumber(+level1.data.result[0].max);
    else if (level2.data) setNextNumber(+level2.data.result[0].max);
    else if (level3.data) setNextNumber(+level3.data.result[0].max);
  }, [level1.data, level2.data, level3.data]);

  const fetchNextNumber = (level: number): void => {
    if (level === currentLevel) return;
    level1.data = level2.data = level3.data = undefined;

    if (level === 1) level1.refetch();
    else if (level === 2) level2.refetch();
    else if (level === 3) level3.refetch();
    setCurrentLevel(level);
  };

  return { nextNumber, fetchNextNumber };
};
```

We can make this code somewhat concise by storing all three useQuery return values in an array. This can result in avoiding if blocks as we can use the level as an index to invoke the required functions. But I have preffered to keep the code simple to make it more readable.

##Mind the rules

When writing your custom hooks, you have to mind the [rules of hooks](https://reactjs.org/docs/hooks-rules.html). I started writing the above hook without going through the rules and ended up spending a fair bit of time wondering why the result if different everytime. It turned out that I was trying to run useQuery hooks inside an if block which is agains the rules.

##Feedback is welcome##

As I mentioned in the beginning, I am learning React and this is my first hook. I am sure there are better ways to do the same thing and I hope to hear and learn from you.

##Resources##

- [Hooks at a Glance](https://reactjs.org/docs/hooks-overview.html)
- [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)
- [Best practices with React Hooks](https://www.smashingmagazine.com/2020/04/react-hooks-best-practices/).

<hr/>
