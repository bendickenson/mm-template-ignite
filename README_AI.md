# Ignite React Native - AI Development Guide

## Template Overview

This is the **Ignite boilerplate** - a battle-tested React Native template maintained by Infinite Red since 2016. It's designed for building production-ready mobile apps with best practices baked in.

**Key Technologies:**
- React Native 0.81 (with Expo modules enabled)
- TypeScript 5
- MobX-State-Tree (state management)
- React Navigation v7
- Expo SDK v54
- Custom component library with theming support

## File Structure

```
app/
├── app.tsx              # Root component with providers
├── screens/             # Screen components (one per route)
│   ├── WelcomeScreen.tsx
│   ├── LoginScreen.tsx
│   └── DemoShowroomScreen/
├── components/          # Reusable UI components
│   ├── Button.tsx       # Themed button with presets
│   ├── Card.tsx         # Container component
│   ├── Text.tsx         # Text with typography presets
│   ├── TextField.tsx    # Form input
│   ├── Icon.tsx         # Icon component
│   ├── Screen.tsx       # Screen wrapper with safe areas
│   ├── Header.tsx       # Screen header
│   ├── ListItem.tsx     # List item component
│   ├── EmptyState.tsx   # Empty state UI
│   └── AutoImage.tsx    # Automatic image sizing
├── navigators/          # Navigation configuration
│   ├── AppNavigator.tsx # Main navigation stack
│   └── navigationUtilities.ts
├── models/              # MobX-State-Tree models (NOT in boilerplate by default, but recommended location)
│   └── RootStore.ts     # Root store setup
├── services/            # API clients and utilities
│   └── api/
│       └── api.ts       # API client (apisauce)
├── theme/               # Theming system
│   ├── colors.ts        # Color palette
│   ├── colorsDark.ts    # Dark mode colors
│   ├── spacing.ts       # Spacing scale
│   ├── typography.ts    # Font definitions
│   └── context.tsx      # Theme provider
├── i18n/                # Internationalization (i18next)
│   └── en.ts            # English translations
├── utils/               # Helper functions
│   ├── storage.ts       # MMKV storage wrapper
│   └── formatDate.ts    # Date formatting
├── config/              # App configuration
└── context/             # React Context providers
    └── AuthContext.tsx  # Example auth context
```

## AI Development Guidelines

### 1. Creating New Screens

**Location:** `app/screens/`

**Pattern:**
```typescript
import { FC } from "react"
import { observer } from "mobx-react-lite"
import { ViewStyle } from "react-native"
import { Screen, Text, Button } from "../components"
import { AppStackScreenProps } from "../navigators"

interface MyScreenProps extends AppStackScreenProps<"MyScreen"> {}

export const MyScreenObserver: FC<MyScreenProps> = observer(function MyScreen(props) {
  const { navigation } = props

  return (
    <Screen
      preset="scroll"
      safeAreaEdges={["top"]}
      contentContainerStyle={$screenContainer}
    >
      <Text preset="heading" tx="myScreen.title" />
      <Text tx="myScreen.description" />

      <Button
        preset="reversed"
        tx="common.continue"
        onPress={() => navigation.navigate("NextScreen")}
      />
    </Screen>
  )
})

const $screenContainer: ViewStyle = {
  paddingHorizontal: 16,
  paddingVertical: 24,
}
```

**Key Points:**
- Use `observer` wrapper from mobx-react-lite
- Extend `AppStackScreenProps<"ScreenName">` for type safety
- Use the `<Screen>` component wrapper (NOT View as root)
- Screen presets: `"auto"` | `"scroll"` | `"fixed"`
- Use safe area edges appropriately
- Export as named export (not default)

**Register in Navigator:**
```typescript
// In app/navigators/AppNavigator.tsx
<Stack.Screen name="MyScreen" component={MyScreenScreen} />
```

### 2. Using Components

**ALWAYS use Ignite's component library instead of raw React Native components:**

| Use This | Instead Of |
|----------|-----------|
| `<Text>` | `<RNText>` |
| `<Button>` | `<Pressable>` |
| `<TextField>` | `<TextInput>` |
| `<Icon>` | SVG or custom icons |
| `<Screen>` | `<View>` (as screen root) |
| `<Card>` | Custom container Views |

