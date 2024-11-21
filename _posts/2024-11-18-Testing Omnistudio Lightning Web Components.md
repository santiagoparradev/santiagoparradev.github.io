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
├── 📁 test
│   └── 📁 utils
│       └── 📄 omnistudio.js
├── 📁 force-app
│   └── 📁 main
│       └── 📁 default
│           └── 📁 lwc
│               └── 📁 omnistudioLightningWebComponent
│                   ├── 📁 __tests__
│                   │   └── 📄 omnistudioLightningWebComponent.test.js
│                   └── 📄 omnistudioLightningWebComponent.js
└── 📄 jest.config.js
```

---

### 1️⃣ **Create `omnistudio.js`**  

🎯 Create the mock implementation for  `OmniscriptBaseMixin` for your tests.  

```javascript
function mockOmniscriptBaseMixing(omniscriptBaseMixingModuleName) {
  const { OmniscriptBaseMixin } = require(omniscriptBaseMixingModuleName);

  jest.mock(
    omniscriptBaseMixingModuleName,
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
        this.checkValidity = jest.fn();
        this.dataLayout = jest.fn();
        this.omniApplyCallResp = jest.fn();
        this.omniCustomState = jest.fn();
        this.omniGetMergeField = jest.fn();
        this.omniGetSaveState = jest.fn();
        this.omniJsonData = jest.fn();
        this.omniJsonDataStr = jest.fn();
        this.omniJsonDef = jest.fn();
        this.omniNavigateTo = jest.fn();
        this.omniNextStep = jest.fn();
        this.omniPrevStep = jest.fn();
        this.omniRemoteCall = jest.fn();
        this.omniResume = jest.fn();
        this.omniSaveForLater = jest.fn();
        this.omniSaveState = jest.fn();
        this.omniScriptHeaderDef = jest.fn();
        this.omniSeedJson = jest.fn();
        this.omniUpdateDataJson = jest.fn();
        this.omniValidate = jest.fn();
        this.reportValidity = jest.fn();
        this.showValidation = jest.fn();
        OmniscriptBaseMixin.mock.instances.push(this);
      }
    };
  });

  return OmniscriptBaseMixin;
}

export { mockOmniscriptBaseMixing };
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
        '^utils/omnistudio$': '<rootDir>/test/utils/omnistudio.js',
    }
};
```

---

### 3️⃣ **Write the Jest Test**  

🧪 Test your component in `omnistudioLightningWebComponent.test.js`.  

```javascript
import { createElement } from "lwc";
import { mockOmniscriptBaseMixing } from "utils/omnistudio";

const OmniscriptBaseMixinMock = mockOmniscriptBaseMixing("omnistudio/omniscriptBaseMixin");
const OmnistudioLightningWebComponent = require("c/omnistudioLightningWebComponent").default;

describe("c-omnistudio-lightning-web-component", () => {
  afterEach(() => {
    while (document.body.firstChild) {
      document.body.removeChild(document.body.firstChild);
    }
    jest.clearAllMocks();
  });

  it("tests the LWC behavior", async () => {
    // 🎯 ARRANGE
    const element = createElement("c-omnistudio-lightning-web-component", {
      is: OmnistudioLightningWebComponent
    });
    const omni = OmniscriptBaseMixinMock.mocks.instance[0];
    omni.omniJsonData.mockReturnValue({ mocked: 'prop' });
    omni.omniRemoteCall.mockResolvedValueOnce({ mocked: 'promise' });

    document.body.appendChild(element);
    await Promise.resolve();

    // ✅ ASSERT
    expect(omni.omniApplyCallResp).toHaveBeenCalledWith({ updated: true });
  });
});
```

---

### 📝 **Notes**  
👉 Always use `require` instead of `import` for your LWC to ensure `OmniscriptBaseMixin` is mocked **before** your `lwc` component is defined.  

---

### 📚 **References**  

[OmniStudio Winter '22 Documentation](https://help.salesforce.com/s/articleView?id=sf.os_omniscript_readme_reference_24281.htm&type=5)  

---

This guide ensures you can confidently test your OmniStudio-integrated LWCs while keeping your tests clean and effective! 😊
