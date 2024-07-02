React Testing Library (RTL) is a powerful tool for testing React components by focusing on the behavior and user interactions rather than the implementation details. Let's dive into an end-to-end training on RTL, covering from basic to advanced topics.

### **Basics of React Testing Library**

**1. What is RTL?**
RTL is a set of utilities to test React components in a way that resembles how users interact with your application. It encourages writing tests that focus on what the user sees and does.

**2. Why Use RTL?**
- Encourages best practices by testing user interactions.
- Provides utilities to query elements similar to how users interact with them.
- Reduces the reliance on implementation details.

**3. Setting Up RTL**

First, ensure you have `@testing-library/react` and `@testing-library/jest-dom` installed in your project:
```bash
npm install @testing-library/react @testing-library/jest-dom --save-dev
```

**4. Writing Your First Test**

```jsx
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import MyComponent from './MyComponent';

test('renders MyComponent', () => {
  render(<MyComponent />);
  const linkElement = screen.getByText(/hello world/i);
  expect(linkElement).toBeInTheDocument();
});
```
- `render`: Renders the component for testing.
- `screen`: Provides a way to query the DOM.
- `expect`: Used for assertions (provided by Jest).

### **Intermediate Concepts**

**1. Queries**

RTL provides various queries to find elements:
- `getBy`: Throws an error if no element is found.
- `queryBy`: Returns null if no element is found.
- `findBy`: Returns a promise and waits for the element.

```jsx
// Examples:
const button = screen.getByRole('button', { name: /submit/i });
const input = screen.queryByPlaceholderText('Enter name');
const asyncText = await screen.findByText(/loading complete/i);
```

**2. User Events**

To simulate user interactions, you can use `@testing-library/user-event`:
```bash
npm install @testing-library/user-event --save-dev
```

```jsx
import userEvent from '@testing-library/user-event';

test('user interaction', async () => {
  render(<MyComponent />);
  const button = screen.getByRole('button', { name: /submit/i });
  userEvent.click(button);
  const message = await screen.findByText(/submission successful/i);
  expect(message).toBeInTheDocument();
});
```

**3. Mocking and Spies**

For mocking functions and spying on calls, Jest provides utilities:
```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import MyComponent from './MyComponent';

test('calls onSubmit when form is submitted', () => {
  const handleSubmit = jest.fn();
  render(<MyComponent onSubmit={handleSubmit} />);
  const input = screen.getByLabelText(/name/i);
  userEvent.type(input, 'John Doe');
  const button = screen.getByRole('button', { name: /submit/i });
  userEvent.click(button);
  expect(handleSubmit).toHaveBeenCalledWith('John Doe');
});
```

### **Advanced Topics**

**1. Custom Render Functions**

You might want to wrap your components in context providers or routers. RTL allows custom render functions:
```jsx
import { render as rtlRender } from '@testing-library/react';
import { MyProvider } from './MyContext';

const render = (ui, options) => 
  rtlRender(ui, { wrapper: MyProvider, ...options });

export * from '@testing-library/react';
export { render };
```

**2. Testing Asynchronous Code**

Use `findBy` queries and `waitFor` to handle asynchronous code:
```jsx
import { render, screen, waitFor } from '@testing-library/react';
import MyComponent from './MyComponent';

test('displays data after fetching', async () => {
  render(<MyComponent />);
  await waitFor(() => expect(screen.getByText(/data loaded/i)).toBeInTheDocument());
});
```

**3. Performance Testing**

Measure the performance impact of your components:
```jsx
import { render } from '@testing-library/react';

test('performance', () => {
  const { container } = render(<MyComponent />);
  expect(container).toMatchSnapshot();
});
```

**4. Handling Redux**

For components connected to Redux, use `Provider` to wrap your component:
```jsx
import { render } from '@testing-library/react';
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import rootReducer from './reducers';
import MyComponent from './MyComponent';

const store = createStore(rootReducer);

const renderWithRedux = (component) => 
  render(<Provider store={store}>{component}</Provider>);

test('renders with redux', () => {
  renderWithRedux(<MyComponent />);
  expect(screen.getByText(/redux state/i)).toBeInTheDocument();
});
```

### **Custom Render Functions**

Custom render functions are useful for wrapping components with providers or other contexts.

#### **test-utils.js**

Create a custom render function that wraps the component with `UserProvider`.

```jsx
import React from 'react';
import { render } from '@testing-library/react';
import { UserProvider } from './context/UserContext';

const customRender = (ui, options) => {
  return render(ui, { wrapper: UserProvider, ...options });
};

export * from '@testing-library/react';
export { customRender as render };
```

Use this in your tests to avoid repetitive wrapping.

#### **App.test.js**

