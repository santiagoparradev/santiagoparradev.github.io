---
title: Display nested card border using lightning-card component
date: 2024-11-25 22:15:00 +0000
layout: post
categories: [lwc]
tags: [lwc, lightning-card]
description: "lightning-card not displaying borders in nested card"
---

# 🛠 Fixing the Missing Border Bug in Salesforce Lightning Cards

As of today, Salesforce has not resolved a **bug** with the `lightning-card` component where:  
1. The **inner card's border** fails to display.  
2. This issue can also affect the **parent card**, especially when nested inside another card.

---

## 🚨 The Problem  
When nesting cards, you might encounter a scenario where the **borders disappear**. For example, in the code snippet below, the **nested card** does not display its border:

```html
<template>
  <lightning-card title="Parent Card">
    <div class="slds-p-horizontal_small">
      <lightning-card title="Nested Card">
        <p class="slds-p-horizontal_small">Something</p>
      </lightning-card>
    </div>
  </lightning-card>
</template>
```

This can happen in **Lightning App Builder pages** or any other context where cards are nested.  
If you're not aware of this issue, it might lead to unexpected UI glitches.  

---

## ✅ The Solution  
To fix this, **add the following properties** to the nested `lightning-card`:  
- `class="slds-card_boundary"`  
- `style="display: block;"`  

### Fixed Code Example:
```html
<template>
  <lightning-card title="Parent Card">
    <div class="slds-p-horizontal_small">
      <lightning-card title="Nested Card" class="slds-card_boundary" style="display:block;">
        <p class="slds-p-horizontal_small">Something</p>
      </lightning-card>
    </div>
  </lightning-card>
</template>
```

---

## 🔗 For Complex Scenarios  
If you're working on **more intricate designs** with nested cards, consider avoiding `lightning-card` altogether. Instead, use [**SLDS Cards**](https://www.lightningdesignsystem.com/components/cards/#Nested-Cards) for more flexibility and control.

---

By following these steps, you can ensure your UI remains consistent and avoids border display issues with nested cards.  

---

Let me know if you'd like further tweaks!
