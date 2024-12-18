---
title: "Understanding Performance in Salesforce: Choosing the Right Looping Strategy"
date: 2024-12-06 09:20:00 +0000
layout: post
categories: [apex]
tags: [apex]
description: "Understanding Performance in Salesforce: Choosing the Right Looping Strategy"
---

## **Understanding Performance in Salesforce: Choosing the Right Looping Strategy**

When developing in Salesforce, the method you use to filter large datasets can significantly affect your code's performance. Let’s compare three common approaches: a single for-loop with all conditions, chained for-loops, and using an evaluator class. We will explain how each approach works, why one may be more efficient than the others, and how flexibility can be a deciding factor when choosing the right method for your project.

---

### **1. Single For-Loop with All Conditions 🏃‍♂️💨**

This approach involves checking all conditions within a single loop. It’s simple and efficient, as it iterates over the records only once.

#### **Usage Example:**

```apex
List<Case> result = new List<Case>();
for (Case record : cases) {
    if (condition1(record) && condition2(record) && condition3(record)) {
        result.add(record);
    }
}
```

- **Efficiency**: 🚀 This is the most efficient approach in terms of both CPU time and heap usage.
- **Use Case**: Ideal when the conditions are fixed and performance is a top priority.

#### **Performance Outcome**:
- **CPU Time**: 🟢 Low  
- **Heap Usage**: 🟢 Low  

---

### **2. Chained For-Loops Approach 🔗**

In this method, each condition is checked in separate loops, filtering the data progressively. It’s commonly used in frameworks like **FFLIB** (Apex Enterprise Patterns), which is designed for large Salesforce applications.

#### **Usage Example:**

```apex
List<Case> result = cases;
result = filterWithCondition(result, condition1);
result = filterWithCondition(result, condition2);
result = filterWithCondition(result, condition3);
```

- **Efficiency**: ⚖️ Chained for-loops have a higher CPU cost compared to a single for-loop because they perform multiple iterations over the data.
- **Flexibility**: 🔄 This method is more modular and flexible, which is beneficial for more complex filtering scenarios.
- **FFLIB**: This pattern is widely adopted in Salesforce enterprise development because it promotes modular, maintainable code. [Explore FFLIB GitHub](https://github.com/apex-enterprise-patterns/fflib-apex-common).

#### **Performance Outcome**:
- **CPU Time**: 🟡 Moderate  
- **Heap Usage**: 🟡 Moderate  

---

### **3. Using an Evaluator Class (Callback Approach) 🎯**

This approach uses an evaluator class with callback functions to evaluate complex conditions dynamically. While it provides flexibility, it is generally slower due to the overhead of the callback mechanism.

#### **Usage Example:**

```apex
Evaluator evaluator = new Evaluator()
    .add(condition1)
    .add(condition2)
    .add(condition3);

List<Case> result = new List<Case>();
for (Case record : cases) {
    if (evaluator.evaluate(record)) {
        result.add(record);
    }
}
```

- **Flexibility**: 🔧 The evaluator pattern offers great flexibility by allowing dynamic condition combinations. Specifically, methods like `.anyMatch()` and `.checkExpression()` provide ways to evaluate complex logical conditions:

    - **`.anyMatch()`**: This method checks if *any* condition in a list matches the current record, effectively translating to an OR operation. For example:
    
    ```apex
    Evaluator evaluator = new Evaluator()
        .anyMatch()
        .add(condition1)
        .add(condition2);
    ```
    This translates to a logical expression: `condition1 OR condition2`.
    
    - **`.checkExpression()`**: This method enables the evaluation of more complex logical expressions involving AND/OR operators. For instance:
    
    ```apex
    Evaluator evaluator = new Evaluator()
        .checkExpression('NOT 1 AND NOT 2')
        .add(condition1)
        .add(condition2);
    ```
    This translates to `NOT condition1 AND NOT condition2`.

- **Performance**: ⚠️ While flexible, the evaluator pattern results in higher CPU time and heap usage due to the added complexity and dynamic nature of its evaluation.

#### **Performance Outcome**:
- **CPU Time**: 🔴 High  
- **Heap Usage**: 🔴 High  

---

### **Key Takeaways 🎓**
1. **Single For-Loop with All Conditions 🏃‍♂️** is the most efficient for simple, static conditions where performance is critical.
2. **Chained For-Loops 🔗** provide a good balance between modularity and performance and are often used in enterprise frameworks like **FFLIB**.
3. **Evaluator Class 🎯** offers great flexibility for complex conditions but sacrifices performance due to the added complexity and dynamic nature of its evaluation.

Here’s the revised table with the test scenario information added:

### **Test Scenario:**
- **Data**: 55,555 Salesforce records
- **Conditions**: 14 different conditions to filter the records

| **Approach**                            | **CPU Time** | **Heap Usage** | **Key Observations**                                             |
| --------------------------------------- | ------------ | -------------- | ---------------------------------------------------------------- |
| **Single For-Loop with All Conditions** | 1110 ms      | 14,993 bytes   | Most efficient in terms of both CPU time and heap usage.         |
| **Chained For-Loop with Callbacks**     | 2374 ms      | -207,235 bytes | Moderate CPU time, significant increase in heap usage.           |
| **Chained For-Loop with Conditions**    | 6720 ms      | 15,570 bytes   | Highest CPU time, heap usage relatively similar to chained loop. |

---

### **Further Reading 📚**:
- [FFLIB Apex Common GitHub](https://github.com/apex-enterprise-patterns/fflib-apex-common)  
- [Salesforce Evaluator GitHub](https://github.com/santiagoparradev/lambda-land)