```jsx
import { render, screen, waitFor } from '../test-utils';
import App from '../App';
import { fetchUsers } from '../api/userApi';

jest.mock('../api/userApi');

test('renders User Management title', () => {
  render(<App />);
  const titleElement = screen.getByText(/user management/i);
  expect(titleElement).toBeInTheDocument();
});

test('fetches and displays users', async () => {
  fetchUsers.mockResolvedValueOnce(['John Doe', 'Jane Doe']);
  render(<App />);
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
  });
});
```

### **Handling Asynchronous Code**

For handling asynchronous code, use `waitFor` and `findBy` queries.

#### **App.test.js**

```jsx
import { render, screen, waitFor } from '../test-utils';
import App from '../App';
import { fetchUsers } from '../api/userApi';

jest.mock('../api/userApi');

test('fetches and displays users', async () => {
  fetchUsers.mockResolvedValueOnce(['John Doe', 'Jane Doe']);
  render(<App />);
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
  });
});
```

### **Testing Redux**

When using Redux, wrap your component with the Redux provider.

#### **redux/store.js**

Set up a Redux store.

```jsx
import { createStore } from 'redux';
import rootReducer from './reducers';

const store = createStore(rootReducer);

export default store;
```

#### **redux/reducers.js**

Define a simple reducer.

```jsx
const initialState = {
  users: []
};

const rootReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ADD_USER':
      return { ...state, users: [...state.users, action.payload] };
    default:
      return state;
  }
};

export default rootReducer;
```

#### **redux/actions.js**

Define actions.

```jsx
export const addUser = (user) => ({
  type: 'ADD_USER',
  payload: user
});
```

#### **App.js**

Update `App.js` to use Redux.

```jsx
import React, { useEffect } from 'react';
import { useDispatch } from 'react-redux';
import UserForm from './components/UserForm';
import UserList from './components/UserList';
import { fetchUsers } from './api/userApi';
import { addUser } from './redux/actions';

const App = () => {
  const dispatch = useDispatch();

  useEffect(() => {
    fetchUsers().then((data) => {
      data.forEach(user => dispatch(addUser(user)));
    });
  }, [dispatch]);

  return (
    <div>
      <h1>User Management</h1>
      <UserForm />
      <UserList />
    </div>
  );
};

export default App;
```

#### **App.test.js**

Update the test to use Redux.

```jsx
import { render, screen, waitFor } from '../test-utils';
import { Provider } from 'react-redux';
import store from '../redux/store';
import App from '../App';
import { fetchUsers } from '../api/userApi';

jest.mock('../api/userApi');

test('fetches and displays users', async () => {
  fetchUsers.mockResolvedValueOnce(['John Doe', 'Jane Doe']);
  render(
    <Provider store={store}>
      <App />
    </Provider>
  );
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
  });
});
```

### **Performance Testing**

Performance testing ensures your components render efficiently.

#### **App.test.js**

Use `Profiler` to measure performance.

```jsx
import React from 'react';
import { render, screen, waitFor } from '../test-utils';
import { Profiler } from 'react';
import App from '../App';
import { fetchUsers } from '../api/userApi';

jest.mock('../api/userApi');

const onRender = (id, phase, actualDuration) => {
  console.log(`Rendering ${id} took ${actualDuration}ms in ${phase} phase`);
};

test('fetches and displays users with performance profiling', async () => {
  fetchUsers.mockResolvedValueOnce(['John Doe', 'Jane Doe']);
  render(
    <Profiler id="App" onRender={onRender}>
      <App />
    </Profiler>
  );
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
  });
});
```

### **Code Coverage and Reporting**

Ensure you have sufficient test coverage.

#### **package.json**

Add a script for coverage.

```json
"scripts": {
  "test": "react-scripts test --coverage"
}
```

#### **Run Coverage Report**

```bash
npm run test
```

This generates a coverage report in the `coverage` directory. You can view it in the browser.

### **Advanced Queries and User Events**

#### **App.test.js**

Use more advanced queries and user events.

```jsx
import { render, screen, waitFor, fireEvent } from '../test-utils';
import App from '../App';
import { fetchUsers } from '../api/userApi';
import userEvent from '@testing-library/user-event';

jest.mock('../api/userApi');

test('renders and interacts with user form', async () => {
  fetchUsers.mockResolvedValueOnce([]);
  render(<App />);
  
  // Ensure the initial state is empty
  expect(screen.queryByText(/john doe/i)).toBeNull();
  
  // Interact with form
  const input = screen.getByPlaceholderText(/enter name/i);
  userEvent.type(input, 'John Doe');
  const button = screen.getByText(/add user/i);
  userEvent.click(button);

  // Check if user is added
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
  });
});
```

## Real-life scenario of using React Testing Library (RTL) in an enterprise application. Example of a user management system

### **File Structure**

Here's a simple file structure for our application:

```
/user-management
├── public
│   └── index.html
├── src
│   ├── api
│   │   └── userApi.js
│   ├── components
│   │   ├── UserForm.js
│   │   ├── UserList.js
│   │   └── UserListItem.js
│   ├── context
│   │   └── UserContext.js
│   ├── App.js
│   ├── index.js
│   ├── setupTests.js
│   └── __tests__
│       ├── App.test.js
│       ├── UserForm.test.js
│       └── UserList.test.js
└── package.json
```

