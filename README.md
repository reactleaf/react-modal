# @reactleaf/react-modal

React modal with context and hooks

## Concept

This library uses HoC to provide context and container, and provide hooks to open and close modal.
Main Concept of this library is providing **type-safe** method to open specific modals on anywhere of your code.

More details are on below

## Installation and Usage

```sh
npm install @reactleaf/react-modal
# or
yarn add @reactleaf/react-modal
```

### Modal Register

At first, you should make a your own modal register.
we are using dynamic import, to reduce initial loading size.
Any modals registered will be loaded on when modal is called to open, not on initialize sequence.

```typescript
const register = {
  Alert: () => import("./Alert"),
  Confirm: () => import("./Confirm"),
};

export default register;
```

### Modal Context

Now provide this register to your app.
This HoC will provide modalContext to your app, and also modal container, that modals will be rendered.
So Simple!

```typescript
import { withModal } from "@reactleaf/react-modal";
import register from "./modals/register";

function App() {
  ...
}

export default withModal(register, App);
```

### useModal Hook

To open modal, you should use useModal hook.
for **type-safe** of your code, do not use useModal directly.

#### Don't

If you are non-typescript user, it's okay to use this way.

```typescript
import { useModal } from "@reactleaf/react-modal";

const { openModal } = useModal();
function openAlert() {
  // openModal cannot check types.
  openModal({ type: "Alert", props: { title: "Hello" } });
}
```

#### Recommended Way

With this way, you can check type and props are properly provided.

```typescript
// useModal.ts
import { createModalHook } from "@reactleaf/react-modal";
import register from "./register";

export const useModal = createModalHook<typeof register>();
```

`openModal` will check modal type

```typescript
import { useModal } from './modals/useModal'

const { openModal } = useModal()
function openAlert() {
  openModal({ type: 'Confrim', props: { title: 'Hello', message: 'Wow' } })
              ^^^^
              type 'Confrim' is not assignable to type 'Alert' | 'Confirm'
}
```

`openModal` will check props type for the matching type

```typescript
import { useModal } from './modals/useModal'

const { openModal } = useModal()
function openAlert() {
  openModal({ type: 'Alert', props: { title: 'Hello' } })
                             ^^^^^
                             property 'message' is missing
}
```

## Props

#### withModal(register, App)

- `register` - your modal register
- `App` - your App
- `returns` - Higher ordered App

#### createModalHook<Register>()

```typescript
const useModal = createModalHook<typeof yourModalRegister>();
```

#### useModal()

- `retuns` - { openModal, closeAll }

#### openModal(payload)

open selected typed modal with given props

```typescript
openModal({ type: keyof Register, props: Props, overlayOptions: OverlayOptions })
```

- `props` - Matching Props as type. if type === "Alert", props should be `React.ComponentProps<Alert>`
- `overlayOptions`

```typescript
export interface OverlayOptions {
  dim?: boolean; // default: true
  transitionDuration?: number; // default: 0, as ms. this will make modal close(unmount) delayed.
  closeOnOverlayClick?: boolean; // default: true
  preventScroll?: boolean; // default: true, when modal is opened, body scroll is blocked.
}
```

#### closeAll()

close all opened modals

## How to close opened modal?

Modal can only closed by modal itself. see more on [below](#BasicModalProps)

But there are 2 exceptions.

- `closeAll()` - If you close All Modals, then every opened modals are closed.
- `closeOnOverlayClick: true` - if user click outside of modal (may be darken with dim color), top modal is closed.

### BasicModalProps

When modal is opened by `openModal`, 2 more props are injected to your modal.

- `close(): void`
- `visible: boolean`

So, When implementing modal, you can consider `close` props like this.

```tsx
import { BasicModalProps } from "@reactleaf/react-modal";

interface Props extends BasicModalProps {
  title: string;
  message: string;
}
const Alert = ({
  title,
  message,
  close, // injected by react-modal
}: Props) => {
  return (
    <div className="alert modal">
      <p className="modal-title">{title}</p>
      <div className="modal-body">
        <p className="message">{message}</p>
      </div>
      <div className="modal-buttons">
        <button onClick={close}>Close</button>
      </div>
    </div>
  );
};
```

## Working Examples

See more on [Examples](https://github.com/reactleaf/react-modal/tree/main/examples)
