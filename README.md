# React-Ts-Best-Practices

Documentation for best practices to use with React with Typescript. Note that this documentation builds off of <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>, so we won't mention practices that are already mentioned there.
<br/><br/>


## The project folder structure

### Project Folders Overview

```
- app
-   public/
-   src/
-     assets/
-     common/
-     components/
-       lg/
-       md/
-       sm/
-     hooks/
-     models/
-     pages/
-     styles/
-     util/
-     App.test.tsx
-     App.ctx.tsx
-     App.tsx
-     index.css
-     index.tsx
-     react-app-env.d.ts
-     reportWebVitals.ts
-   .env
-   .eslintrc.json
-   .gitignore
-   README.md
-   package.json
-   tsconfig.json
```

### Folders Explained (files generated by create-react-app are not listed)
- `public/` generated by create-react-app (holds assets like favicons)
- `src/` generated by react
-   `assets/` downloaded assets like images, third-party fonts etc
-   `common/` shared miscellaneous items like a `Paths.ts` which holds all the route string values of your APIs.
-   `components/` custom react components (might not be shared like a navigation bar or shared such as a styled button you used in multiple places)
      - `/lg` single components that take up multiple files
      - `/md` single components that take up one file
      - `/sm` multiple components per file
-   `/hooks` custom hooks (i.e. useSetState.ts)
-   `/models` files for description your data (i.e. User.ts)
-   `/pages` the various pages of your application.
  -   <b>NOTE:</b> You should try to structure your `pages/` folders as close as possible in the same way as they are navigated to by the user. So if your site is like `https://my-site.com/home`, `https://my-site.com/account`, `https://my-site.com/posts/edit`, and `https://my-site.com/posts/new` the `pages/` folder should look like how it does in <b>Snippet 1</b>. Of course this is not always possible and it's normal to not follow this to-a-tee. For example, you might have the `id` of a `Post` record inserted somewhere in your url, so it can be automatically selected when the user refreshes the browser and 'View' might be the default view for a particular post that's selected (i.e. `https://my-site.com/posts/9Z8AO5C844R` displays the <View/> component).
-   `styles/` various shared styles (i.e. Colors.ts)z
-   `util/` miscellanous shared logic. Could be modules or inventory scripts .

### Structuring your app
- In addition to what's listed above, you should name your files/folders after the React component they mean to represent. If a file represents a single-component, and there are not multiple files for that component (that includes test files and child components), name that file after the component. However, for large components that include multiple files, name the folder after the component, and name the core file `index.ts`, which is the JavaScript convention for naming the file in a folder where everything else is exported from (see <b>Snippet 1</b>).
- If you have items (child-components, functions, etc) that you know will never be used beyond the scope of a certain pages folder, you can create folders named `common/` in addition to the root `src/common/` folder. 

### Snippet 1
```
- components/
- common/
-   Paths.ts
- models/
-   User.ts
-   Post.ts
- pages/
-   Home/ (https://my-site.com/home)
-     index.tsx
-     index.test.tsx
-   Account/ (https://my-site.com/account)
-     UpdatePaymentForm/
-       index.tsx
-       index.test.tsx
-     index.tsx // <- Imports the <UpdatePaymentForm/> component
-     index.test.tsx
-   Posts/ (https://my-site.com/posts)
-     common/
-       PostForm.tsx // Used by both New and Edit pages
-       types.ts // Has stuff that's used by both the View, Edit, and New pages
-     Edit/ (https://my-site.com/posts/9Z8AO5C844R/edit)
-       index.test.tsx
-       index.tsx
-     New/ (https://my-site.com/posts/new)
-       index.test.tsx
-       index.tsx
-     View/ (https://my-site.com/posts/9Z8AO5C844R)
-       index.test.tsx
-       index.tsx // Displays a particular post
-     index.tsx // Displays the <PostsTable/> component if no singular post is selected.
-     index.ctx.tsx
-     index.test.tsx
-     PostsTable.tsx 
```


## Basics of Functional-Components

