---
title: Testing setTimeout in Lightning Web Components (JEST)
date: 2024-11-29 12:00:00 +0000
layout: post
categories: [lwc, jest]
tags: [jest, lwc, setTimeout]
description: "Testing setTimeout functions in JEST for an lwc"
---


### **Managing Timers in Tests**

#### Single `setTimeout`

Use `jest.runOnlyPendingTimers` when there’s a single `setTimeout`.

```javascript
const flushTimersAndPromises = async () => {
  await Promise.resolve(); // Optional: Wait for DOM updates
  jest.runOnlyPendingTimers();
};
```

#### Nested `setTimeout` (No DOM Updates Needed)

Use `jest.runAllTimers` for nested `setTimeout` calls when DOM updates between them are irrelevant.

```javascript
const flushTimersAndPromises = async () => {
  await Promise.resolve(); // Optional: Wait for DOM updates
  jest.runAllTimers();
};
```

#### Nested `setTimeout` (With DOM Updates Between)

For nested `setTimeout` calls where DOM updates matter, use recursion to handle timers and promises:

```javascript
const flushTimersAndPromises = async () => {
  await Promise.resolve(); // Wait for DOM updates
  if (jest.getTimerCount() > 0) {
    jest.runOnlyPendingTimers();
    await flushTimersAndPromises(); // Resolve remaining timers
  }
};
```

---

### **Component Overview**

This component interacts with the user:

1. **Asks a question**: *"How was your day?"*
2. **Conditional prompt**: If the user types "bad," a second input box appears: *"Tell me more about it."*
3. **User experience**: Ensures smooth interaction by delaying reactions until typing pauses.

---

### **Component Code**

#### `setTimeoutLightningWebComponent.html`

```html
<template>
  <label for="userInput">How was your day?</label>
  <input type="text" data-id="question-a" onchange={handleChange}/>
  <template lwc:if={badDay}>
    <label for="userInput">Tell me more about it</label>
    <input type="text" data-id="question-b"/>
  </template>
</template> 
```

#### `setTimeoutLightningWebComponent.js`

```javascript
/* eslint-disable @lwc/lwc/no-async-operation */
import { LightningElement } from 'lwc';

export default class SetTimeoutLightningWebComponent extends LightningElement {
  badDay = false; // 🚩 Tracks if the user's day is "bad" and controls showing the second input box.

  handleChange(event) {
    // ⏳ Wait for the user to stop typing (debouncing rapid input)
    clearTimeout(this.timeoutId); // 🧹 Clear any previous timer
    this.timeoutId = setTimeout(() => {
      this.focusInput(event.detail.value); // 🔍 Process the user's input after 300ms of no typing
    }, 300);
  }

  focusInput(value) {
    if (value === 'bad') {
      // 😞 If the user says "bad," mark `badDay` as true and show the second input box
      this.badDay = true;
      
      console.log(this.template.querySelector(`[data-id="question-b"]`)); // ❓ Logs undefined because the second input box isn't in the DOM yet

      // 🕒 Wait for the DOM to update before interacting with the new input box
      setTimeout(() => {
        const input = this.template.querySelector(`[data-id="question-b"]`); // 🔎 Find the second input box
        console.log(input); // ✅ Now the input exists, and we can use it!
        input.focus(); // 🚀 Automatically focus the second input box for the user
      }, 0);
    }
  }
}
```

---

### **Test Code**

#### `setTimeoutLightningWebComponent.test.js`

```javascript
import { createElement } from 'lwc';
import SetTimeoutLightningWebComponent from 'c/setTimeoutLightningWebComponent';

// 🧪 Test suite for SetTimeoutLightningWebComponent
describe("c-set-timeout-lightning-web-component", () => {

  // 🔄 Helper to ensure all timers and promises are resolved in tests
  const flushTimersAndPromises = async () => {
    await Promise.resolve();
    if (jest.getTimerCount() > 0) {
      jest.runOnlyPendingTimers();
      await flushTimersAndPromises(); // Recursively resolve remaining timers
    }
  }

  // 🎯 Simulates user input in the component
  const emulateUserInput = (element) => {
    const questionA = element.shadowRoot.querySelector(`[data-id="question-a"]`);
    // Simulate typing "bad" step by step
    questionA.dispatchEvent(new CustomEvent("change", { detail: { value: 'b' } }));
    questionA.dispatchEvent(new CustomEvent("change", { detail: { value: 'ba' } }));
    questionA.dispatchEvent(new CustomEvent("change", { detail: { value: 'bad' } }));
  }

  // 🔧 Test setup: use fake timers for better control
  beforeEach(() => {
    jest.useFakeTimers();
  });

  // 🧹 Cleanup: reset DOM and timers after each test
  afterEach(() => {
    while (document.body.firstChild) {
      document.body.removeChild(document.body.firstChild);
    }
    jest.clearAllTimers();
    jest.useRealTimers(); // Restore default timer behavior
  });

  // 📝 Test case: Verify the component reacts correctly to input and timers
  it("runAllTimers", async () => {
    // GIVEN: Component setup
    const element = createElement("c-set-timeout-example", {
      is: SetTimeoutLightningWebComponent
    });
    document.body.appendChild(element);

    // Simulate user input
    emulateUserInput(element);

    // Verify that "question-b" is not yet rendered
    expect(element.shadowRoot.querySelector(`[data-id="question-b"]`)).toBeNull();

    // Resolve timers and promises
    await flushTimersAndPromises();

    // Verify "question-b" is now rendered and focused
    expect(element.shadowRoot.querySelector(`[data-id="question-b"]`))
      .toBe(element.shadowRoot.activeElement);
  });
});

```

---

### **Key Notes**

- **Test Timer Management:**
  - Use `jest.runOnlyPendingTimers` for single `setTimeout`.
  - Use `jest.runAllTimers` for nested `setTimeout` without DOM updates.
  - Use recursion for nested `setTimeout` with DOM updates.
