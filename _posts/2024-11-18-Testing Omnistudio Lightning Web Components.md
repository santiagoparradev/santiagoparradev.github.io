---
title: Testing Lightning Web Components in OmniStudio
date: 2024-11-18 12:00:00 +0000
layout: post
categories: [lwc]
tags: [omnistudio, jest, lwc]
description: "A short guide on how to test Lightning Web Components (LWC) that are integrated into OmniStudio, covering tools and methods for effective testing."
---

## 🚀 **Test OmniStudio Lightning Web Components**  

Imagine you're testing a Lightning Web Component (LWC) that extends `OmniscriptBaseMixin` and relies on methods like `omniRemoteCall`, `omniJsonData`, or `omniApplyCallResp`. Here's how you can easily mock and test these components!

### **`OmnistudioLightningWebComponent`**  

```javascript
import { LightningElement } from 'lwc';
import { OmniscriptBaseMixin } from "omnistudio/omniscriptBaseMixin";

export default class OmnistudioLightningWebComponent extends OmniscriptBaseMixin(LightningElement) {
  async connectedCallback() {
    console.log(this.omniJsonData); // { mocked: 'prop'}
    console.log(await this.omniRemoteCall()) // { mocked: 'promise'}
    this.omniApplyCallResp({ updated: true });
  }
}
```

---

## 🛠️ **Step-by-Step Guide**  

### 📂 **Folder Structure**  

```plaintext
.
├── 📁 force-app
│   ├── 📁 main
│   │   └── 📁 default
│   │       └── 📁 lwc
│   │           └── 📁 omnistudioLightningWebComponent
│   │               ├── 📁 __tests__
│   │               │   └── 📄 omnistudioLightningWebComponent.test.js
│   │               └── 📄 omnistudioLightningWebComponent.js
│   └── 📁 test
│       └── 📁 jest-mocks
│           └── 📁 omnistudio
│               └── 📄 omniscriptBaseMixin.js
└── 📄 jest.config.js
```

---

### 1️⃣ **Create `omniscriptBaseMixin.js`**  

🎯 Create the mock implementation for  `OmniscriptBaseMixin` for your tests.  

```javascript
const OmniscriptBaseMixin = jest.fn();

OmniscriptBaseMixin.mockImplementation((Base) => {
  OmniscriptBaseMixin.instances = [];
  
  return class extends Base {

    mockProperty(propName) {
      Object.defineProperty(this, propName, {
        get: jest.fn(() => this[`_${propName}`]),
        set: jest.fn((value) => {
          this[`_${propName}`] = value;
        }),
      });
    }

    constructor() {
      super();

      // properties
      this.mockProperty('omniJsonData');
      this.mockProperty('omniJsonDef');
      this.mockProperty('omniSeedJson');
      this.mockProperty('omniResume');
      this.mockProperty('omniScriptHeaderDef');
      this.mockProperty('omniJsonDataStr');
      this.mockProperty('omniCustomState');
      this.mockProperty('dataLayout');
      this.mockProperty('showValidation');

      // methods
      this.checkValidity = jest.fn();
      this.omniApplyCallResp = jest.fn();
      this.omniGetMergeField = jest.fn();
      this.omniGetSaveState = jest.fn();
      this.omniNavigateTo = jest.fn();
      this.omniNextStep = jest.fn();
      this.omniPrevStep = jest.fn();
      this.omniRemoteCall = jest.fn();
      this.omniSaveForLater = jest.fn();
      this.omniSaveState = jest.fn();
      this.omniUpdateDataJson = jest.fn();
      this.omniValidate = jest.fn();
      this.reportValidity = jest.fn();
      OmniscriptBaseMixin.instances.push(this);
    }
  };
});

export { OmniscriptBaseMixin };
```

---

### 2️⃣ **Configure `jest.config.js`**  

🛠️ Map the mock file in your Jest configuration.  

```javascript
const { jestConfig } = require('@salesforce/sfdx-lwc-jest/config');

module.exports = {
    ...jestConfig,
    modulePathIgnorePatterns: ['<rootDir>/.localdevserver'],
    testMatch: ['**/__tests__/**/*.test.js'],
    moduleNameMapper: {
        '^omnistudio/omniscriptBaseMixin$': '<rootDir>/force-app/test/jest-mocks/omniscriptBaseMixin.js',
    }
};
```
👉 if you are using velocity package

```javascript
'^vlocity_ins/omniscriptBaseMixin$': '<rootDir>/force-app/test/jest-mocks/omniscriptBaseMixin.js',
```
---

### 3️⃣ **Write the Jest Test**  

🧪 Test your component `omnistudioLightningWebComponent.test.js`.  

```javascript
import { createElement } from "lwc";
import { OmniscriptBaseMixin } from "omnistudio/omniscriptBaseMixin";
import OmnistudioLightningWebComponent from "c/omnistudioLightningWebComponent";

describe("c-omnistudio-lightning-web-component", () => {
  afterEach(() => {
    while (document.body.firstChild) {
      document.body.removeChild(document.body.firstChild);
    }

    jest.clearAllMocks();
  });

  it("test desktop", async () => {
    // ARRANGE
    const element = createElement("c-facility-details-l-w-c", {
      is: OmnistudioLightningWebComponent
    });
    const omni = OmniscriptBaseMixin.instances[0];
    omni.omniJsonData = { mocked: 'prop'};
    omni.omniRemoteCall.mockResolvedValueOnce({ mocked: 'promise'});
    document.body.appendChild(element);

    await Promise.resolve();

    expect(omni.omniApplyCallResp).toHaveBeenCalledWith({ updated: true });
  });
  
});
```

---

### 📝 **More**  
👉 if you want to have more control over the **properties** you can use
  - `Object.getOwnPropertyDescriptor(object, 'property').get` or
  - `Object.getOwnPropertyDescriptor(object, 'property').set`


This can be usefull for properties such as `showValidation` that can change without user/component interaction

```javascript
const omni = OmniscriptBaseMixin.instances[0];

// for example mocking multiple usages of the property
Object.getOwnPropertyDescriptor(omni, 'omniJsonData').get
  .mockReturnValueOnce({ mocked: 'first'})
  .mockReturnValueOnce({ mocked: 'second'});
console.log(omni.omniJsonData); // { mocked: 'first'}
console.log(omni.omniJsonData); // { mocked: 'second'}

// or making sure the propery value changed
omni.omniJsonData = 'valueChanged';
expect(Object.getOwnPropertyDescriptor(omni, 'omniJsonData').set)
  .toHaveBeenCalledWith('valueChanged');
```

---

### 📚 **References**  

[OmniStudio Winter '22 Documentation](https://help.salesforce.com/s/articleView?id=sf.os_omniscript_readme_reference_24281.htm&type=5)  

---

This guide ensures you can confidently test your OmniStudio-integrated LWCs while keeping your tests clean and effective! 😊