**Component Examples:**

```typescript
// Text with presets
<Text preset="heading">Title</Text>
<Text preset="subheading">Subtitle</Text>
<Text preset="default">Body text</Text>

// Button with presets
<Button preset="default" text="Click Me" onPress={handlePress} />
<Button preset="filled" text="Primary Action" />
<Button preset="reversed" text="Secondary" />

// TextField
<TextField
  label="Email"
  placeholder="Enter email"
  value={email}
  onChangeText={setEmail}
  keyboardType="email-address"
  autoCapitalize="none"
/>

// Icon
<Icon icon="back" size={24} />
<Icon icon="check" color={colors.palette.primary500} />

// Card
<Card
  preset="default"
  heading="Card Title"
  content="Card content here"
  footer={<Button text="Action" />}
/>
```

### 3. State Management with MobX-State-Tree

Ignite uses **MobX-State-Tree** for state management - NOT Redux, NOT Context API, NOT useState for global state.

**Important Note:** The boilerplate doesn't include MobX-State-Tree models by default, but it's the recommended approach for state management in Ignite apps.

**Creating a Model:**
```typescript
// app/models/Todo.ts
import { Instance, SnapshotOut, types } from "mobx-state-tree"

export const TodoModel = types
  .model("Todo")
  .props({
    id: types.identifier,
    text: types.string,
    done: types.boolean,
  })
  .actions((self) => ({
    toggle() {
      self.done = !self.done
    },
    setText(text: string) {
      self.text = text
    },
  }))

export interface Todo extends Instance<typeof TodoModel> {}
export interface TodoSnapshot extends SnapshotOut<typeof TodoModel> {}
```

**Creating a Store:**
```typescript
// app/models/TodoStore.ts
import { Instance, SnapshotOut, types } from "mobx-state-tree"
import { TodoModel } from "./Todo"

export const TodoStoreModel = types
  .model("TodoStore")
  .props({
    todos: types.array(TodoModel),
  })
  .actions((self) => ({
    addTodo(text: string) {
      self.todos.push({
        id: Date.now().toString(),
        text,
        done: false,
      })
    },
    removeTodo(id: string) {
      const todo = self.todos.find(t => t.id === id)
      if (todo) self.todos.remove(todo)
    },
  }))
  .views((self) => ({
    get completedCount() {
      return self.todos.filter(t => t.done).length
    },
  }))

export interface TodoStore extends Instance<typeof TodoStoreModel> {}
export interface TodoStoreSnapshot extends SnapshotOut<typeof TodoStoreModel> {}
```

**Root Store Setup:**
```typescript
// app/models/RootStore.ts
import { Instance, SnapshotOut, types } from "mobx-state-tree"
import { TodoStoreModel } from "./TodoStore"

export const RootStoreModel = types.model("RootStore").props({
  todoStore: types.optional(TodoStoreModel, {}),
})

export interface RootStore extends Instance<typeof RootStoreModel> {}
export interface RootStoreSnapshot extends SnapshotOut<typeof RootStoreModel> {}

// Create store instance
export const rootStore = RootStoreModel.create({})
```

**Using in Components:**
```typescript
import { observer } from "mobx-react-lite"
import { rootStore } from "../models/RootStore"

export const TodoList = observer(function TodoList() {
  const { todoStore } = rootStore

  return (
    <Screen>
      <Text preset="heading">
        {todoStore.completedCount} of {todoStore.todos.length} completed
      </Text>
      {todoStore.todos.map((todo) => (
        <ListItem
          key={todo.id}
          text={todo.text}
          onPress={() => todo.toggle()}
        />
      ))}
    </Screen>
  )
})
```

**DO NOT:**
- ❌ Use Redux or Redux Toolkit
- ❌ Use React Context for complex state
- ❌ Use useState for app-wide state
- ❌ Use useReducer for complex state

**DO:**
- ✅ Use MobX-State-Tree models
- ✅ Wrap components with `observer()`
- ✅ Use `.views()` for computed values
- ✅ Use `.actions()` for mutations

### 4. Styling and Theming

**Use the Theme System:**