### Declaring
- Use PascalCase for naming functional-components.
- Use functions for declaring components not classes. Procedural/functional programming is the dominant trend in JavaScript and is much more convenient for making smaller components. Use function declarations (`function`) not arrow functions for making components so that they are hoisted.
- For components declared in the same file, follow a top down approach. That is, declare children components below the parent so that the pattern of programming for both for doing logic and creating elements stays consistent.
- If you use TypeScript, always typesafe a functional-component's properties. For large/complex components, create an interface for the props argument (i.e. `IProps`) and place it in the `// **** Types **** //` section of the file (see <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>). If you're using JSDoc, you should also typesafe the properties for shared components, but for single using components (like a the file for a page) specifying the types might be overkill. If a jsdoc custom type also gets large/complex, you could also place it in the `Types` section of your file. Both for typescript and jsdoc, you don't need to specify the return type for functional components cause its always JSX.Element. A good way to remember these rules is that whenever another developer needs to use the component you created, your typesafety should reflect that.

### Organizing the code of a functional-component
- Don't place static values inside functional-components, if you do they'll have to be reinitialized everytime the component state is updated, which could affect performance for large complex applications. Place them at the top of the file under the `// **** Variables **** //` section (see <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>).
- If a function inside a functional-component is large and its logic does not need to change with the component, move it outside the component and put it in the `// **** Helper Functions **** //` section at the bottom of the file: this is will stop the logic from needing to be reinitialized each time.
- When posititing sibling-components in relation to each other, do the positioning in the parent-component, that way all the positioning between siblings can be seen at once and we don't have to dig into the code of each individual child component to move them (see <b>Snippet 2</b>).
- Although comments (not spaces) should generally be used to separate chunks of logic within traditional functions (as mentioned in <a href="https://github.com/seanpmaxwell/Typescript-Best-Practices">Typescript best practices</a>), for jsx component-functions, we can use spacing to separate chunks of logic. Use single-spaces + comments to separate hook calls, related DOM elements with the `return` statement, and initializing related variables (see <b>Snippet 3</b>).

### Component properties
- Extract the component properties at the top of the function-component from the `props` param, don't use `props` in a bunch of places to access values. This makes it easier to intialize default values when a property is undefined and makes the code more robust because you can see if a property is no longer being used but might still be getting passed down by parent-component. Another aspect to this is that when creating a functional-component that wraps around another functional-component, it's generally a good idea to mimick the child properties as much as you can so that way you don't have to recreate/redeclare these properties again (see <b>Snippet 2</b> and <b>Snippet 3</b>).

### Snippet 2 
```.tsx
import Box, { BoxProps } from '@mui/material/Box';

function Parent() {
  return (
    <Box>
      <Child1 mb={1}/>
      <Child2/>
      <SomeOtherChild/>
    </Box>
  );
}


// GOOD

interface IProps1 extends BoxProps {
  name?: string;
  posts?: string[];
}

function Child1(props: IProps1) {
  const {
    name = '',
    posts = [],
    ...otherProps
  } = props;
  return (
    <Box {...otherProps}>
      Name: {name} Posts: {posts.length}
    </Box>
  );
}


// BAD

interface IProps2 {
  name?: string;
  posts?: string[];
}

function Child2(props: IProps2) {
  return (
    <Box mb={2}>
      Name: {props.name ?? ''} Posts: {props.posts?.length ?? 0}
    </Box>
  );
}
```

- In complying with TypeScript best practices, use function-declarations for functions at the top scope of a file and arrow-functions if a function is declared inside of another function:
```typescript
function Parent() {
  return (
    <Child
      onClick={() => doSomething()} // GOOD
      onMouseDown={function () { ..doSomething }} // BAD
    />
  );
}
```

### Snippet 3

