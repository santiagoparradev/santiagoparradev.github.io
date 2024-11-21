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
export const omniscriptBaseMixinMock = jest.fn();

function mockProperty(propName) {
  // in case your api variable conflict with the mock implementation
  if (!(propName in this) || !(`_${propName}` in this)) {
    Object.defineProperty(this, propName, {
      get: jest.fn(() => this[`_${propName}`]),
      set: jest.fn((value) => {
        this[`_${propName}`] = value;
      }),
    });
  }
}

function mockFunction (functionName) {
  // in case your api methods conflict with the mock implementation
  if (!(functionName in this)) {
    this[functionName] = jest.fn();
  }
}

export const OmniscriptBaseMixin = (Base) => {
  return class extends Base {
    constructor() {
      super();
      mockFunction.call(this, 'checkValidity');
      mockFunction.call(this, 'omniApplyCallResp');
      mockFunction.call(this, 'omniGetMergeField');
      mockFunction.call(this, 'omniGetSaveState');
      mockFunction.call(this, 'omniNavigateTo');
      mockFunction.call(this, 'omniNextStep');
      mockFunction.call(this, 'omniPrevStep');
      mockFunction.call(this, 'omniRemoteCall');
      mockFunction.call(this, 'omniSaveForLater');
      mockFunction.call(this, 'omniSaveState');
      mockFunction.call(this, 'omniUpdateDataJson');
      mockFunction.call(this, 'omniValidate');
      mockFunction.call(this, 'reportValidity');
      mockProperty.call(this, 'dataLayout');
      mockProperty.call(this, 'omniCustomState');
      mockProperty.call(this, 'omniJsonData');
      mockProperty.call(this, 'omniJsonDataStr');
      mockProperty.call(this, 'omniJsonDef');
      mockProperty.call(this, 'omniResume');
      mockProperty.call(this, 'omniScriptHeaderDef');
      mockProperty.call(this, 'omniSeedJson');
      mockProperty.call(this, 'showValidation');
      omniscriptBaseMixinMock(this);
    }
  };
};

export const getOmniscriptBaseMixinMock = () => omniscriptBaseMixinMock.mock.lastCall[0];
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
import { getOmniscriptBaseMixinMock } from "omnistudio/omniscriptBaseMixin";
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
    const omni = getOmniscriptBaseMixinMock();
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
const omni = getOmniscriptBaseMixinMock();

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
👉 Ensure that you mock your child components properly, as you may encounter multiple instances of `OmniscriptBaseMixin`. The `getOmniscriptBaseMixinMock` function returns only the last instance(*this is so you dont have to reset anything after each test*).If you need to handle multiple instances, you can take the following approach:

```javascript
import { omniscriptBaseMixinMock } from "omnistudio/omniscriptBaseMixin";
```

and then:

```javascript
const omni = omniscriptBaseMixinMock.mock.calls[0][0];
```

---

### 📚 **References**  

[OmniStudio Winter '22 Documentation](https://help.salesforce.com/s/articleView?id=sf.os_omniscript_readme_reference_24281.htm&type=5)  

[Extend OmniscriptBaseMixing Component](https://help.salesforce.com/s/articleView?id=sf.os_extend_the_omniscriptbasemixin_component_17717.htm&type=5)

---

This guide ensures you can confidently test your OmniStudio-integrated LWCs while keeping your tests clean and effective! 😊