```typescript
import { useAppTheme } from "../theme"
import { ViewStyle, TextStyle } from "react-native"

function MyComponent() {
  const { themed, theme } = useAppTheme()

  return (
    <View style={themed($container)}>
      <Text style={{ color: theme.colors.text }}>
        Hello
      </Text>
    </View>
  )
}

const $container: ThemedStyle<ViewStyle> = (theme) => ({
  backgroundColor: theme.colors.background,
  padding: theme.spacing.md,
  borderRadius: 8,
})
```

**Theme Values:**

```typescript
// Colors
theme.colors.background
theme.colors.text
theme.colors.tint
theme.colors.error
theme.colors.palette.primary100  // through primary900
theme.colors.palette.neutral100  // through neutral900

// Spacing (based on 4px grid)
theme.spacing.xxxs  // 2
theme.spacing.xxs   // 4
theme.spacing.xs    // 8
theme.spacing.sm    // 12
theme.spacing.md    // 16
theme.spacing.lg    // 24
theme.spacing.xl    // 32
theme.spacing.xxl   // 48
theme.spacing.xxxl  // 64

// Typography
import { typography } from "../theme/typography"
theme.typography.primary  // Font family
theme.typography.secondary
theme.typography.code
```

**Styling Rules:**
- ✅ Use theme system for colors, spacing, fonts
- ✅ Use StyleSheet.create() or inline styles
- ✅ Use TypeScript types (ViewStyle, TextStyle, ImageStyle)
- ❌ NO CSS files (.css, .scss)
- ❌ NO Tailwind CSS or other CSS-in-JS libraries
- ❌ NO HTML styling patterns

### 5. Navigation

**Navigation is pre-configured in `app/navigators/AppNavigator.tsx`.**

**Adding a Screen:**
1. Create screen component in `app/screens/`
2. Add to navigator:

```typescript
// app/navigators/AppNavigator.tsx
export type AppStackParamList = {
  Welcome: undefined
  Demo: undefined
  MyNewScreen: { userId: string }  // with params
}

// In the stack component
<Stack.Navigator>
  <Stack.Screen name="Welcome" component={WelcomeScreen} />
  <Stack.Screen name="MyNewScreen" component={MyNewScreen} />
</Stack.Navigator>
```

**Navigation in Components:**
```typescript
import { AppStackScreenProps } from "../navigators"

interface MyScreenProps extends AppStackScreenProps<"MyScreen"> {}

export const MyScreen: FC<MyScreenProps> = ({ navigation }) => {
  // Navigate to screen
  navigation.navigate("OtherScreen")

  // With params
  navigation.navigate("Profile", { userId: "123" })

  // Go back
  navigation.goBack()

  // Replace
  navigation.replace("Login")
}
```

### 6. API Integration

**Use the apisauce API client:**

```typescript
// app/services/api/api.ts (pre-configured)
import { ApiResponse } from "apisauce"
import { Api } from "./api"

const api = new Api()
api.setup()

// Make requests
const response: ApiResponse<User> = await api.apisauce.get("/users/1")

if (response.ok && response.data) {
  // Success
  const user = response.data
} else {
  // Error
  console.error(response.problem)
}
```

**Store API calls in actions:**
```typescript
// In MobX-State-Tree store
.actions((self) => ({
  loadUsers: flow(function* () {
    const response = yield api.apisauce.get("/users")
    if (response.ok) {
      self.users.replace(response.data)
    }
  }),
}))
```

### 7. Internationalization (i18n)

**Using translations:**

```typescript
import { translate, TxKeyPath } from "../i18n"

// In components
<Text tx="welcomeScreen.title" />
<Text tx="common.continue" />

// With parameters
<Text tx="welcomeScreen.greeting" txOptions={{ name: "John" }} />

// In code
const title = translate("welcomeScreen.title")
```

**Adding translations:**
```typescript
// app/i18n/en.ts
export const en = {
  common: {
    ok: "OK",
    cancel: "Cancel",
    continue: "Continue",
  },
  welcomeScreen: {
    title: "Welcome!",
    greeting: "Hello, {{name}}!",
  },
}
```

### 8. Common Patterns

**Loading States:**
```typescript
const [isLoading, setIsLoading] = useState(false)

// In UI
{isLoading ? (
  <ActivityIndicator />
) : (
  <Button text="Load Data" onPress={loadData} />
)}
```