### **Component Implementation**

#### **UserForm.js**

```jsx
import React, { useState, useContext } from 'react';
import { UserContext } from '../context/UserContext';

const UserForm = () => {
  const [name, setName] = useState('');
  const { addUser } = useContext(UserContext);

  const handleSubmit = (e) => {
    e.preventDefault();
    addUser(name);
    setName('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Enter name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <button type="submit">Add User</button>
    </form>
  );
};

export default UserForm;
```

#### **UserList.js**

```jsx
import React, { useContext } from 'react';
import { UserContext } from '../context/UserContext';
import UserListItem from './UserListItem';

const UserList = () => {
  const { users } = useContext(UserContext);

  return (
    <ul>
      {users.map((user, index) => (
        <UserListItem key={index} user={user} />
      ))}
    </ul>
  );
};

export default UserList;
```

#### **UserListItem.js**

```jsx
import React from 'react';

const UserListItem = ({ user }) => {
  return <li>{user}</li>;
};

export default UserListItem;
```

### **Context and State Management**

#### **UserContext.js**

```jsx
import React, { createContext, useState } from 'react';

export const UserContext = createContext();

export const UserProvider = ({ children }) => {
  const [users, setUsers] = useState([]);

  const addUser = (name) => {
    setUsers([...users, name]);
  };

  return (
    <UserContext.Provider value={{ users, addUser }}>
      {children}
    </UserContext.Provider>
  );
};
```

### **API Mocking**

#### **userApi.js**

```jsx
export const fetchUsers = () => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(['John Doe', 'Jane Doe']);
    }, 1000);
  });
};
```

### **Application Entry Point**

#### **App.js**

```jsx
import React, { useEffect, useContext } from 'react';
import UserForm from './components/UserForm';
import UserList from './components/UserList';
import { UserProvider, UserContext } from './context/UserContext';
import { fetchUsers } from './api/userApi';

const App = () => {
  const { addUser } = useContext(UserContext);

  useEffect(() => {
    fetchUsers().then((data) => {
      data.forEach(user => addUser(user));
    });
  }, [addUser]);

  return (
    <UserProvider>
      <div>
        <h1>User Management</h1>
        <UserForm />
        <UserList />
      </div>
    </UserProvider>
  );
};

export default App;
```

#### **index.js**

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { UserProvider } from './context/UserContext';

ReactDOM.render(
  <UserProvider>
    <App />
  </UserProvider>,
  document.getElementById('root')
);
```

#### **setupTests.js**

```jsx
import '@testing-library/jest-dom';
```

### **Testing with React Testing Library**

#### **App.test.js**

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import App from '../App';
import { fetchUsers } from '../api/userApi';
import { UserProvider } from '../context/UserContext';

jest.mock('../api/userApi');

test('renders User Management title', () => {
  render(
    <UserProvider>
      <App />
    </UserProvider>
  );
  const titleElement = screen.getByText(/user management/i);
  expect(titleElement).toBeInTheDocument();
});

test('fetches and displays users', async () => {
  fetchUsers.mockResolvedValueOnce(['John Doe', 'Jane Doe']);
  render(
    <UserProvider>
      <App />
    </UserProvider>
  );
  await waitFor(() => {
    expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    expect(screen.getByText(/jane doe/i)).toBeInTheDocument();
  });
});
```

#### **UserForm.test.js**

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import UserForm from '../components/UserForm';
import { UserProvider } from '../context/UserContext';

test('renders UserForm and adds a user', () => {
  render(
    <UserProvider>
      <UserForm />
    </UserProvider>
  );

  const input = screen.getByPlaceholderText(/enter name/i);
  fireEvent.change(input, { target: { value: 'John Doe' } });

  const button = screen.getByText(/add user/i);
  fireEvent.click(button);

  const user = screen.getByText(/john doe/i);
  expect(user).toBeInTheDocument();
});
```

#### **UserList.test.js**

```jsx
import { render, screen } from '@testing-library/react';
import UserList from '../components/UserList';
import { UserProvider } from '../context/UserContext';

test('renders UserList with users', () => {
  const users = ['John Doe', 'Jane Doe'];
  render(
    <UserProvider value={{ users }}>
      <UserList />
    </UserProvider>
  );

  const john = screen.getByText(/john doe/i);
  const jane = screen.getByText(/jane doe/i);
  expect(john).toBeInTheDocument();
  expect(jane).toBeInTheDocument();
});
```

### **Conclusion**

React Testing Library shifts the focus from testing implementation details to testing the actual user interactions, making your tests more robust and maintainable. By following this guide, you can write comprehensive tests that cover various aspects of your React components, ensuring their reliability and performance.
