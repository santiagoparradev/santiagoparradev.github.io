---
title: Testing Lightning Web Components in OmniStudio
date: 2024-11-18 12:00:00 +0000
layout: post
categories: [lwc]
tags: [omnistudio, jest, lwc]
description: "A short guide on how to test Lightning Web Components (LWC) that are integrated into OmniStudio, covering tools and methods for effective testing."
---

### **The Solution**

#### *JavaScript*
```javascript
import { LightningElement } from "lwc";
import { OmniscriptBaseMixin } from "omnistudio/omniscriptBaseMixin";

export default class OmnistudioLightningWebComponent extends OmniscriptBaseMixin(LightningElement) {
  connectedCallback() {
    console.log(this.omniJsonData.Name); // Santiago
    console.log(this.omniJsonData.Properties); // ["1", "2", "3"]
    console.log(this.omniJsonData.Parent); // { Name: "Popeye", Age: 25 }
    this.omniApplyCallResp({ Name: 'Giovanna' });
  }
}
```
#### *Jest*
```javascript
import { createElement } from "lwc";
const { OmniscriptBaseMixin } = require("omnistudio/omniscriptBaseMixin");

jest.mock(
  "omnistudio/omniscriptBaseMixin",
  () => ({
    OmniscriptBaseMixin: jest.fn(),
  }),
  { virtual: true }
);

OmniscriptBaseMixin.mockImplementation(Base => {
  OmniscriptBaseMixin.mock.instances = [];

  return class extends Base {
    constructor() {
      super();
      this.omniJsonData = jest.fn();
      this.omniApplyCallResp = jest.fn();
      OmniscriptBaseMixin.mock.instances.push(this);
    }
  };
});

const OmnistudioLightningWebComponent =
  require("c/omnistudioLightningWebComponent").default;

describe("c-omnistudio-lightning-web-component", () => {
  it("should set omniJsonData and assert omniApplyCallResp function", async () => {
    const element = createElement("c-omnistudio-lightning-web-component", {
      is: OmnistudioLightningWebComponent,
    });

    const omni = OmniscriptBaseMixin.mock.instances[0];

    omni.omniJsonData = {
      Name: "Santiago",
      Properties: ["1", "2", "3"],
      Parent: { Name: "Popeye", Age: 25 },
    };

    document.body.appendChild(element);

    expect(omni.omniApplyCallResp).toHaveBeenCalledWith({ Name: 'Giovanna' });
  });
});
```

### Explanation

#### 1. Mock **`OmniscriptBaseMixin`**
We first define a mock implementation for `OmniscriptBaseMixin` to simulate the behavior of its methods like `omniJsonData` and `omniApplyCallResp`.

```javascript
const { OmniscriptBaseMixin } = require("omnistudio/omniscriptBaseMixin");

jest.mock(
  "omnistudio/omniscriptBaseMixin",
  () => ({
    OmniscriptBaseMixin: jest.fn(),
  }),
  { virtual: true }
);

OmniscriptBaseMixin.mockImplementation(Base => {
  OmniscriptBaseMixin.mock.instances = [];

  return class extends Base {
    constructor() {
      super();
      this.omniJsonData = jest.fn();
      this.omniApplyCallResp = jest.fn();
      OmniscriptBaseMixin.mock.instances.push(this);
    }
  };
})
```

#### 2. **Import the component**
Only now we can import our component. Notice we use `require` and not `import`
as `import` will be before the mocking of the `OmniscriptBaseMixin` occurs
```javascript
const OmnistudioLightningWebComponent =
  require("c/omnistudioLightningWebComponent").default;
```
#### 3. **Test the Component**:
- `OmniscriptBaseMixin.mock.instances[0]` stores the instance that was created as part
of the creation of the component
- We manually assign mock data to `omniJsonData`.
- The test checks if `omniApplyCallResp` is called with the expected argument `{Name: 'Giovanna'}`.
```javascript
describe("c-omnistudio-lightning-web-component", () => {
  it("should set omniJsonData and assert omniApplyCallResp function", async () => {
    const element = createElement("c-omnistudio-lightning-web-component", { is: OmnistudioLightningWebComponent });

    const omni = OmniscriptBaseMixin.mock.instances[0];

    omni.omniJsonData = {
      Name: 'Santiago',
      Properties: ['1', '2', '3'],
      Parent: { Name: 'Popeye', Age: 25 }
    };

    document.body.appendChild(element);
    
    expect(omni.omniApplyCallResp).toHaveBeenCalledWith({ Name: 'Giovanna' });
  });
});
```
