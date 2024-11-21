---
title: Import CSV Button LWC
date: 2024-11-19 12:00:00 +0000
layout: post
categories: [lwc]
tags: [lightning-input, file-upload, lwc, csv]
description: "A short guide on how to import data from a csv file in lwc"
---

## 📤 **Solution**: File Upload with `<lightning-input>`  

Using Salesforce's built-in [Lightning Input](https://developer.salesforce.com/docs/component-library/bundle/lightning-input/documentation), we can create a button to upload CSV files seamlessly.  

---

### 🛠️ **Implementation**  

#### **HTML Code**  
Create a hidden file input styled as a button.  

```html
<label class="slds-button slds-button_neutral">
  <lightning-input
    type="file"
    data-id="csvButton"
    accept=".csv"
    label={labels.buttons.upload}
    onchange={handleImportFile}
    style="display: none"
  ></lightning-input>
  {labels.buttons.upload}
</label>
```

#### **JavaScript Code**  
Handle file uploads and parse CSV content.  

```javascript
import { LightningElement, track } from "lwc";

const MAX_FILE_SIZE = 2000000;

export default class CsvButtonUploadSample extends LightningElement {
  labels = {
    errors: {
      fileSize: "File Exceeded size limit",
      invalidInput: "CSV Input is Invalid",
    },
    buttons: {
      upload: "Upload CSV",
    },
  };

  @track employees = [];

  async handleImportFile(event) {
    const { files } = event.target;

    try {
      if (!files?.length) return; // 📂 No file selected

      if (files[0].size > MAX_FILE_SIZE) {
        throw new Error(this.labels.errors.fileSize); // 🚨 File too large
      }

      const csvString = await this.readCsv(files[0]); // 📖 Read file
      this.employees = this.getCsvData(csvString); // 🛠️ Parse CSV
    } catch (error) {
      console.error({ message: error.message }); // ❌ Handle errors
    }
  }

  readCsv(file) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = event => resolve(event.target.result);
      reader.onerror = error => reject(error);
      reader.readAsText(file); // 📖 Read file as text
    });
  }

  getCsvData(csvString) {
    const employees = [];
    const lines = csvString.split(/\r\n|\n/);
    const headers = lines[0].split(",");

    for (let i = 1; i < lines.length; i++) {
      const currentLine = lines[i].split(",");
      const employee = {};

      headers.forEach((header, index) => {
        employee[header] = currentLine[index];
      });

      employees.push(employee);
    }

    return employees; // ✅ Parsed data
  }
}
```

---

## 🧪 **Testing with Jest**  

Since `FileReader` can be tricky to handle in Jest, you’ll want to mock it for reliable tests.  

```javascript
describe("c-csv-button-upload-sample", () => {
  it("should load employees from csv", async () => {
    const element = createElement("c-csv-button-upload-sample", {
      is: CsvButtonUploadSample,
    });

    document.body.appendChild(element);

    const csvString = "Name,Age\nAlice,30\nBob,25";

    // Mock FileReader
    Object.defineProperty(global, "FileReader", {
      writable: true,
      value: jest.fn().mockImplementation(() => ({
        readAsText: jest.fn(function () {
          this.onload({
            target: {
              result: csvString,
            },
          });
        }),
      })),
    });

    const csvButton = element.shadowRoot.querySelector('lightning-input[data-id="csvButton"]');

    // Mock file selection
    Object.defineProperty(csvButton, "files", {
      value: [""],
      writable: false,
    });

    csvButton.dispatchEvent(new CustomEvent("change")); // Trigger file upload

    // Add assertions as needed
    expect(1).toBe(1); // 🔍 Example placeholder
  });
});
```

---

## 🔍 **How It Works**  

1. **File Selection**: A file input listens for changes (`onchange` event).  
2. **Validation**: The `handleImportFile` method checks:  
   - 📂 A file is selected.  
   - 🚨 The file size is within the allowed limit.  
3. **File Reading**: The `readCsv` method uses `FileReader` to convert the file into a text string.  
4. **Parsing Data**: The `getCsvData` function splits the CSV string into an array of employee objects.  

---

### 🌟 **Enhancements**  

To improve usability, style the input to look like a button:  

```html
<label class="slds-button slds-button_neutral">
  <lightning-input
    type="file"
    data-id="csvButton"
    accept=".csv"
    label={labels.buttons.upload}
    onchange={handleImportFile}
    style="display: none"
  ></lightning-input>
  {labels.buttons.upload}
</label>
```

---

This approach ensures a clean and user-friendly CSV upload experience. 🚀