```.tsx
import { useCallback } from 'react';
import axios from 'axios';

import Box, { BoxProps } from '@mui/material-ui/Box';
import Button, { ButtonProps } from '@mui/material-ui/Button';

import Indicator from 'components/md/Indicator';
import Colors from 'styles/Colors.ts';


// **** Functional Components **** //

/**
 * Login a User
 */
function LoginForm(props: BoxProps) {
  const {
    sx,
    ...otherProps
  } = props;

  // Misc
  const navigate = useNavigate();

  // Init state
  const [ state, setState ] = useSetState({
    username: '',
    password: '',
    isLoading: false,
  });

  // Call the "submit" API
  const submit = useCallback(async () => {
    setState({ isLoading: true });
    const success = await axios.post({
      username: state.username,
      password: state.password,
    });
    setState({ isLoading: false });
    if (success) {
      navigate('/account');
    }
  }, [setState, state.username, state.password, navigate]);

  // Return
  return (
    <Box
      sx={{
        position: 'relative',
        backgroundColor: Colors.Bkg.Grey, // don't do '#808080'
        ...sx,
      }}
      {...otherProps}
    >
     {/* Indicator */}
     {state.isLoading &&
       <Indicator/>
     }

     {/* Input Fields */}
     <TextField
       type="text"
       value={state.username}
       onChange(e => setState({ username: e.currentTargetValue })}
     />
     <TextField
       type="password"
       value={state.password}
       onChange(e => setState({ password: e.currentTargetValue })}
     />
     
     {/* Action Buttons */}
      <Button
        color="error"
        onClick={() => navigate('/home')}
      >
        Cancel
      </Button>
      <Button
        color="primary"
        onClick={() => submit()}
      >
        Login
      </Button>
    </Box>
  );
}
```
<br/>


## Handling state managment
- State management is done using `useState`, `useContext`, and maybe `redux` if you have it. We'll cover the best practices for all of them.

### useState()
- For small components that only have one or two state values, using `useState` directly is fine, but once a component starts to have large numbers of state values, using a custom hook to handle all the state values as a single object will make your code much more readable and easier to manage. There might be libraries for this or you copy and paste the source for the custom hook <a href="https://github.com/seanpmaxwell/useSetState/blob/main/src/useSetState.ts">here</a>.
- This will make your code more readable cause now all variables that belong to the local state will began with `state`, and you only need one function managing them `useState()`.

### useContext()
- If a state value in a parent component only needs to go down one layer to a child component that exists in the same file, then passing it through the function properties (props) is fine; `context` or `redux` is probably overkill. If however you have a large/complex component that needs to pass data to multiple children, spread across different files, then don't use props, use `context` or `redux` instead.
- If your component contains both a large amount of jsx code and a lot of logic as well whose data needs to be passed down, it might be worth it break your context and your jsx code into different files. You should append these files with `ctx.tsx`. For example, suppose your App.tsx file contains a lof of jsx code and a lot of logic for managing the user sessions, you could create a seperate App.ctx.tsx file which uses `createContext()` whose default export is the context's provider (see <b>Snippet 4</b>).

### Snippet 4
```.tsx
// App.ctx.tsx

import { createContext } from 'react';

export const AppCtx = createContext({});

function AppProvider(props) {
  const { children } = props,
    [ session, setSession ] = useState({});

  const resetSessionData = useCallback(newData => {
    const newSession = ...bunch of logic
    setSession(newSession);
  }, [setSession])

  return (
    <AppCxt.Provider value={{
      session,
      resetSessionData: val => resetSessionData(val),
    }}>
      {children}
    </AppCxt.Provider>
  );
}

export default AppProvider;


// App.tsx

import AppProvider from 'App.ctx';

function App() {
  return (
    <div>
      <AppProvider>
        <NavBar/>
        <Home/>
        ...some large amount of jsx code
      </AppProvider>
    </div>
  );
}

export default App;


// Navbar.tsx

import { AppCtx } from '../App.ctx';

function Navbar() {
  const { session } = useContext(AppCtx);
  return (
    <div>Hello {session.name}</div>
  );
}

export default Navbar;

```
<br/>


## Misc Code Styling Rules

### Conditional Elements

- Don't need to wrap DOM elements in parenthesis for `&&`. Do use parenthesis for ternary-statements though:
  
```.tsx
// BAD
{isLoading && (
  <div>
    <Indicator allPage />
  </div>
)}

// GOOD
{isLoading && 
  <div>
    <Indicator allPage />
  </div>
}

// GOOD
{(isError && !!errMsg) ? (
  <div>
    {errMsg}
  </div>
) : (
  <div>
    Foo Bar
  </div>
)}
```

### Styling the UI
- Rules could vary depending on which library you decided to use, but generally don't hardcode hex color strings directly inside jsx elements. To keep your styling consistent, have a `src/styles/Colors.ts` file which exports a single object containing all your colors (see <b>Snippet 3</b>).