**Empty States:**
```typescript
import { EmptyState } from "../components"

{items.length === 0 ? (
  <EmptyState
    preset="generic"
    heading="No items yet"
    content="Add your first item to get started"
    button="Add Item"
    buttonOnPress={handleAddItem}
  />
) : (
  // List of items
)}
```

**Forms:**
```typescript
const [email, setEmail] = useState("")
const [password, setPassword] = useState("")

<TextField
  label="Email"
  value={email}
  onChangeText={setEmail}
  autoCapitalize="none"
  keyboardType="email-address"
/>
<TextField
  label="Password"
  value={password}
  onChangeText={setPassword}
  secureTextEntry
/>
<Button
  text="Sign In"
  onPress={handleSignIn}
  disabled={!email || !password}
/>
```

### 9. Testing

**Component Tests:**
```typescript
import { render } from "@testing-library/react-native"
import { Text } from "./Text"

describe("Text", () => {
  it("renders correctly", () => {
    const { getByText } = render(<Text>Hello</Text>)
    expect(getByText("Hello")).toBeTruthy()
  })
})
```

### 10. What NOT to Do

**❌ NEVER:**
- Use HTML elements (`<div>`, `<p>`, `<span>`, `<img>`, `<a>`)
- Use CSS files or styled-components
- Use Tailwind CSS classes
- Use Redux or Redux Toolkit
- Use raw React Native components when Ignite components exist
- Modify `app.tsx` or core navigation unless absolutely necessary
- Use class components (always use functional components with hooks)
- Skip the `observer()` wrapper when using MobX-State-Tree
- Put business logic directly in screens (use stores/models)

**✅ ALWAYS:**
- Use React Native components (`<View>`, `<Text>`, `<ScrollView>`, `<FlatList>`)
- Use Ignite's component library
- Use the theme system for styling
- Use MobX-State-Tree for state management
- Wrap components with `observer()` when using observables
- Use the `<Screen>` component as the root of screens
- Check types with TypeScript
- Use i18n for all user-facing text
- Follow the established folder structure

## Quick Start Checklist for AI

When starting to build in an Ignite app:

1. ✅ Check what screens exist in `app/screens/`
2. ✅ Review available components in `app/components/`
3. ✅ Understand the navigation structure in `app/navigators/`
4. ✅ Check if state management (MobX-State-Tree) is set up in `app/models/`
5. ✅ Review the theme in `app/theme/`
6. ✅ Build UI first using existing components
7. ✅ Add state/logic with MobX-State-Tree models
8. ✅ Use i18n for all text
9. ✅ Test on the preview URL (not localhost)
10. ✅ Commit incrementally as you build

## Example: Building a Todo App

```typescript
// 1. Create model (app/models/Todo.ts)
export const TodoModel = types.model({
  id: types.identifier,
  text: types.string,
  done: types.boolean,
}).actions(self => ({
  toggle() { self.done = !self.done }
}))

// 2. Create store (app/models/TodoStore.ts)
export const TodoStoreModel = types.model({
  todos: types.array(TodoModel),
}).actions(self => ({
  addTodo(text: string) {
    self.todos.push({ id: Date.now().toString(), text, done: false })
  },
}))

// 3. Create screen (app/screens/TodoScreen.tsx)
export const TodoScreen = observer(function TodoScreen() {
  const [text, setText] = useState("")
  const { todoStore } = rootStore

  return (
    <Screen preset="scroll">
      <Header title="Todos" />

      <TextField
        value={text}
        onChangeText={setText}
        placeholder="New todo"
      />
      <Button
        text="Add"
        onPress={() => {
          todoStore.addTodo(text)
          setText("")
        }}
      />

      {todoStore.todos.map(todo => (
        <ListItem
          key={todo.id}
          text={todo.text}
          rightIcon={todo.done ? "check" : "x"}
          onPress={() => todo.toggle()}
        />
      ))}
    </Screen>
  )
})

// 4. Register in navigator
<Stack.Screen name="Todo" component={TodoScreen} />
```

This pattern demonstrates:
- ✅ MobX-State-Tree for state
- ✅ Ignite components (Screen, Header, TextField, Button, ListItem)
- ✅ Observer pattern
- ✅ TypeScript
- ✅ React Native patterns

Now you're ready to build amazing Ignite apps! 🔥
