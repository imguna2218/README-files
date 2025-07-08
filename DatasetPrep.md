### Clarifying What I Meant vs. What You Understood

**Your Understanding**:
You understood my suggestion as:
1. Write a script (like `generate_data.py`) to create synthetic examples automatically.
2. Get examples from online sources (like forums).
3. Store user inputs and outputs as memory when people test ENG, adding them to the dataset.

**What I Meant**:
You’re mostly correct, but let me refine it:
1. **Synthetic Data**: Yes, write a script to generate tons of English instructions (like “sum 4 and 5” or “loop through numbers”) paired with their programming logic (intents, parameters, and program structure). This is the bulk of your dataset because it’s fast and controllable.
2. **Real-World Data**: Instead of finding a ready-made dataset online (which probably doesn’t exist for ENG), I meant collecting English descriptions of algorithms from places like Stack Overflow or coding tutorials. This is called **scraping**—basically, using code to grab text from websites. Since this takes time, we’ll focus on a quick way to do it.
3. **User Data**: Yes, as users interact with ENG, their inputs (e.g., “add 4 and 5”) and the system’s outputs can be saved to improve the dataset. But this happens *after* you’ve built the initial dataset and started training/testing.

**Key Clarification**:
- You’re right that there’s no ready-made dataset online for ENG because it’s a unique language. Scraping means grabbing relevant text (like algorithm descriptions) from websites, not finding a complete dataset. For example, someone on Stack Overflow might write, “To add two numbers, take input A and B, then return A + B.” You’d grab that text and turn it into a dataset entry.
- Since you want this done in a week, we’ll prioritize generating synthetic data (fast and easy) and add a small number of manual examples instead of heavy scraping. User data will come later when ENG is running.

**Real-World Example**:
Think of building a recipe book for a new type of cuisine you invented. You can:
- Write recipes yourself (synthetic data).
- Look at food blogs for inspiration and copy some ideas (scraping).
- Add recipes from people who try your cuisine later (user data).
We’ll focus on writing recipes (synthetic data) and adding a few by hand (manual) to keep it quick.

---

### What Is Scraping?
Scraping means using a program to pull text or data from websites. For ENG, it’s like going to Stack Overflow, finding posts where people describe algorithms in English (e.g., “To find the maximum, compare each number…”), and saving that text to use in your dataset. You’d use tools like `BeautifulSoup` in Python to grab this text automatically.

**Why It’s Hard for ENG in a Week**:
- Scraping takes time to set up (writing code, handling website rules, cleaning messy text).
- Websites like Stack Overflow have rules (e.g., rate limits) that slow you down.
- You’d need to process the text to fit your dataset format, which adds work.

**Solution for One Week**:
Instead of heavy scraping, we’ll:
- Generate most of the dataset (90%) using a script.
- Manually add a small number of examples (10%) by writing them yourself or copying a few from tutorials.
- Plan for scraping later if you want more real-world data.

---
Alright, let’s tackle your request to expand the dataset creation for ENG, your AI-driven programming language, to include a comprehensive set of intents that cover everything a normal programming language does—and more. You want a dataset that supports a wide range of built-in functions and programming constructs, surpassing what Java and Python offer, with 10–15 varied English phrasings per intent to train a robust model. I’ll provide a detailed, step-by-step plan to create this dataset in one week, including all necessary files, a full project structure, and PowerShell commands to execute the process in VS Code. I’ll keep it simple, avoid complex English, and use examples to make it clear. I’ll also list all intents, add new ones (like `IsEven`, `IsOdd`, and others), and ensure the dataset is structured for a powerful, flexible model without predefining every possible phrasing.

---

### Understanding Your Request

**What You Want**:
- Expand the `generate_data.py` script to include a comprehensive list of intents that cover all programming language constructs (arithmetic, relational operators, logical operations, control structures, loops, data structures, etc.) and go beyond Java/Python with advanced functions like `IsEven`, `IsOdd`, and more.
- Each intent should have 10–15 English phrasings (templates) to handle varied and advanced English inputs.
- The dataset should include 100,000 examples (90,000 synthetic, 10,000 manual) to train a model that understands diverse phrasings without needing every phrase predefined.
- Provide the full project structure (not just dataset-related files) for clarity.
- Focus on dataset creation files (`generate_data.py`, `data_augmentation.py`, `convert_manual_data.py`, `merge_data.py`) and include their complete code.
- Provide VS Code PowerShell commands to execute the dataset creation process.
- List all intents clearly so you can add or remove them if needed.
- Ensure the plan is structured, detailed, and completes in one week.

**My Approach**:
- **Intents**: I’ll include all the intents you listed (e.g., `Add`, `Print`, `If`, `ListSort`, etc.) and add new ones like `IsEven`, `IsOdd`, `GCD`, `LCM`, `Mean`, `Median`, `StandardDeviation`, and more to make ENG more advanced than Java/Python.
- **Templates**: Each intent will have 10–15 English phrasings, including simple and advanced forms (e.g., “Add 4 and 5” to “Compute the summation of 4 with 5”).
- **Dataset**: Generate 90,000 synthetic examples using templates and augmentation, and add 10,000 manual examples with advanced phrasings.
- **Model Flexibility**: Use data augmentation and a transformer model (like BERT, to be trained later) to handle new phrasings without predefining everything.
- **Files**: Update `generate_data.py`, `data_augmentation.py`, `convert_manual_data.py`, and `merge_data.py` to support the expanded intents.
- **Project Structure**: Provide the full structure, but only include code for dataset-related files.
- **PowerShell Commands**: Give exact commands to run in VS Code’s PowerShell terminal.
- **Plan**: Stick to the 7-day dataset creation plan, ensuring it’s doable and structured.

**Real-World Example**:
Think of building a massive cookbook for a new cuisine. You write a few recipe templates (e.g., “Make a {flavor} soup with {ingredient}”), generate tons of recipes by swapping words (synthetic data), and add some unique recipes by hand (manual data). The cookbook covers every dish type (soups, desserts, etc.) and has enough variety to teach a chef to cook any recipe, even if described differently. ENG’s dataset will do this for programming instructions.

---

### Full Project Structure

Below is the complete project structure for ENG, as requested, to provide context. I’ll only provide code for dataset-related files (`generate_data.py`, `data_augmentation.py`, `convert_manual_data.py`, `merge_data.py`) to focus on the dataset creation plan.

```
eng/
├── data/
│   ├── raw/                  # Manual or scraped data (CSV)
│   │   ├── manual_data.csv   # Manually written examples
│   │   └── user_data.csv     # User inputs (added later)
│   ├── processed/            # Final dataset (Parquet)
│   │   ├── synthetic_data.parquet  # Synthetic examples
│   │   ├── manual_data.parquet     # Manual examples
│   │   └── final_data.parquet      # Merged dataset
│   └── synthetic/            # Generated examples (CSV for review)
│       ├── synthetic_data_sample.csv
├── src/
│   ├── nlu/
│   │   ├── model.py          # Transformer model (for training later)
│   │   ├── tokenizer.py      # NLP processing
│   │   └── trainer.py        # Training logic
│   ├── executor/
│   │   ├── executor.rs       # Rust execution engine
│   │   └── ir.rs             # Intermediate representation
│   ├── memory/
│   │   ├── memory.py         # Vector database for user inputs
│   │   └── feedback.py       # User feedback system
│   ├── ui/
│   │   └── ui.py             # CLI or GUI for user interaction
│   └── utils/
│       ├── data_augmentation.py  # Generate varied phrasings
│       └── logging.py        # Centralized logging
├── scripts/
│   ├── generate_data.py      # Create synthetic dataset
│   ├── convert_manual_data.py # Convert manual CSV to Parquet
│   ├── merge_data.py         # Merge synthetic and manual data
│   ├── train.py              # Train model (not this week)
│   └── evaluate.py           # Test accuracy (not this week)
├── tests/
│   ├── test_nlu.py           # NLU tests
│   ├── test_executor.py      # Executor tests
│   ├── test_integration.py   # End-to-end tests
├── config/
│   ├── model_config.yaml     # Model settings
│   ├── training_config.yaml  # Training settings
├── Cargo.toml                # Rust dependencies
├── pyproject.toml            # Python dependencies
└── README.md                 # Documentation
```

**Dataset-Related Files**:
- `scripts/generate_data.py`: Generates 90,000 synthetic examples.
- `src/utils/data_augmentation.py`: Creates varied phrasings for templates.
- `scripts/convert_manual_data.py`: Converts manual CSV to Parquet.
- `scripts/merge_data.py`: Merges synthetic and manual data.
- `data/raw/manual_data.csv`: Stores manual examples.
- `data/processed/*`: Stores final Parquet files.
- `data/synthetic/synthetic_data_sample.csv`: Sample for review.

---

### List of Intents

To make ENG more advanced than Java and Python, I’ve included all the intents you listed, plus new ones to cover advanced functionality and common programming tasks. Each intent will have 10–15 English phrasings in the templates. Here’s the full list, grouped by category for clarity:

**Arithmetic Operations**:
1. `Add`: Add two numbers.
2. `Subtract`: Subtract one number from another.
3. `Multiply`: Multiply two numbers.
4. `Divide`: Divide one number by another.
5. `Modulus`: Find the remainder of division.
6. `Power`: Raise a number to a power.
7. `SquareRoot`: Calculate the square root.
8. `Factorial`: Compute factorial of a number.
9. `GCD`: Find the greatest common divisor of two numbers.
10. `LCM`: Find the least common multiple of two numbers.

**Statistical Operations**:
11. `Max`: Find the maximum of two or more numbers.
12. `Min`: Find the minimum of two or more numbers.
13. `Mean`: Calculate the average of a list of numbers.
14. `Median`: Find the median of a list of numbers.
15. `StandardDeviation`: Calculate the standard deviation of a list.

**Relational Operators**:
16. `GreaterThan`: Check if one number is greater than another.
17. `LessThan`: Check if one number is less than another.
18. `Equal`: Check if two numbers are equal.
19. `NotEqual`: Check if two numbers are not equal.
20. `GreaterOrEqual`: Check if one number is greater than or equal to another.
21. `LessOrEqual`: Check if one number is less than or equal to another.

**Logical Operations**:
22. `And`: Logical AND operation.
23. `Or`: Logical OR operation.
24. `Not`: Logical NOT operation.

**Number Checks**:
25. `IsEven`: Check if a number is even.
26. `IsOdd`: Check if a number is odd.
27. `IsPrime`: Check if a number is prime.
28. `IsPerfectSquare`: Check if a number is a perfect square.
29. `IsDivisible`: Check if one number is divisible by another.

**Control Structures**:
30. `If`: Execute a block if a condition is true.
31. `IfElse`: Execute one block if true, another if false.
32. `Ternary`: Conditional expression (e.g., x if condition else y).

**Loops**:
33. `ForLoop`: Iterate over a range or list.
34. `WhileLoop`: Iterate while a condition is true.
35. `Break`: Exit a loop.
36. `Continue`: Skip to the next iteration.

**Functions**:
37. `Return`: Return a value from a function.
38. `FunctionCall`: Call a predefined or user-defined function.

**Input/Output**:
39. `Print`: Output a value or string.
40. `Read`: Read input from the user.

**List Operations**:
41. `ListAppend`: Add an element to a list.
42. `ListRemove`: Remove an element from a list.
43. `ListLength`: Get the length of a list.
44. `ListSort`: Sort a list in ascending order.
45. `ListReverse`: Reverse a list.
46. `ListContains`: Check if a list contains an element.
47. `ListIndex`: Get the index of an element in a list.
48. `ListSlice`: Extract a slice of a list.

**Linked List Operations**:
49. `AddLinkedLists`: Add two linked lists.
50. `ReverseLinkedList`: Reverse a linked list.
51. `LinkedListLength`: Get the length of a linked list.
52. `LinkedListInsert`: Insert a node into a linked list.

**String Operations**:
53. `StringConcat`: Concatenate two strings.
54. `StringLength`: Get the length of a string.
55. `StringSubstring`: Extract a substring.
56. `StringUpper`: Convert a string to uppercase.
57. `StringLower`: Convert a string to lowercase.
58. `StringContains`: Check if a string contains a substring.

**Advanced Algorithms**:
59. `SumEvens`: Sum even numbers in a range.
60. `SumOdds`: Sum odd numbers in a range.
61. `Fibonacci`: Calculate the nth Fibonacci number.
62. `BinarySearch`: Perform binary search on a sorted list.
63. `GraphDFS`: Perform depth-first search on a graph.
64. `GraphBFS`: Perform breadth-first search on a graph.
65. `SortQuick`: Perform quicksort on a list.
66. `SortMerge`: Perform mergesort on a list.

**Error Handling**:
67. `TryCatch`: Handle errors in a block.

**Advanced Data Structures**:
68. `HashSetAdd`: Add an element to a hash set.
69. `HashSetContains`: Check if a hash set contains an element.
70. `StackPush`: Push an element onto a stack.
71. `StackPop`: Pop an element from a stack.
72. `QueueEnqueue`: Add an element to a queue.
73. `QueueDequeue`: Remove an element from a queue.

**Why These Intents?**:
- Covers all standard programming constructs (arithmetic, control structures, loops, data structures).
- Adds advanced features not always built into Java/Python (e.g., `GCD`, `LCM`, `Mean`, `Median`, `StandardDeviation`, `IsEven`, `IsOdd`, `IsPerfectSquare`).
- Includes complex algorithms (e.g., `GraphDFS`, `GraphBFS`, `SortQuick`) for robust programming.
- Each intent has 10–15 phrasings to handle varied English, ensuring the model learns diverse inputs.

**Can You Add/Remove Intents?**:
- **Add**: If you want more intents (e.g., `MatrixMultiply`, `TreeTraversal`), let me know, and I can add them to the templates.
- **Remove**: If any intent seems unnecessary (e.g., `GraphBFS` if you don’t need graph algorithms yet), you can skip it by commenting out its template in `generate_data.py`.

---

### 7-Day Dataset Creation Plan

This plan builds on the previous 7-day plan, updated to handle the expanded intents and ensure advanced English support. We’ll create 100,000 examples (90,000 synthetic, 10,000 manual) in `data/processed/final_data.parquet`.

**Dataset Structure**:
Each example is a row with:
- `input`: English instruction (e.g., “Compute summation of 4 with 5”).
- `tokenized`: Words with POS tags (e.g., [“compute:VERB”, “summation:NOUN”]).
- `intent`: Main action (e.g., “Add”).
- `parameters`: JSON string (e.g., `{"numbers": [4, 5]}`).
- `ast`: Program structure (e.g., `add(4, 5)`).
- `output`: Result (e.g., 9).

**Day 1: Set Up Environment**
- **Tasks**:
  1. Install Python libraries.
  2. Create project directories.
  3. Test `generate_data.py` with a few intents.
- **PowerShell Commands**:
  ```powershell
  # Navigate to project directory
  cd .\eng
  # Install dependencies
  pip install pandas pyarrow spacy nltk transformers
  python -m spacy download en_core_web_sm
  python -m nltk.downloader wordnet
  # Create directories
  mkdir data\raw data\processed data\synthetic src\nlu src\executor src\memory src\utils scripts tests config
  # Create empty files (optional, to set up structure)
  New-Item -ItemType File -Path src\nlu\model.py, src\nlu\tokenizer.py, src\nlu\trainer.py, src\executor\executor.rs, src\executor\ir.rs, src\memory\memory.py, src\memory\feedback.py, src\ui\ui.py, src\utils\logging.py, scripts\train.py, scripts\evaluate.py, tests\test_nlu.py, tests\test_executor.py, tests\test_integration.py, config\model_config.yaml, config\training_config.yaml, Cargo.toml, pyproject.toml, README.md
  ```
- **Time**: 2–3 hours.
- **Output**: Project structure set up, dependencies installed.

**Day 2: Generate Synthetic Data**
- **Tasks**: Run `generate_data.py` to create 90,000 synthetic examples with all 72 intents.
- **Files**:
  - `generate_data.py`: Generates examples with expanded templates.
  - `data_augmentation.py`: Enhances phrasings for variety.
- **PowerShell Command**:
  ```powershell
  python .\scripts\generate_data.py
  ```
- **Time**: 4–5 hours (coding and running).
- **Output**:
  - `data/processed/synthetic_data.parquet`: 90,000 examples.
  - `data/synthetic/synthetic_data_sample.csv`: 1,000 examples for review.

**Day 3: Enhance Data Augmentation**
- **Tasks**: Finalize `data_augmentation.py` to generate advanced English variations.
- **Files**: `data_augmentation.py`.
- **PowerShell Command**:
  ```powershell
  python .\src\utils\data_augmentation.py
  ```
- **Time**: 3–4 hours (testing and tweaking).
- **Output**: Augmented examples with varied phrasings.

**Day 4–6: Manual Data Collection**
- **Tasks**: Write 10,000 manual examples in `data/raw/manual_data.csv`, including advanced English.
  - Day 4: 3,000 examples (1,500 simple, 1,000 medium, 500 complex).
  - Day 5: 3,500 examples (1,500 simple, 1,500 medium, 500 complex).
  - Day 6: 3,500 examples (1,500 simple, 1,500 medium, 500 complex).
  - Use advanced phrasings (e.g., “Evaluate the product of numbers up to 5” for `Factorial`).
- **Files**:
  - `data/raw/manual_data.csv`: Manual examples.
  - `convert_manual_data.py`: Converts CSV to Parquet.
- **PowerShell Commands**:
  ```powershell
  # Run daily to convert manual data
  python .\scripts\convert_manual_data.py
  ```
- **Time**: 6–8 hours/day (18–24 hours total).
- **Output**: `data/processed/manual_data.parquet` with 10,000 examples.

**Day 7: Merge and Validate**
- **Tasks**: Combine synthetic and manual data, check for duplicates and errors.
- **Files**: `merge_data.py`.
- **PowerShell Command**:
  ```powershell
  python .\scripts\merge_data.py
  ```
- **Time**: 3–4 hours.
- **Output**: `data/processed/final_data.parquet` with 100,000 examples.

---

### Dataset-Related Files

Below are the complete, updated codes for the dataset creation files, incorporating all 72 intents with 10–15 templates each. I’ve kept the code efficient and focused on generating a robust dataset.

**1. scripts/generate_data.py**
```python
import pandas as pd
import json
import random
import spacy
import nltk
from nltk.corpus import wordnet
import os
from src.utils.data_augmentation import augment_text
import math

try:
    nltk.data.find('wordnet')
except LookupError:
    nltk.download('wordnet')

nlp = spacy.load("en_core_web_sm")

TEMPLATES = {
    "Add": [
        "Add {0} and {1}",
        "Sum {0} plus {1}",
        "Calculate {0} + {1}",
        "Total of {0} and {1}",
        "Compute the summation of {0} with {1}",
        "Perform addition on {0} and {1}",
        "Find the aggregate of {0} and {1}",
        "Combine {0} with {1} numerically",
        "Evaluate {0} plus {1}",
        "Determine the sum of {0} and {1}",
        "Add together {0} and {1}",
        "Compute {0} added to {1}",
        "Get the total of {0} plus {1}",
        "Sum up {0} and {1}",
        "Calculate the addition of {0} with {1}"
    ],
    "Subtract": [
        "Subtract {1} from {0}",
        "Take {1} away from {0}",
        "Calculate {0} minus {1}",
        "Find the difference between {0} and {1}",
        "Compute {0} subtracted by {1}",
        "Determine {0} less {1}",
        "Evaluate {0} - {1}",
        "Get {0} minus {1}",
        "Reduce {0} by {1}",
        "Calculate the subtraction of {1} from {0}",
        "Find {0} decreased by {1}",
        "Compute the remainder of {0} after subtracting {1}",
        "Take away {1} from {0}",
        "Determine the result of {0} minus {1}",
        "Subtract {1} out of {0}"
    ],
    "Multiply": [
        "Multiply {0} by {1}",
        "Calculate {0} times {1}",
        "Find the product of {0} and {1}",
        "Compute {0} multiplied by {1}",
        "Determine {0} * {1}",
        "Evaluate the product of {0} with {1}",
        "Get {0} times {1}",
        "Perform multiplication of {0} and {1}",
        "Calculate {0} x {1}",
        "Find the result of {0} multiplied by {1}",
        "Compute the multiplication of {0} with {1}",
        "Multiply together {0} and {1}",
        "Get the product of {0} and {1}",
        "Evaluate {0} multiplied with {1}",
        "Determine the outcome of {0} times {1}"
    ],
    "Divide": [
        "Divide {0} by {1}",
        "Calculate {0} divided by {1}",
        "Find the quotient of {0} and {1}",
        "Compute {0} over {1}",
        "Determine {0} / {1}",
        "Evaluate the division of {0} by {1}",
        "Get {0} divided by {1}",
        "Perform division of {0} by {1}",
        "Calculate the result of {0} over {1}",
        "Find {0} split by {1}",
        "Compute the quotient of {0} divided by {1}",
        "Divide {0} into {1} parts",
        "Evaluate {0} by {1}",
        "Determine the division of {0} with {1}",
        "Get the result of {0} divided by {1}"
    ],
    "Modulus": [
        "Find the remainder of {0} divided by {1}",
        "Calculate {0} mod {1}",
        "Compute the modulus of {0} by {1}",
        "Determine {0} % {1}",
        "Get the remainder when {0} is divided by {1}",
        "Evaluate the modulus of {0} with {1}",
        "Find {0} modulo {1}",
        "Calculate the rest of {0} divided by {1}",
        "Compute {0} remainder {1}",
        "Determine the leftover of {0} after division by {1}",
        "Get {0} mod {1}",
        "Find the modulus result of {0} by {1}",
        "Calculate what remains of {0} after dividing by {1}",
        "Evaluate {0} modulo {1}",
        "Compute the remainder of {0} over {1}"
    ],
    "Power": [
        "Raise {0} to the power of {1}",
        "Calculate {0} to the {1}th power",
        "Find {0} raised to {1}",
        "Compute {0} ** {1}",
        "Determine {0} to the power {1}",
        "Evaluate {0} raised to the power of {1}",
        "Get {0} to the {1} power",
        "Calculate the result of {0} to the power {1}",
        "Find {0} exponentiated by {1}",
        "Compute {0} powered by {1}",
        "Determine the power of {0} with exponent {1}",
        "Raise {0} to {1}",
        "Evaluate {0} ^ {1}",
        "Compute {0} to the power of {1}",
        "Get the result of {0} raised to {1}"
    ],
    "SquareRoot": [
        "Calculate the square root of {0}",
        "Find the square root of {0}",
        "Compute the root of {0}",
        "Determine the square root value of {0}",
        "Get the square root of {0}",
        "Evaluate the sqrt of {0}",
        "Find the value of square root {0}",
        "Compute sqrt({0})",
        "Calculate root of {0}",
        "Determine the principal square root of {0}",
        "Get square root of {0}",
        "Evaluate the square root for {0}",
        "Find the sqrt value of {0}",
        "Compute the square root for {0}",
        "Calculate the square root value of {0}"
    ],
    "Factorial": [
        "Calculate factorial of {0}",
        "Find {0} factorial",
        "Compute the factorial of {0}",
        "Determine the product of numbers up to {0}",
        "Evaluate factorial for {0}",
        "Calculate {0}!",
        "Find the factorial value of {0}",
        "Compute {0} factorial",
        "Determine {0}!",
        "Evaluate the product of integers up to {0}",
        "Get the factorial of {0}",
        "Calculate the result of {0} factorial",
        "Find the product of all numbers up to {0}",
        "Compute factorial({0})",
        "Determine the factorial result of {0}"
    ],
    "GCD": [
        "Find the greatest common divisor of {0} and {1}",
        "Calculate the GCD of {0} and {1}",
        "Compute the greatest common factor of {0} and {1}",
        "Determine the highest common divisor of {0} and {1}",
        "Get the GCD of {0} and {1}",
        "Evaluate the greatest common divisor for {0} and {1}",
        "Find the largest common factor of {0} and {1}",
        "Compute GCD({0}, {1})",
        "Calculate the highest common factor of {0} and {1}",
        "Determine the GCD value of {0} and {1}",
        "Get the greatest divisor common to {0} and {1}",
        "Find the common divisor of {0} and {1}",
        "Compute the greatest common denominator of {0} and {1}",
        "Calculate the GCD for {0} and {1}",
        "Determine the largest divisor shared by {0} and {1}"
    ],
    "LCM": [
        "Find the least common multiple of {0} and {1}",
        "Calculate the LCM of {0} and {1}",
        "Compute the smallest common multiple of {0} and {1}",
        "Determine the lowest common multiple of {0} and {1}",
        "Get the LCM of {0} and {1}",
        "Evaluate the least common multiple for {0} and {1}",
        "Find the smallest multiple common to {0} and {1}",
        "Compute LCM({0}, {1})",
        "Calculate the lowest common multiple of {0} and {1}",
        "Determine the LCM value of {0} and {1}",
        "Get the least multiple shared by {0} and {1}",
        "Find the common multiple of {0} and {1}",
        "Compute the smallest common denominator of {0} and {1}",
        "Calculate the LCM for {0} and {1}",
        "Determine the smallest multiple shared by {0} and {1}"
    ],
    "Max": [
        "Find max of {0} and {1}",
        "Get the larger of {0} and {1}",
        "What’s the maximum of {0} and {1}",
        "Determine the greatest value between {0} and {1}",
        "Identify the larger number of {0} and {1}",
        "Select the maximum of {0} and {1}",
        "Compute the highest of {0} and {1}",
        "Find the biggest number between {0} and {1}",
        "Calculate the maximum value of {0} and {1}",
        "Get the greatest of {0} and {1}",
        "Determine the larger value of {0} and {1}",
        "Find the highest number of {0} and {1}",
        "Compute max({0}, {1})",
        "Evaluate the maximum of {0} and {1}",
        "Select the bigger of {0} and {1}"
    ],
    "Min": [
        "Find min of {0} and {1}",
        "Get the smaller of {0} and {1}",
        "What’s the minimum of {0} and {1}",
        "Determine the smallest value between {0} and {1}",
        "Identify the smaller number of {0} and {1}",
        "Select the minimum of {0} and {1}",
        "Compute the lowest of {0} and {1}",
        "Find the smallest number between {0} and {1}",
        "Calculate the minimum value of {0} and {1}",
        "Get the least of {0} and {1}",
        "Determine the smaller value of {0} and {1}",
        "Find the lowest number of {0} and {1}",
        "Compute min({0}, {1})",
        "Evaluate the minimum of {0} and {1}",
        "Select the lesser of {0} and {1}"
    ],
    "Mean": [
        "Calculate the average of {0}",
        "Find the mean of {0}",
        "Compute the average value of {0}",
        "Determine the mean value of {0}",
        "Get the average of {0}",
        "Evaluate the mean of {0}",
        "Find the arithmetic mean of {0}",
        "Compute mean({0})",
        "Calculate the central value of {0}",
        "Determine the average of the list {0}",
        "Get the mean value of {0}",
        "Find the average for {0}",
        "Compute the average of numbers in {0}",
        "Evaluate the central mean of {0}",
        "Calculate the mean for {0}"
    ],
    "Median": [
        "Calculate the median of {0}",
        "Find the median of {0}",
        "Compute the middle value of {0}",
        "Determine the median value of {0}",
        "Get the median of {0}",
        "Evaluate the median for {0}",
        "Find the central value of {0}",
        "Compute median({0})",
        "Calculate the middle number of {0}",
        "Determine the median of the list {0}",
        "Get the central value of {0}",
        "Find the median for {0}",
        "Compute the median of numbers in {0}",
        "Evaluate the middle value of {0}",
        "Calculate the median for {0}"
    ],
    "StandardDeviation": [
        "Calculate the standard deviation of {0}",
        "Find the standard deviation of {0}",
        "Compute the std dev of {0}",
        "Determine the standard deviation value of {0}",
        "Get the standard deviation of {0}",
        "Evaluate the std dev for {0}",
        "Find the deviation of {0}",
        "Compute std_dev({0})",
        "Calculate the spread of {0}",
        "Determine the standard deviation of the list {0}",
        "Get the std dev value of {0}",
        "Find the standard deviation for {0}",
        "Compute the standard deviation of numbers in {0}",
        "Evaluate the spread of {0}",
        "Calculate the std dev for {0}"
    ],
    "GreaterThan": [
        "Check if {0} is greater than {1}",
        "Is {0} larger than {1}",
        "Determine if {0} exceeds {1}",
        "Verify if {0} is more than {1}",
        "Evaluate if {0} > {1}",
        "See if {0} is bigger than {1}",
        "Check whether {0} is greater than {1}",
        "Determine if {0} is above {1}",
        "Is {0} more than {1}",
        "Verify if {0} surpasses {1}",
        "Evaluate whether {0} is larger than {1}",
        "Check if {0} is higher than {1}",
        "Determine if {0} is greater than {1}",
        "See if {0} exceeds {1}",
        "Is {0} greater than {1}"
    ],
    "LessThan": [
        "Check if {0} is less than {1}",
        "Is {0} smaller than {1}",
        "Determine if {0} is below {1}",
        "Verify if {0} is less than {1}",
        "Evaluate if {0} < {1}",
        "See if {0} is smaller than {1}",
        "Check whether {0} is less than {1}",
        "Determine if {0} is under {1}",
        "Is {0} less than {1}",
        "Verify if {0} is below {1}",
        "Evaluate whether {0} is smaller than {1}",
        "Check if {0} is lower than {1}",
        "Determine if {0} is less than {1}",
        "See if {0} is under {1}",
        "Is {0} smaller than {1}"
    ],
    "Equal": [
        "Check if {0} equals {1}",
        "Is {0} equal to {1}",
        "Determine if {0} is the same as {1}",
        "Verify if {0} == {1}",
        "Evaluate if {0} equals {1}",
        "See if {0} is identical to {1}",
        "Check whether {0} is equal to {1}",
        "Determine if {0} matches {1}",
        "Is {0} the same as {1}",
        "Verify if {0} is equal to {1}",
        "Evaluate whether {0} is the same as {1}",
        "Check if {0} is equivalent to {1}",
        "Determine if {0} equals {1}",
        "See if {0} is equal to {1}",
        "Is {0} equivalent to {1}"
    ],
    "NotEqual": [
        "Check if {0} is not equal to {1}",
        "Is {0} different from {1}",
        "Determine if {0} is not the same as {1}",
        "Verify if {0} != {1}",
        "Evaluate if {0} is not equal to {1}",
        "See if {0} differs from {1}",
        "Check whether {0} is not equal to {1}",
        "Determine if {0} does not match {1}",
        "Is {0} not the same as {1}",
        "Verify if {0} is different from {1}",
        "Evaluate whether {0} is not the same as {1}",
        "Check if {0} is not equivalent to {1}",
        "Determine if {0} is not equal to {1}",
        "See if {0} differs from {1}",
        "Is {0} not equal to {1}"
    ],
    "GreaterOrEqual": [
        "Check if {0} is greater than or equal to {1}",
        "Is {0} at least {1}",
        "Determine if {0} is no less than {1}",
        "Verify if {0} >= {1}",
        "Evaluate if {0} is greater or equal to {1}",
        "See if {0} is at least {1}",
        "Check whether {0} is greater than or equal to {1}",
        "Determine if {0} is not less than {1}",
        "Is {0} no smaller than {1}",
        "Verify if {0} is at least equal to {1}",
        "Evaluate whether {0} is greater or equal to {1}",
        "Check if {0} is greater than or equals {1}",
        "Determine if {0} is at least {1}",
        "See if {0} is not smaller than {1}",
        "Is {0} greater than or equal to {1}"
    ],
    "LessOrEqual": [
        "Check if {0} is less than or equal to {1}",
        "Is {0} at most {1}",
        "Determine if {0} is no more than {1}",
        "Verify if {0} <= {1}",
        "Evaluate if {0} is less or equal to {1}",
        "See if {0} is at most {1}",
        "Check whether {0} is less than or equal to {1}",
        "Determine if {0} is not more than {1}",
        "Is {0} no larger than {1}",
        "Verify if {0} is at most equal to {1}",
        "Evaluate whether {0} is less or equal to {1}",
        "Check if {0} is less than or equals {1}",
        "Determine if {0} is at most {1}",
        "See if {0} is not larger than {1}",
        "Is {0} less than or equal to {1}"
    ],
    "And": [
        "Check if {0} and {1} are true",
        "Is {0} and {1} both true",
        "Determine if both {0} and {1} hold",
        "Verify if {0} && {1}",
        "Evaluate if {0} and {1} are true",
        "See if both {0} and {1} are true",
        "Check whether {0} and {1} are both true",
        "Determine if {0} and {1} are satisfied",
        "Is {0} and {1} true",
        "Verify if both {0} and {1} are true",
        "Evaluate whether {0} and {1} hold",
        "Check if {0} and {1} are both valid",
        "Determine if {0} and {1} are true together",
        "See if {0} and {1} are both true",
        "Is {0} and {1} satisfied"
    ],
    "Or": [
        "Check if {0} or {1} is true",
        "Is either {0} or {1} true",
        "Determine if at least one of {0} or {1} holds",
        "Verify if {0} || {1}",
        "Evaluate if {0} or {1} is true",
        "See if either {0} or {1} is true",
        "Check whether {0} or {1} is true",
        "Determine if {0} or {1} is satisfied",
        "Is {0} or {1} true",
        "Verify if at least {0} or {1} is true",
        "Evaluate whether {0} or {1} holds",
        "Check if {0} or {1} is valid",
        "Determine if either {0} or {1} is true",
        "See if {0} or {1} is true",
        "Is either {0} or {1} satisfied"
    ],
    "Not": [
        "Check if {0} is not true",
        "Is {0} false",
        "Determine if {0} is not true",
        "Verify if !{0}",
        "Evaluate if {0} is not true",
        "See if {0} is false",
        "Check whether {0} is not true",
        "Determine if {0} is not satisfied",
        "Is {0} not true",
        "Verify if {0} is false",
        "Evaluate whether {0} is not true",
        "Check if {0} is not valid",
        "Determine if {0} is false",
        "See if {0} is not true",
        "Is {0} not satisfied"
    ],
    "IsEven": [
        "Check if {0} is even",
        "Is {0} an even number",
        "Determine if {0} is divisible by 2",
        "Verify if {0} is even",
        "Evaluate if {0} is an even integer",
        "See if {0} is even",
        "Check whether {0} is an even number",
        "Determine if {0} is evenly divisible by 2",
        "Is {0} even",
        "Verify if {0} is an even value",
        "Evaluate whether {0} is even",
        "Check if {0} is an even integer",
        "Determine if {0} is even",
        "See if {0} is divisible by two",
        "Is {0} an even number"
    ],
    "IsOdd": [
        "Check if {0} is odd",
        "Is {0} an odd number",
        "Determine if {0} is not divisible by 2",
        "Verify if {0} is odd",
        "Evaluate if {0} is an odd integer",
        "See if {0} is odd",
        "Check whether {0} is an odd number",
        "Determine if {0} is not evenly divisible by 2",
        "Is {0} odd",
        "Verify if {0} is an odd value",
        "Evaluate whether {0} is odd",
        "Check if {0} is an odd integer",
        "Determine if {0} is odd",
        "See if {0} is not divisible by two",
        "Is {0} an odd number"
    ],
    "IsPrime": [
        "Check if {0} is prime",
        "Is {0} a prime number",
        "Determine if {0} is a prime",
        "Verify if {0} is prime",
        "Evaluate if {0} is a prime number",
        "See if {0} is prime",
        "Check whether {0} is a prime number",
        "Determine if {0} has no divisors other than 1 and itself",
        "Is {0} prime",
        "Verify if {0} is a prime value",
        "Evaluate whether {0} is prime",
        "Check if {0} is a prime integer",
        "Determine if {0} is prime",
        "See if {0} is a prime number",
        "Is {0} a prime integer"
    ],
    "IsPerfectSquare": [
        "Check if {0} is a perfect square",
        "Is {0} a perfect square",
        "Determine if {0} is a square number",
        "Verify if {0} is a perfect square",
        "Evaluate if {0} is a square of an integer",
        "See if {0} is a perfect square",
        "Check whether {0} is a perfect square number",
        "Determine if {0} is the square of a number",
        "Is {0} a square number",
        "Verify if {0} is a perfect square value",
        "Evaluate whether {0} is a perfect square",
        "Check if {0} is a square integer",
        "Determine if {0} is a perfect square",
        "See if {0} is a square of an integer",
        "Is {0} a perfect square number"
    ],
    "IsDivisible": [
        "Check if {0} is divisible by {1}",
        "Is {0} divisible by {1}",
        "Determine if {0} can be divided by {1}",
        "Verify if {0} is divisible by {1}",
        "Evaluate if {0} is evenly divisible by {1}",
        "See if {0} is divisible by {1}",
        "Check whether {0} is divisible by {1}",
        "Determine if {0} divides evenly by {1}",
        "Is {0} evenly divisible by {1}",
        "Verify if {0} is a multiple of {1}",
        "Evaluate whether {0} is divisible by {1}",
        "Check if {0} is a multiple of {1}",
        "Determine if {0} is divisible by {1}",
        "See if {0} divides by {1} evenly",
        "Is {0} divisible by {1}"
    ],
    "If": [
        "If {0} is true, then {1}",
        "Execute {1} if {0} is true",
        "If {0} holds, perform {1}",
        "Check if {0} is true and do {1}",
        "Evaluate if {0} then {1}",
        "If condition {0} is true, run {1}",
        "Perform {1} when {0} is true",
        "If {0} is satisfied, execute {1}",
        "Do {1} if {0} is true",
        "Run {1} when condition {0} holds",
        "If {0} is true, compute {1}",
        "Execute {1} if {0} is valid",
        "If {0}, then perform {1}",
        "Check {0} and if true, do {1}",
        "If {0} is met, run {1}"
    ],
    "IfElse": [
        "If {0} is true, then {1}, else {2}",
        "Execute {1} if {0} is true, otherwise {2}",
        "If {0} holds, perform {1}, else {2}",
        "Check if {0} is true and do {1}, else do {2}",
        "Evaluate if {0} then {1}, else {2}",
        "If condition {0} is true, run {1}, else run {2}",
        "Perform {1} when {0} is true, otherwise {2}",
        "If {0} is satisfied, execute {1}, else {2}",
        "Do {1} if {0} is true, else do {2}",
        "Run {1} when condition {0} holds, else {2}",
        "If {0} is true, compute {1}, else compute {2}",
        "Execute {1} if {0} is valid, else {2}",
        "If {0}, then perform {1}, otherwise {2}",
        "Check {0} and if true, do {1}, else {2}",
        "If {0} is met, run {1}, else run {2}"
    ],
    "Ternary": [
        "{1} if {0} else {2}",
        "Return {1} if {0} is true, else {2}",
        "Evaluate {1} if {0} holds, else {2}",
        "Compute {1} when {0} is true, otherwise {2}",
        "If {0} then {1} else {2}",
        "Select {1} if {0} is true, else {2}",
        "Get {1} if {0} is valid, else {2}",
        "Determine {1} if {0} is satisfied, else {2}",
        "Choose {1} if {0} holds, else {2}",
        "Evaluate {1} if {0} is true, otherwise {2}",
        "Return {1} when {0} is true, else {2}",
        "If {0} is true, get {1}, else get {2}",
        "Compute {1} if {0}, else {2}",
        "Select {1} when {0} is true, otherwise {2}",
        "Determine {1} if {0} is true, else {2}"
    ],
    "ForLoop": [
        "For each number from {0} to {1}, do {2}",
        "Loop from {0} to {1} and perform {2}",
        "Iterate from {0} to {1} and execute {2}",
        "For numbers {0} to {1}, run {2}",
        "Repeat from {0} to {1} doing {2}",
        "For each value from {0} to {1}, compute {2}",
        "Loop through {0} to {1} and do {2}",
        "Iterate over range {0} to {1} and perform {2}",
        "For each integer from {0} to {1}, execute {2}",
        "Run {2} for each number from {0} to {1}",
        "Loop from {0} to {1} performing {2}",
        "Iterate from {0} to {1} computing {2}",
        "For {0} to {1}, do {2}",
        "Repeat {2} for numbers {0} to {1}",
        "Execute {2} for each value from {0} to {1}"
    ],
    "WhileLoop": [
        "While {0} is true, do {1}",
        "Loop while {0} holds and perform {1}",
        "Repeat {1} while {0} is true",
        "Execute {1} as long as {0} is true",
        "While {0} is valid, run {1}",
        "Continue {1} while {0} is satisfied",
        "Loop while {0} and do {1}",
        "Perform {1} while {0} remains true",
        "While condition {0} holds, execute {1}",
        "Repeat {1} as long as {0} is true",
        "While {0}, compute {1}",
        "Execute {1} while {0} is true",
        "Loop while {0} is true and perform {1}",
        "Continue running {1} while {0} holds",
        "While {0} is met, do {1}"
    ],
    "Break": [
        "Break the loop",
        "Exit the loop",
        "Stop the loop",
        "Terminate the loop",
        "End the loop",
        "Break out of the loop",
        "Cease the loop",
        "Halt the loop",
        "Exit from the loop",
        "Stop looping",
        "Terminate looping",
        "Break the iteration",
        "End the iteration",
        "Stop the iteration",
        "Cease looping"
    ],
    "Continue": [
        "Continue to the next iteration",
        "Skip to the next loop iteration",
        "Proceed to the next iteration",
        "Move to the next loop step",
        "Continue with the next loop",
        "Skip the current iteration",
        "Go to the next iteration",
        "Advance to the next loop iteration",
        "Continue the loop",
        "Proceed with the next loop step",
        "Skip to the next cycle",
        "Move to the next cycle",
        "Continue to the next step",
        "Skip this iteration and continue",
        "Go to the next loop iteration"
    ],
    "Return": [
        "Return {0}",
        "Give back {0}",
        "Output {0} as result",
        "Send back {0}",
        "Return the value {0}",
        "Provide {0} as output",
        "Get {0} as result",
        "Return {0} as the result",
        "Send {0} back",
        "Output the value {0}",
        "Give {0} as the result",
        "Return {0} value",
        "Provide the result {0}",
        "Send back the value {0}",
        "Return {0} as output"
    ],
    "FunctionCall": [
        "Call function {0} with {1}",
        "Execute function {0} with parameters {1}",
        "Run function {0} with {1}",
        "Invoke {0} with {1}",
        "Perform function {0} with {1}",
        "Call {0} using {1}",
        "Execute {0} with arguments {1}",
        "Run {0} with parameters {1}",
        "Invoke function {0} with {1}",
        "Perform {0} with arguments {1}",
        "Call the function {0} with {1}",
        "Execute {0} using {1}",
        "Run the function {0} with {1}",
        "Invoke {0} with parameters {1}",
        "Perform function {0} using {1}"
    ],
    "Print": [
        "Print {0}",
        "Display {0}",
        "Show {0}",
        "Output {0}",
        "Present the value {0}",
        "Exhibit {0} on screen",
        "Render {0} as output",
        "Write out {0}",
        "Print the value {0}",
        "Display {0} on console",
        "Show the result {0}",
        "Output the value {0}",
        "Present {0} on the screen",
        "Write {0} to output",
        "Exhibit the value {0}"
    ],
    "Read": [
        "Read input",
        "Get input from user",
        "Take input",
        "Receive input",
        "Read a value",
        "Get a value from input",
        "Take user input",
        "Receive a value",
        "Read input value",
        "Get input from the user",
        "Take a value from input",
        "Receive user input",
        "Read the input",
        "Get the user input",
        "Take input value"
    ],
    "SumEvens": [
        "Sum even numbers from {0} to {1}",
        "Total all even numbers between {0} and {1}",
        "Add up evens from {0} to {1}",
        "Calculate the sum of even integers from {0} to {1}",
        "Aggregate even values in range {0} to {1}",
        "Compute total of even numbers between {0} and {1}",
        "Find the sum of evens from {0} to {1}",
        "Calculate the total of even numbers from {0} to {1}",
        "Sum all even integers between {0} and {1}",
        "Add even numbers in range {0} to {1}",
        "Compute the sum of even values from {0} to {1}",
        "Total even numbers from {0} to {1}",
        "Find the aggregate of even integers from {0} to {1}",
        "Calculate sum of even numbers between {0} and {1}",
        "Sum up even values from {0} to {1}"
    ],
    "SumOdds": [
        "Sum odd numbers from {0} to {1}",
        "Total all odd numbers between {0} and {1}",
        "Add up odds from {0} to {1}",
        "Calculate the sum of odd integers from {0} to {1}",
        "Aggregate odd values in range {0} to {1}",
        "Compute total of odd numbers between {0} and {1}",
        "Find the sum of odds from {0} to {1}",
        "Calculate the total of odd numbers from {0} to {1}",
        "Sum all odd integers between {0} and {1}",
        "Add odd numbers in range {0} to {1}",
        "Compute the sum of odd values from {0} to {1}",
        "Total odd numbers from {0} to {1}",
        "Find the aggregate of odd integers from {0} to {1}",
        "Calculate sum of odd numbers between {0} and {1}",
        "Sum up odd values from {0} to {1}"
    ],
    "ListAppend": [
        "Append {0} to list {1}",
        "Add {0} to the list {1}",
        "Insert {0} into list {1}",
        "Put {0} in list {1}",
        "Add element {0} to {1}",
        "Append element {0} to {1}",
        "Insert {0} into {1}",
        "Add {0} to the end of {1}",
        "Put {0} at the end of list {1}",
        "Append {0} to the collection {1}",
        "Add {0} to {1} list",
        "Insert element {0} to list {1}",
        "Put element {0} in {1}",
        "Append {0} to the list named {1}",
        "Add {0} to the list {1}"
    ],
    "ListRemove": [
        "Remove {0} from list {1}",
        "Delete {0} from {1}",
        "Take {0} out of list {1}",
        "Remove element {0} from {1}",
        "Delete element {0} from list {1}",
        "Remove {0} from the list {1}",
        "Take out {0} from {1}",
        "Delete {0} from the collection {1}",
        "Remove {0} from {1} list",
        "Erase {0} from list {1}",
        "Take {0} out of the list {1}",
        "Delete element {0} from {1}",
        "Remove {0} from the list named {1}",
        "Erase {0} from {1}",
        "Take element {0} out of {1}"
    ],
    "ListLength": [
        "Find the length of list {0}",
        "Get the size of {0}",
        "Calculate the number of elements in {0}",
        "Determine the length of {0}",
        "Compute the size of list {0}",
        "Find how many items in {0}",
        "Get the length of the list {0}",
        "Calculate the count of elements in {0}",
        "Determine the size of {0}",
        "Find the number of elements in list {0}",
        "Compute the length of {0}",
        "Get the count of items in {0}",
        "Calculate the length of the list {0}",
        "Find the size of the list {0}",
        "Determine how many elements in {0}"
    ],
    "ListSort": [
        "Sort list {0} in ascending order",
        "Arrange {0} in ascending order",
        "Order list {0} from smallest to largest",
        "Sort {0} ascending",
        "Put list {0} in order",
        "Arrange the list {0} in ascending order",
        "Sort the elements of {0} from low to high",
        "Order {0} in ascending sequence",
        "Sort {0} in increasing order",
        "Arrange {0} from smallest to largest",
        "Put {0} in ascending order",
        "Sort the list {0} ascending",
        "Order the elements of {0} in ascending order",
        "Arrange list {0} in increasing order",
        "Sort {0} from low to high"
    ],
    "ListReverse": [
        "Reverse list {0}",
        "Flip the order of {0}",
        "Invert list {0}",
        "Reverse the elements of {0}",
        "Turn {0} backwards",
        "Flip list {0}",
        "Invert the order of {0}",
        "Reverse the sequence of {0}",
        "Put {0} in reverse order",
        "Turn the list {0} backwards",
        "Reverse the list {0}",
        "Flip the elements of {0}",
        "Invert {0} sequence",
        "Reverse order of list {0}",
        "Turn {0} around"
    ],
    "ListContains": [
        "Check if list {0} contains {1}",
        "Does {0} include {1}",
        "Is {1} in list {0}",
        "Verify if {0} contains {1}",
        "Determine if {1} is in {0}",
        "See if {0} includes {1}",
        "Check whether {0} has {1}",
        "Is {1} part of list {0}",
        "Verify if {1} exists in {0}",
        "Determine if {0} contains element {1}",
        "Check if {1} is in the list {0}",
        "See if {1} is included in {0}",
        "Is {1} within {0}",
        "Verify if {0} includes {1}",
        "Check if {0} contains the element {1}"
    ],
    "ListIndex": [
        "Find the index of {0} in list {1}",
        "Get the position of {0} in {1}",
        "Locate {0} in list {1}",
        "Determine the index of {0} in {1}",
        "Find where {0} is in {1}",
        "Get the index of {0} in the list {1}",
        "Locate the position of {0} in {1}",
        "Find the location of {0} in {1}",
        "Determine where {0} is in list {1}",
        "Get the position of {0} in list {1}",
        "Find the index of element {0} in {1}",
        "Locate {0} in the list {1}",
        "Determine the position of {0} in {1}",
        "Find where {0} appears in {1}",
        "Get the location of {0} in {1}"
    ],
    "ListSlice": [
        "Get slice of list {0} from {1} to {2}",
        "Extract elements of {0} from index {1} to {2}",
        "Take a slice of {0} between {1} and {2}",
        "Get part of list {0} from {1} to {2}",
        "Slice {0} from {1} to {2}",
        "Extract {0} from index {1} to {2}",
        "Get a portion of {0} from {1} to {2}",
        "Take elements of {0} between {1} and {2}",
        "Slice the list {0} from {1} to {2}",
        "Extract a section of {0} from {1} to {2}",
        "Get {0} elements from {1} to {2}",
        "Take a part of {0} from index {1} to {2}",
        "Slice {0} between {1} and {2}",
        "Extract portion of list {0} from {1} to {2}",
        "Get the slice of {0} from {1} to {2}"
    ],
    "AddLinkedLists": [
        "Step 1: Check if first list is null or zero, return second list\nStep 2: Check if second list is null or zero, return first list\nStep 3: Initialize pointers first, second, prev\nStep 4: Find lengths of both lists\nStep 5: If second list is longer, swap first and second\nStep 6: Traverse first list, add second list values and carry, update carry\nStep 7: If carry remains, add new node\nStep 8: Return longer list",
        "Step 1: Verify if first linked list is empty or zero, return second list\nStep 2: Verify if second linked list is empty or zero, return first list\nStep 3: Set up pointers for first, second, and previous nodes\nStep 4: Compute lengths of both lists\nStep 5: If second list has more nodes, swap pointers\nStep 6: Iterate through first list, add values from second list with carry\nStep 7: If carry exists, append new node\nStep 8: Return list with greater length",
        "Step 1: Check if list one is null or zero, return list two\nStep 2: Check if list two is null or zero, return list one\nStep 3: Initialize first, second, prev pointers\nStep 4: Calculate the lengths of both lists\nStep 5: Swap lists if second is longer\nStep 6: Traverse first list, add second list values with carry\nStep 7: Add new node if carry remains\nStep 8: Return the longer list",
        "Step 1: If first list is empty or zero, return second list\nStep 2: If second list is empty or zero, return first list\nStep 3: Set pointers first, second, and prev\nStep 4: Find the lengths of both lists\nStep 5: If second list is longer, swap them\nStep 6: Iterate first list, add values from second with carry\nStep 7: Append node if carry is non-zero\nStep 8: Return the list with more nodes",
        "Step 1: Check if first linked list is null, return second\nStep 2: Check if second linked list is null, return first\nStep 3: Initialize pointers for first, second, prev\nStep 4: Compute list lengths\nStep 5: Swap if second list is longer\nStep 6: Traverse first, add second’s values and carry\nStep 7: If carry remains, add node\nStep 8: Return longer list",
        "Step 1: Verify if first list is empty, return second\nStep 2: Verify if second list is empty, return first\nStep 3: Set up first, second, prev pointers\nStep 4: Calculate lengths of lists\nStep 5: If second list has more nodes, swap\nStep 6: Iterate first list, sum second list values with carry\nStep 7: Append new node if carry exists\nStep 8: Return list with greater length",
        "Step 1: Check if first list is zero or null, return second\nStep 2: Check if second list is zero or null, return first\nStep 3: Initialize first, second, and prev pointers\nStep 4: Find lengths of both lists\nStep 5: Swap pointers if second list is longer\nStep 6: Traverse first, add second values with carry\nStep 7: Add node if carry remains\nStep 8: Return the longer list",
        "Step 1: If first list is null, return second list\nStep 2: If second list is null, return first list\nStep 3: Set pointers for first, second, prev\nStep 4: Compute lengths of both lists\nStep 5: Swap if second list is longer\nStep 6: Iterate first list, add second values and carry\nStep 7: If carry is non-zero, append node\nStep 8: Return longer list",
        "Step 1: Check if first linked list is empty or zero, return second\nStep 2: Check if second linked list is empty or zero, return first\nStep 3: Initialize first, second, prev pointers\nStep 4: Calculate list lengths\nStep 5: If second list is longer, swap pointers\nStep 6: Traverse first list, sum second list values with carry\nStep 7: Add new node if carry remains\nStep 8: Return the list with more nodes",
        "Step 1: Verify if first list is null or zero, return second list\nStep 2: Verify if second list is null or zero, return first list\nStep 3: Set up pointers first, second, prev\nStep 4: Find lengths of both lists\nStep 5: Swap lists if second is longer\nStep 6: Iterate first, add second list values with carry\nStep 7: Append node if carry exists\nStep 8: Return longer list"
    ],
    "ReverseLinkedList": [
        "Reverse linked list {0}",
        "Invert the linked list {0}",
        "Flip the linked list {0}",
        "Reverse the order of {0}",
        "Turn {0} backwards",
        "Invert linked list {0}",
        "Flip the order of linked list {0}",
        "Reverse the sequence of {0}",
        "Put linked list {0} in reverse order",
        "Turn the linked list {0} backwards",
        "Reverse {0}",
        "Invert the order of {0}",
        "Flip {0} sequence",
        "Reverse the linked list {0}",
        "Turn {0} around"
    ],
    "LinkedListLength": [
        "Find the length of linked list {0}",
        "Get the size of linked list {0}",
        "Calculate the number of nodes in {0}",
        "Determine the length of {0}",
        "Compute the size of linked list {0}",
        "Find how many nodes in {0}",
        "Get the length of the linked list {0}",
        "Calculate the count of nodes in {0}",
        "Determine the size of {0}",
        "Find the number of nodes in linked list {0}",
        "Compute the length of {0}",
        "Get the count of nodes in {0}",
        "Calculate the length of the linked list {0}",
        "Find the size of linked list {0}",
        "Determine how many nodes in {0}"
    ],
    "LinkedListInsert": [
        "Insert {0} into linked list {1} at position {2}",
        "Add {0} to linked list {1} at index {2}",
        "Put {0} in linked list {1} at {2}",
        "Insert node {0} into {1} at position {2}",
        "Add node {0} to {1} at {2}",
        "Insert {0} at {2} in linked list {1}",
        "Put node {0} in {1} at index {2}",
        "Add {0} into {1} at position {2}",
        "Insert element {0} into linked list {1} at {2}",
        "Add {0} to the linked list {1} at {2}",
        "Put {0} at position {2} in {1}",
        "Insert {0} into {1} at index {2}",
        "Add node {0} into linked list {1} at {2}",
        "Insert {0} at index {2} in {1}",
        "Put {0} into linked list {1} at {2}"
    ],
    "StringConcat": [
        "Concatenate {0} and {1}",
        "Join {0} with {1}",
        "Combine {0} and {1}",
        "Append {1} to {0}",
        "Merge {0} with {1}",
        "Concatenate strings {0} and {1}",
        "Join the strings {0} and {1}",
        "Combine strings {0} and {1}",
        "Append string {1} to {0}",
        "Merge strings {0} and {1}",
        "Concatenate {0} with {1}",
        "Join {0} to {1}",
        "Combine {0} with string {1}",
        "Append {1} to string {0}",
        "Merge {0} and {1} strings"
    ],
    "StringLength": [
        "Find the length of string {0}",
        "Get the size of {0}",
        "Calculate the number of characters in {0}",
        "Determine the length of {0}",
        "Compute the size of string {0}",
        "Find how many characters in {0}",
        "Get the length of the string {0}",
        "Calculate the count of characters in {0}",
        "Determine the size of {0}",
        "Find the number of characters in string {0}",
        "Compute the length of {0}",
        "Get the count of characters in {0}",
        "Calculate the length of the string {0}",
        "Find the size of string {0}",
        "Determine how many characters in {0}"
    ],
    "StringSubstring": [
        "Get substring of {0} from {1} to {2}",
        "Extract part of {0} from index {1} to {2}",
        "Take a substring of {0} between {1} and {2}",
        "Get part of string {0} from {1} to {2}",
        "Slice {0} from {1} to {2}",
        "Extract {0} from index {1} to {2}",
        "Get a portion of {0} from {1} to {2}",
        "Take characters of {0} between {1} and {2}",
        "Slice the string {0} from {1} to {2}",
        "Extract a section of {0} from {1} to {2}",
        "Get {0} characters from {1} to {2}",
        "Take a part of {0} from index {1} to {2}",
        "Slice {0} between {1} and {2}",
        "Extract portion of string {0} from {1} to {2}",
        "Get the substring of {0} from {1} to {2}"
    ],
    "StringUpper": [
        "Convert {0} to uppercase",
        "Make {0} uppercase",
        "Change {0} to all caps",
        "Transform {0} to uppercase",
        "Convert string {0} to uppercase",
        "Make the string {0} uppercase",
        "Change string {0} to all caps",
        "Transform {0} to all uppercase",
        "Convert {0} to capital letters",
        "Make {0} all caps",
        "Change {0} to uppercase letters",
        "Transform string {0} to uppercase",
        "Convert {0} to upper case",
        "Make {0} upper case",
        "Change {0} to all capital letters"
    ],
    "StringLower": [
        "Convert {0} to lowercase",
        "Make {0} lowercase",
        "Change {0} to all lowercase",
        "Transform {0} to lowercase",
        "Convert string {0} to lowercase",
        "Make the string {0} lowercase",
        "Change string {0} to all lowercase",
        "Transform {0} to all lowercase",
        "Convert {0} to small letters",
        "Make {0} all lowercase",
        "Change {0} to lowercase letters",
        "Transform string {0} to lowercase",
        "Convert {0} to lower case",
        "Make {0} lower case",
        "Change {0} to all small letters"
    ],
    "StringContains": [
        "Check if string {0} contains {1}",
        "Does {0} include {1}",
        "Is {1} in string {0}",
        "Verify if {0} contains {1}",
        "Determine if {1} is in {0}",
        "See if {0} includes {1}",
        "Check whether {0} has {1}",
        "Is {1} part of string {0}",
        "Verify if {1} exists in {0}",
        "Determine if {0} contains substring {1}",
        "Check if {1} is in the string {0}",
        "See if {1} is included in {0}",
        "Is {1} within {0}",
        "Verify if {0} includes {1}",
        "Check if {0} contains the substring {1}"
    ],
    "Fibonacci": [
        "Calculate the {0}th Fibonacci number",
        "Find the {0}th Fibonacci value",
        "Compute Fibonacci number {0}",
        "Determine the {0}th number in Fibonacci sequence",
        "Get the {0}th Fibonacci number",
        "Evaluate Fibonacci for {0}",
        "Find Fibonacci({0})",
        "Compute the {0}th Fibonacci",
        "Calculate the Fibonacci number at position {0}",
        "Determine the {0}th Fibonacci value",
        "Get the Fibonacci number for {0}",
        "Find the {0}th number in the Fibonacci sequence",
        "Compute Fibonacci value for {0}",
        "Calculate the {0}th Fibonacci term",
        "Determine Fibonacci number {0}"
    ],
            "BinarySearch": [
            "Perform binary search for {0} in list {1}",
            "Search for {0} in sorted list {1} using binary search",
            "Find {0} in {1} with binary search",
            "Locate {0} in sorted list {1} using binary search",
            "Binary search {0} in list {1}",
            "Search {0} in {1} with binary search",
            "Find the position of {0} in {1} using binary search",
            "Locate {0} in {1} with binary search",
            "Perform binary search to find {0} in {1}",
            "Search for {0} in sorted {1} using binary search",
            "Binary search for {0} in {1}",
            "Find {0} in sorted list {1} with binary search",
            "Locate the position of {0} in {1} using binary search",
            "Search {0} in list {1} using binary search",
            "Perform binary search on {1} for {0}"
        ],
        "GraphDFS": [
            "Perform depth-first search on graph {0} starting at {1}",
            "Execute DFS on graph {0} from node {1}",
            "Traverse graph {0} using depth-first search starting at {1}",
            "Explore {0} with DFS from {1}",
            "Run depth-first search on {0} starting at {1}",
            "Perform DFS traversal on graph {0} from node {1}",
            "Search graph {0} with depth-first search from {1}",
            "Traverse {0} using DFS starting at {1}",
            "Execute depth-first search on {0} from {1}",
            "Explore graph {0} using depth-first search at {1}",
            "Run DFS on {0} starting from node {1}",
            "Perform depth-first traversal on {0} from {1}",
            "Search {0} with DFS from node {1}",
            "Traverse graph {0} with depth-first search at {1}",
            "Execute DFS on graph {0} starting at node {1}"
        ],
        "GraphBFS": [
            "Perform breadth-first search on graph {0} starting at {1}",
            "Execute BFS on graph {0} from node {1}",
            "Traverse graph {0} using breadth-first search starting at {1}",
            "Explore {0} with BFS from {1}",
            "Run breadth-first search on {0} starting at {1}",
            "Perform BFS traversal on graph {0} from node {1}",
            "Search graph {0} with breadth-first search from {1}",
            "Traverse {0} using BFS starting at {1}",
            "Execute breadth-first search on {0} from {1}",
            "Explore graph {0} using breadth-first search at {1}",
            "Run BFS on {0} starting from node {1}",
            "Perform breadth-first traversal on {0} from {1}",
            "Search {0} with BFS from node {1}",
            "Traverse graph {0} with breadth-first search at {1}",
            "Execute BFS on graph {0} starting at node {1}"
        ],
        "SortQuick": [
            "Sort list {0} using quicksort",
            "Apply quicksort to {0}",
            "Order {0} with quicksort",
            "Sort {0} using quicksort algorithm",
            "Arrange list {0} with quicksort",
            "Perform quicksort on {0}",
            "Sort the list {0} using quicksort",
            "Order list {0} with quicksort algorithm",
            "Apply quicksort to sort {0}",
            "Sort {0} in ascending order with quicksort",
            "Arrange {0} using quicksort",
            "Perform quicksort to order {0}",
            "Sort list {0} with quicksort method",
            "Order {0} using quicksort",
            "Apply quicksort algorithm to {0}"
        ],
        "SortMerge": [
            "Sort list {0} using mergesort",
            "Apply mergesort to {0}",
            "Order {0} with mergesort",
            "Sort {0} using mergesort algorithm",
            "Arrange list {0} with mergesort",
            "Perform mergesort on {0}",
            "Sort the list {0} using mergesort",
            "Order list {0} with mergesort algorithm",
            "Apply mergesort to sort {0}",
            "Sort {0} in ascending order with mergesort",
            "Arrange {0} using mergesort",
            "Perform mergesort to order {0}",
            "Sort list {0} with mergesort method",
            "Order {0} using mergesort",
            "Apply mergesort algorithm to {0}"
        ],
        "HashSetAdd": [
            "Add {0} to hash set {1}",
            "Insert {0} into hash set {1}",
            "Put {0} in hash set {1}",
            "Add element {0} to {1}",
            "Insert element {0} into {1}",
            "Put element {0} in {1}",
            "Add {0} to the hash set {1}",
            "Insert {0} into the set {1}",
            "Put {0} into hash set {1}",
            "Add item {0} to {1}",
            "Insert item {0} into hash set {1}",
            "Put item {0} in {1}",
            "Add {0} to set {1}",
            "Insert {0} to hash set {1}",
            "Put {0} into the hash set {1}"
        ],
        "HashSetContains": [
            "Check if hash set {0} contains {1}",
            "Does {0} include {1}",
            "Is {1} in hash set {0}",
            "Verify if {0} contains {1}",
            "Determine if {1} is in {0}",
            "See if {0} includes {1}",
            "Check whether {0} has {1}",
            "Is {1} part of hash set {0}",
            "Verify if {1} exists in {0}",
            "Determine if {0} contains element {1}",
            "Check if {1} is in the hash set {0}",
            "See if {1} is included in {0}",
            "Is {1} within {0}",
            "Verify if {0} includes {1}",
            "Check if {0} contains the element {1}"
        ],
        "StackPush": [
            "Push {0} onto stack {1}",
            "Add {0} to stack {1}",
            "Insert {0} onto {1}",
            "Put {0} on stack {1}",
            "Push element {0} to {1}",
            "Add element {0} onto stack {1}",
            "Insert element {0} to {1}",
            "Put element {0} on {1}",
            "Push {0} to the stack {1}",
            "Add {0} to the top of {1}",
            "Insert {0} onto stack {1}",
            "Put {0} onto {1}",
            "Push item {0} to {1}",
            "Add {0} onto stack {1}",
            "Insert {0} on top of {1}"
        ],
        "StackPop": [
            "Pop from stack {0}",
            "Remove the top element from {0}",
            "Take the top item off {0}",
            "Pop the top of stack {0}",
            "Remove element from {0}",
            "Take off the top of {0}",
            "Pop element from stack {0}",
            "Remove the top from {0}",
            "Take the top element from stack {0}",
            "Pop top item from {0}",
            "Remove top item from stack {0}",
            "Take top off {0}",
            "Pop from the stack {0}",
            "Remove the top element of {0}",
            "Take the top from stack {0}"
        ],
        "QueueEnqueue": [
            "Enqueue {0} into queue {1}",
            "Add {0} to queue {1}",
            "Insert {0} into {1}",
            "Put {0} in queue {1}",
            "Enqueue element {0} to {1}",
            "Add element {0} into queue {1}",
            "Insert element {0} to {1}",
            "Put element {0} in {1}",
            "Enqueue {0} to the queue {1}",
            "Add {0} to the end of {1}",
            "Insert {0} into queue {1}",
            "Put {0} into {1}",
            "Enqueue item {0} to {1}",
            "Add {0} into queue {1}",
            "Insert {0} at the end of {1}"
        ],
        "QueueDequeue": [
            "Dequeue from queue {0}",
            "Remove the front element from {0}",
            "Take the front item off {0}",
            "Dequeue the front of queue {0}",
            "Remove element from {0}",
            "Take off the front of {0}",
            "Dequeue element from queue {0}",
            "Remove the front from {0}",
            "Take the front element from queue {0}",
            "Dequeue front item from {0}",
            "Remove front item from queue {0}",
            "Take front off {0}",
            "Dequeue from the queue {0}",
            "Remove the front element of {0}",
            "Take the front from queue {0}"
        ]
    }

def generate_examples(num_examples=90000):
    examples = []
    intents = list(TEMPLATES.keys())
    
    for _ in range(num_examples):
        intent = random.choice(intents)
        template = random.choice(TEMPLATES[intent])
        
        # Arithmetic Operations
        if intent in ["Add", "Subtract", "Multiply", "Divide", "Modulus", "Power", "Max", "Min", "GCD", "LCM"]:
            num1 = random.randint(1, 100)
            num2 = random.randint(1, 100) if intent != "Divide" else random.randint(1, 100)  # Avoid division by zero
            input_text = template.format(num1, num2)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"numbers": [num1, num2]}
            ast = {
                "Add": f"add({num1}, {num2})",
                "Subtract": f"subtract({num1}, {num2})",
                "Multiply": f"multiply({num1}, {num2})",
                "Divide": f"divide({num1}, {num2})",
                "Modulus": f"modulus({num1}, {num2})",
                "Power": f"power({num1}, {num2})",
                "Max": f"max({num1}, {num2})",
                "Min": f"min({num1}, {num2})",
                "GCD": f"gcd({num1}, {num2})",
                "LCM": f"lcm({num1}, {num2})"
            }[intent]
            output = {
                "Add": num1 + num2,
                "Subtract": num1 - num2,
                "Multiply": num1 * num2,
                "Divide": num1 / num2 if num2 != 0 else "Error: Division by zero",
                "Modulus": num1 % num2 if num2 != 0 else "Error: Modulus by zero",
                "Power": num1 ** num2,
                "Max": max(num1, num2),
                "Min": min(num1, num2),
                "GCD": math.gcd(num1, num2),
                "LCM": abs(num1 * num2) // math.gcd(num1, num2) if math.gcd(num1, num2) != 0 else "Error: GCD is zero"
            }[intent]
        
        # Square Root
        elif intent == "SquareRoot":
            num = random.randint(1, 100)
            input_text = template.format(num)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"number": num}
            ast = f"sqrt({num})"
            output = math.sqrt(num)
        
        # Factorial
        elif intent == "Factorial":
            num = random.randint(1, 10)
            input_text = template.format(num)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"number": num}
            ast = f"factorial({num})"
            output = math.factorial(num)
        
        # Statistical Operations
        elif intent in ["Mean", "Median", "StandardDeviation"]:
            numbers = [random.randint(1, 100) for _ in range(random.randint(3, 10))]
            numbers_str = f"[{', '.join(map(str, numbers))}]"
            input_text = template.format(numbers_str)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"numbers": numbers}
            ast = {
                "Mean": f"mean({numbers_str})",
                "Median": f"median({numbers_str})",
                "StandardDeviation": f"std_dev({numbers_str})"
            }[intent]
            output = {
                "Mean": sum(numbers) / len(numbers),
                "Median": sorted(numbers)[len(numbers)//2] if len(numbers) % 2 else sum(sorted(numbers)[len(numbers)//2-1:len(numbers)//2+1])/2,
                "StandardDeviation": math.sqrt(sum((x - sum(numbers)/len(numbers))**2 for x in numbers) / len(numbers))
            }[intent]
        
        # Relational Operators
        elif intent in ["GreaterThan", "LessThan", "Equal", "NotEqual", "GreaterOrEqual", "LessOrEqual"]:
            num1 = random.randint(1, 100)
            num2 = random.randint(1, 100)
            input_text = template.format(num1, num2)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"numbers": [num1, num2]}
            ast = {
                "GreaterThan": f"greater_than({num1}, {num2})",
                "LessThan": f"less_than({num1}, {num2})",
                "Equal": f"equal({num1}, {num2})",
                "NotEqual": f"not_equal({num1}, {num2})",
                "GreaterOrEqual": f"greater_or_equal({num1}, {num2})",
                "LessOrEqual": f"less_or_equal({num1}, {num2})"
            }[intent]
            output = {
                "GreaterThan": num1 > num2,
                "LessThan": num1 < num2,
                "Equal": num1 == num2,
                "NotEqual": num1 != num2,
                "GreaterOrEqual": num1 >= num2,
                "LessOrEqual": num1 <= num2
            }[intent]
        
        # Logical Operations
        elif intent in ["And", "Or"]:
            bool1 = random.choice([True, False])
            bool2 = random.choice([True, False])
            input_text = template.format(str(bool1).lower(), str(bool2).lower())
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"conditions": [bool1, bool2]}
            ast = {
                "And": f"and({bool1}, {bool2})",
                "Or": f"or({bool1}, {bool2})"
            }[intent]
            output = {
                "And": bool1 and bool2,
                "Or": bool1 or bool2
            }[intent]
        
        elif intent == "Not":
            bool_val = random.choice([True, False])
            input_text = template.format(str(bool_val).lower())
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"condition": bool_val}
            ast = f"not({bool_val})"
            output = not bool_val
        
        # Number Checks
        elif intent in ["IsEven", "IsOdd", "IsPrime", "IsPerfectSquare", "IsDivisible"]:
            num1 = random.randint(1, 100)
            num2 = random.randint(1, 100) if intent == "IsDivisible" else None
            input_text = template.format(num1, num2) if intent == "IsDivisible" else template.format(num1)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"number": num1} if intent != "IsDivisible" else {"numbers": [num1, num2]}
            ast = {
                "IsEven": f"is_even({num1})",
                "IsOdd": f"is_odd({num1})",
                "IsPrime": f"is_prime({num1})",
                "IsPerfectSquare": f"is_perfect_square({num1})",
                "IsDivisible": f"is_divisible({num1}, {num2})"
            }[intent]
            output = {
                "IsEven": num1 % 2 == 0,
                "IsOdd": num1 % 2 != 0,
                "IsPrime": all(num1 % i != 0 for i in range(2, int(math.sqrt(num1)) + 1)) and num1 > 1,
                "IsPerfectSquare": int(math.sqrt(num1)) ** 2 == num1,
                "IsDivisible": num1 % num2 == 0 if num2 != 0 else "Error: Division by zero"
            }[intent]
        
        # Control Structures
        elif intent == "If":
            condition = random.choice(["x > 5", "x < 10", "x == 7"])
            action = random.choice(["print('True')", "x = x + 1"])
            input_text = template.format(condition, action)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"condition": condition, "action": action}
            ast = f"if({condition}, {action})"
            output = f"Evaluated if {condition} then {action}"
        
        elif intent == "IfElse":
            condition = random.choice(["x > 5", "x < 10", "x == 7"])
            action1 = random.choice(["print('True')", "x = x + 1"])
            action2 = random.choice(["print('False')", "x = x - 1"])
            input_text = template.format(condition, action1, action2)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"condition": condition, "action_true": action1, "action_false": action2}
            ast = f"if_else({condition}, {action1}, {action2})"
            output = f"Evaluated if {condition} then {action1} else {action2}"
        
        elif intent == "Ternary":
            condition = random.choice(["x > 5", "x < 10", "x == 7"])
            value1 = random.randint(1, 100)
            value2 = random.randint(1, 100)
            input_text = template.format(condition, value1, value2)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"condition": condition, "value_true": value1, "value_false": value2}
            ast = f"ternary({condition}, {value1}, {value2})"
            output = f"Evaluated {value1} if {condition} else {value2}"
        
        # Loops
        elif intent == "ForLoop":
            start = random.randint(1, 10)
            end = random.randint(start + 1, 20)
            action = random.choice(["print(i)", "sum = sum + i"])
            input_text = template.format(start, end, action)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"range": [start, end], "action": action}
            ast = f"for(i, {start}, {end}, {action})"
            output = f"Looped from {start} to {end} performing {action}"
        
        elif intent == "WhileLoop":
            condition = random.choice(["x < 10", "x > 0"])
            action = random.choice(["print(x)", "x = x + 1"])
            input_text = template.format(condition, action)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"condition": condition, "action": action}
            ast = f"while({condition}, {action})"
            output = f"Looped while {condition} performing {action}"
        
        elif intent in ["Break", "Continue"]:
            input_text = template
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {}
            ast = intent.lower()
            output = f"Performed {intent.lower()} in loop"
        
        # Functions
        elif intent == "Return":
            value = random.choice([random.randint(1, 100), "'result'"])
            input_text = template.format(value)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"value": value}
            ast = f"return({value})"
            output = f"Returned {value}"
        
        elif intent == "FunctionCall":
            func_name = random.choice(["add", "max", "factorial"])
            params = random.choice(["5, 10", "3", "7"])
            input_text = template.format(func_name, params)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"function": func_name, "parameters": params}
            ast = f"call({func_name}, {params})"
            output = f"Called {func_name} with {params}"
        
        # Input/Output
        elif intent == "Print":
            text = random.choice(["hello world", "test", "ENG rocks"])
            input_text = template.format(text)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"text": text}
            ast = f"print('{text}')"
            output = f"Printed: {text}"
        
        elif intent == "Read":
            input_text = template
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {}
            ast = "read()"
            output = "Read input from user"
        
        # List Operations
        elif intent in ["SumEvens", "SumOdds"]:
            start = random.randint(1, 10)
            end = random.randint(start + 1, 100)
            input_text = template.format(start, end)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"range": [start, end], "filter": "even" if intent == "SumEvens" else "odd"}
            ast = f"sum(filter({'even' if intent == 'SumEvens' else 'odd'}, range({start}, {end})))"
            output = sum(i for i in range(start, end + 1) if i % 2 == (0 if intent == "SumEvens" else 1))
        
        elif intent in ["ListAppend", "ListRemove", "ListContains", "ListIndex"]:
            element = random.randint(1, 100)
            list_name = random.choice(["my_list", "data", "numbers"])
            input_text = template.format(element, list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"element": element, "list": list_name}
            ast = {
                "ListAppend": f"append({list_name}, {element})",
                "ListRemove": f"remove({list_name}, {element})",
                "ListContains": f"contains({list_name}, {element})",
                "ListIndex": f"index({list_name}, {element})"
            }[intent]
            output = {
                "ListAppend": f"Appended {element} to {list_name}",
                "ListRemove": f"Removed {element} from {list_name}",
                "ListContains": f"Checked if {list_name} contains {element}",
                "ListIndex": f"Found index of {element} in {list_name}"
            }[intent]
        
        elif intent == "ListLength":
            list_name = random.choice(["my_list", "data", "numbers"])
            input_text = template.format(list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name}
            ast = f"length({list_name})"
            output = f"Length of {list_name}"
        
        elif intent in ["ListSort", "ListReverse"]:
            list_name = random.choice(["my_list", "data", "numbers"])
            input_text = template.format(list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name}
            ast = {
                "ListSort": f"sort({list_name})",
                "ListReverse": f"reverse({list_name})"
            }[intent]
            output = {
                "ListSort": f"Sorted {list_name}",
                "ListReverse": f"Reversed {list_name}"
            }[intent]
        
        elif intent == "ListSlice":
            list_name = random.choice(["my_list", "data", "numbers"])
            start = random.randint(0, 5)
            end = random.randint(start + 1, 10)
            input_text = template.format(list_name, start, end)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name, "start": start, "end": end}
            ast = f"slice({list_name}, {start}, {end})"
            output = f"Sliced {list_name} from {start} to {end}"
        
        # Linked List Operations
        elif intent == "AddLinkedLists":
            input_text = template
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"lists": ["l1", "l2"]}
            ast = "add_linked_lists(l1, l2)"
            output = "Linked list sum"
        
        elif intent == "ReverseLinkedList":
            list_name = random.choice(["list1", "list2"])
            input_text = template.format(list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name}
            ast = f"reverse_linked_list({list_name})"
            output = f"Reversed linked list {list_name}"
        
        elif intent == "LinkedListLength":
            list_name = random.choice(["list1", "list2"])
            input_text = template.format(list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name}
            ast = f"length_linked_list({list_name})"
            output = f"Length of linked list {list_name}"
        
        elif intent == "LinkedListInsert":
            element = random.randint(1, 100)
            list_name = random.choice(["list1", "list2"])
            position = random.randint(0, 5)
            input_text = template.format(element, list_name, position)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"element": element, "list": list_name, "position": position}
            ast = f"insert_linked_list({list_name}, {element}, {position})"
            output = f"Inserted {element} into {list_name} at {position}"
        
        # String Operations
        elif intent in ["StringConcat", "StringContains"]:
            str1 = random.choice(["hello", "world", "test"])
            str2 = random.choice(["world", "ENG", "code"])
            input_text = template.format(str1, str2)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"strings": [str1, str2]}
            ast = {
                "StringConcat": f"concat({str1}, {str2})",
                "StringContains": f"contains({str1}, {str2})"
            }[intent]
            output = {
                "StringConcat": str1 + str2,
                "StringContains": str2 in str1
            }[intent]
        
        elif intent in ["StringLength", "StringUpper", "StringLower"]:
            text = random.choice(["hello", "world", "test"])
            input_text = template.format(text)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"string": text}
            ast = {
                "StringLength": f"length({text})",
                "StringUpper": f"upper({text})",
                "StringLower": f"lower({text})"
            }[intent]
            output = {
                "StringLength": len(text),
                "StringUpper": text.upper(),
                "StringLower": text.lower()
            }[intent]
        
        elif intent == "StringSubstring":
            text = random.choice(["hello", "world", "test"])
            start = random.randint(0, len(text)-2)
            end = random.randint(start+1, len(text))
            input_text = template.format(text, start, end)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"string": text, "start": start, "end": end}
            ast = f"substring({text}, {start}, {end})"
            output = text[start:end]
        
        # Advanced Algorithms
        elif intent == "Fibonacci":
            num = random.randint(1, 10)
            input_text = template.format(num)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"number": num}
            ast = f"fibonacci({num})"
            fib = [0, 1]
            for i in range(2, num + 1):
                fib.append(fib[i-1] + fib[i-2])
            output = fib[num] if num >= 0 else "Error: Negative index"
        
        elif intent == "BinarySearch":
            element = random.randint(1, 100)
            list_name = random.choice(["sorted_list", "data"])
            input_text = template.format(element, list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"element": element, "list": list_name}
            ast = f"binary_search({list_name}, {element})"
            output = f"Searched for {element} in {list_name}"
        
        elif intent in ["GraphDFS", "GraphBFS"]:
            graph_name = random.choice(["graph1", "graph2"])
            start_node = random.randint(1, 10)
            input_text = template.format(graph_name, start_node)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"graph": graph_name, "start": start_node}
            ast = {
                "GraphDFS": f"dfs({graph_name}, {start_node})",
                "GraphBFS": f"bfs({graph_name}, {start_node})"
            }[intent]
            output = {
                "GraphDFS": f"Performed DFS on {graph_name} from {start_node}",
                "GraphBFS": f"Performed BFS on {graph_name} from {start_node}"
            }[intent]
        
        elif intent in ["SortQuick", "SortMerge"]:
            list_name = random.choice(["my_list", "data"])
            input_text = template.format(list_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"list": list_name}
            ast = {
                "SortQuick": f"quicksort({list_name})",
                "SortMerge": f"mergesort({list_name})"
            }[intent]
            output = {
                "SortQuick": f"Sorted {list_name} with quicksort",
                "SortMerge": f"Sorted {list_name} with mergesort"
            }[intent]
        
        # Error Handling
        elif intent == "TryCatch":
            action = random.choice(["divide(10, 0)", "access_list(my_list, 100)"])
            catch_action = random.choice(["print('Error')", "return(-1)"])
            input_text = template.format(action, catch_action)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"action": action, "catch": catch_action}
            ast = f"try_catch({action}, {catch_action})"
            output = f"Tried {action}, caught with {catch_action}"
        
        # Advanced Data Structures
        elif intent in ["HashSetAdd", "HashSetContains"]:
            element = random.randint(1, 100)
            set_name = random.choice(["set1", "set2"])
            input_text = template.format(element, set_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"element": element, "set": set_name}
            ast = {
                "HashSetAdd": f"add_set({set_name}, {element})",
                "HashSetContains": f"contains_set({set_name}, {element})"
            }[intent]
            output = {
                "HashSetAdd": f"Added {element} to {set_name}",
                "HashSetContains": f"Checked if {set_name} contains {element}"
            }[intent]
        
        elif intent in ["StackPush", "StackPop"]:
            element = random.randint(1, 100) if intent == "StackPush" else None
            stack_name = random.choice(["stack1", "stack2"])
            input_text = template.format(element, stack_name) if intent == "StackPush" else template.format(stack_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"stack": stack_name} if intent == "StackPop" else {"element": element, "stack": stack_name}
            ast = {
                "StackPush": f"push({stack_name}, {element})",
                "StackPop": f"pop({stack_name})"
            }[intent]
            output = {
                "StackPush": f"Pushed {element} to {stack_name}",
                "StackPop": f"Popped from {stack_name}"
            }[intent]
        
        elif intent in ["QueueEnqueue", "QueueDequeue"]:
            element = random.randint(1, 100) if intent == "QueueEnqueue" else None
            queue_name = random.choice(["queue1", "queue2"])
            input_text = template.format(element, queue_name) if intent == "QueueEnqueue" else template.format(queue_name)
            tokenized = [f"{token.text}:{token.pos_}" for token in nlp(input_text.lower())]
            parameters = {"queue": queue_name} if intent == "QueueDequeue" else {"element": element, "queue": queue_name}
            ast = {
                "QueueEnqueue": f"enqueue({queue_name}, {element})",
                "QueueDequeue": f"dequeue({queue_name})"
            }[intent]
            output = {
                "QueueEnqueue": f"Enqueued {element} to {queue_name}",
                "QueueDequeue": f"Dequeued from {queue_name}"
            }[intent]
        
        # Augment input for variety
        augmented_input = augment_text(input_text) if random.random() < 0.5 else input_text
        
        examples.append({
            "input": augmented_input,
            "tokenized": tokenized,
            "intent": intent,
            "parameters": json.dumps(parameters),
            "ast": ast,
            "output": output
        })
    
    return examples

def save_to_parquet(examples, output_path):
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    df = pd.DataFrame(examples)
    df.to_parquet(output_path, index=False)
    print(f"Saved {len(examples)} examples to {output_path}")

if __name__ == "__main__":
    examples = generate_examples(90000)
    save_to_parquet(examples, "data/processed/synthetic_data.parquet")
    save_to_parquet(examples[:1000], "data/synthetic/synthetic_data_sample.csv")
```
---
**Notes on `generate_data.py`**:
- **Artifact ID**: Kept the same (`48f052ac-bf61-4559-8a3c-9d9f0e3b5178`) since this is an update to the previous `generate_data.py`.
- **Templates**: Each of the 72 intents has 10–15 phrasings, covering simple to advanced English (e.g., “Add 4 and 5” to “Compute the summation of 4 with 5”).
- **Generation**: Creates 90,000 examples, with augmentation applied to 50% of inputs for variety.
- **Outputs**: Saves to `data/processed/synthetic_data.parquet` and a 1,000-example sample to `data/synthetic/synthetic_data_sample.csv`.
- **Logic**: Each intent generates appropriate `input`, `tokenized`, `intent`, `parameters`, `ast`, and `output` fields, tailored to the operation (e.g., arithmetic, loops, data structures).

---

### Remaining Dataset-Related Files

Below are the complete codes for the other dataset-related files: `data_augmentation.py`, `convert_manual_data.py`, and `merge_data.py`. These ensure the dataset supports varied English phrasings and combines synthetic and manual data effectively.

**1. src/utils/data_augmentation.py**
```python
import spacy
import nltk
from nltk.corpus import wordnet
import random

try:
    nltk.data.find('wordnet')
except LookupError:
    nltk.download('wordnet')

nlp = spacy.load("en_core_web_sm")

def get_synonyms(word):
    synonyms = set()
    for syn in wordnet.synsets(word):
        for lemma in syn.lemmas():
            synonym = lemma.name().lower()
            if synonym != word and "_" not in synonym:
                synonyms.add(synonym)
    return list(synonyms)[:5]

def augment_text(text, probability=0.4):
    doc = nlp(text.lower())
    words = [token.text for token in doc]
    for i, word in enumerate(words):
        if random.random() < probability and not word.isdigit():
            synonyms = get_synonyms(word)
            if synonyms:
                words[i] = random.choice(synonyms)
    
    # Structural variations for advanced English
    if random.random() < 0.3:
        if any(w in text.lower() for w in ["add", "sum"]):
            words = f"Please compute the sum of {words[1]} and {words[-1]}".split()
        elif any(w in text.lower() for w in ["print", "display", "show"]):
            words = f"Kindly output the value {words[-1]}".split()
        elif "even" in text.lower():
            words = f"Aggregate all even integers from {words[words.index('from')+1]} to {words[-1]}".split()
        elif "odd" in text.lower():
            words = f"Aggregate all odd integers from {words[words.index('from')+1]} to {words[-1]}".split()
        elif any(w in text.lower() for w in ["factorial", "product"]):
            words = f"Evaluate the product of integers up to {words[-1]}".split()
        elif any(w in text.lower() for w in ["max", "maximum", "larger"]):
            words = f"Determine the greatest value between {words[words.index('of')+1]} and {words[-1]}".split()
        elif any(w in text.lower() for w in ["min", "minimum", "smaller"]):
            words = f"Determine the smallest value between {words[words.index('of')+1]} and {words[-1]}".split()
        elif any(w in text.lower() for w in ["greater", "more", "larger"]):
            words = f"Verify if {words[1]} surpasses {words[-1]}".split()
        elif any(w in text.lower() for w in ["less", "smaller", "below"]):
            words = f"Verify if {words[1]} is below {words[-1]}".split()
    
    return " ".join(words)

def augment_example(example):
    augmented_input = augment_text(example["input"])
    return {
        **example,
        "input": augmented_input,
        "tokenized": [f"{token.text}:{token.pos_}" for token in nlp(augmented_input.lower())]
    }

if __name__ == "__main__":
    example = {
        "input": "Sum even numbers from 1 to 10",
        "intent": "SumEvens",
        "parameters": '{"range": [1, 10], "filter": "even"}',
        "ast": "sum(filter(even, range(1, 10)))",
        "output": 30
    }
    augmented = augment_example(example)
    print("Original:", example["input"])
    print("Augmented:", augmented["input"])
```

**Notes**:
- **Artifact ID**: Kept the same (`d62ad8cd-d986-4ac0-bcaf-3ce5c2fce2a7`) as it’s an update.
- **Changes**: Added structural variations for more intents (e.g., `factorial`, `max`, `min`, `greater`, `less`) to support advanced English phrasings.
- **Purpose**: Generates varied phrasings (e.g., “Add 4 and 5” → “Please compute the sum of 4 and 5”) to train a flexible model.

**2. scripts/convert_manual_data.py**
```python
import pandas as pd
import json
import spacy
import os

nlp = spacy.load("en_core_web_sm")

def process_manual_data(input_csv, output_parquet):
    df = pd.read_csv(input_csv)
    df["tokenized"] = df["input"].apply(lambda x: [f"{token.text}:{token.pos_}" for token in nlp(x.lower())])
    df["parameters"] = df["parameters"].apply(lambda x: json.dumps(eval(x)) if isinstance(x, str) else json.dumps(x))
    df.to_parquet(output_parquet, index=False)
    print(f"Converted {len(df)} manual examples to {output_parquet}")

if __name__ == "__main__":
    input_csv = "data/raw/manual_data.csv"
    output_parquet = "data/processed/manual_data.parquet"
    process_manual_data(input_csv, output_parquet)
```

**Notes**:
- **Artifact ID**: Kept the same (`9ef7ecce-dff9-4e36-bfed-f4fb5fb95b00`) as it’s unchanged.
- **Purpose**: Converts `data/raw/manual_data.csv` (10,000 manual examples) to `data/processed/manual_data.parquet` with tokenized inputs.

**3. scripts/merge_data.py**
```python
import pandas as pd
import os

def merge_datasets(synthetic_path, manual_path, output_path):
    synthetic_df = pd.read_parquet(synthetic_path)
    manual_df = pd.read_parquet(manual_path)
    final_df = pd.concat([synthetic_df, manual_df], ignore_index=True)
    final_df = final_df.drop_duplicates(subset=["input"])
    if final_df.isnull().any().any():
        print("Warning: Missing values found in dataset")
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    final_df.to_parquet(output_path, index=False)
    print(f"Merged {len(final_df)} examples into {output_path}")

if __name__ == "__main__":
    merge_datasets(
        "data/processed/synthetic_data.parquet",
        "data/processed/manual_data.parquet",
        "data/processed/final_data.parquet"
    )
```

**Notes**:
- **Artifact ID**: Kept the same (`cd336720-b318-4f03-8e93-4b6efb32fde7`) as it’s unchanged.
- **Purpose**: Combines synthetic and manual data into `data/processed/final_data.parquet`, removing duplicates.

---

### VS Code PowerShell Commands

Here are the exact commands to execute the dataset creation process in VS Code’s PowerShell terminal, following the 7-day plan:

**Day 1: Set Up Environment**
```powershell
# Navigate to project directory
cd .\eng
# Install dependencies
pip install pandas pyarrow spacy nltk transformers
python -m spacy download en_core_web_sm
python -m nltk.downloader wordnet
# Create directories
mkdir data\raw data\processed data\synthetic src\nlu src\executor src\memory src\utils scripts tests config
# Create empty files (optional, for full structure)
New-Item -ItemType File -Path src\nlu\model.py, src\nlu\tokenizer.py, src\nlu\trainer.py, src\executor\executor.rs, src\executor\ir.rs, src\memory\memory.py, src\memory\feedback.py, src\ui\ui.py, src\utils\logging.py, scripts\train.py, scripts\evaluate.py, tests\test_nlu.py, tests\test_executor.py, tests\test_integration.py, config\model_config.yaml, config\training_config.yaml, Cargo.toml, pyproject.toml, README.md
```

**Day 2: Generate Synthetic Data**
```powershell
python .\scripts\generate_data.py
```

**Day 3: Test Data Augmentation**
```powershell
python .\src\utils\data_augmentation.py
```

**Day 4–6: Convert Manual Data**
```powershell
# Run after adding manual examples to data/raw/manual_data.csv
python .\scripts\convert_manual_data.py
```

**Day 7: Merge Data**
```powershell
python .\scripts\merge_data.py
```

---

### Manual Data Creation (data/raw/manual_data.csv)

You’ll need to create `data/raw/manual_data.csv` with 10,000 examples during Days 4–6. Here’s an example format and sample entries:

**Format**:
```csv
input,intent,parameters,ast,output
"Compute the summation of 4 with 5","Add","{\"numbers\": [4, 5]}","add(4, 5)","9"
"Kindly output the value 'hello world'","Print","{\"text\": \"hello world\"}","print('hello world')","Printed: hello world"
"Evaluate factorial for 5","Factorial","{\"number\": 5}","factorial(5)","120"
"Check if 6 is even","IsEven","{\"number\": 6}","is_even(6)","True"
"Perform binary search for 7 in sorted_list","BinarySearch","{\"element\": 7, \"list\": \"sorted_list\"}","binary_search(sorted_list, 7)","Searched for 7 in sorted_list"
```

**Guidelines**:
- **Quantity**: 10,000 examples, split as 1,500 simple (e.g., “Add 4 and 5”), 1,000 medium (e.g., “Calculate 4 + 5”), 500 complex (e.g., “Compute the summation of 4 with 5”) per day.
- **Intents**: Cover all 72 intents, with ~140 examples per intent (10,000 ÷ 72).
- **Advanced English**: Include complex phrasings (e.g., “Determine the greatest common divisor of 12 and 18” for `GCD`).
- **Tools**: Use a spreadsheet or text editor to write the CSV, ensuring no missing fields.

---

### Why This Works

- **Comprehensive Intents**: The 72 intents cover all standard programming constructs (arithmetic, loops, data structures) and advanced features (e.g., `GCD`, `IsEven`, `GraphDFS`), making ENG more powerful than Java/Python.
- **Varied Templates**: 10–15 phrasings per intent ensure the model learns diverse English inputs, from simple (“Add 4 and 5”) to advanced (“Compute the summation of 4 with 5”).
- **Augmentation**: `data_augmentation.py` generates variations (e.g., “Sum 4 and 5” → “Please compute the sum of 4 and 5”), reducing the need to predefine every phrasing.
- **Manual Data**: 10,000 examples add real-world complexity, ensuring the model handles advanced English.
- **Model Flexibility**: The dataset is designed for a transformer model (like BERT, trained later) to generalize across phrasings, understanding “Evaluate the sum of 4 and 5” as `Add` without exact matches.

**Real-World Example**:
Think of teaching a robot chef to cook any dish. You give it 90,000 recipe variations from templates (e.g., “Bake a {flavor} cake”) and 10,000 unique recipes written by hand. The robot learns to cook a “chocolate cake” even if someone says “Prepare a cocoa-flavored pastry.” ENG’s dataset works the same way for programming.

---

### Next Steps

- **Run Commands**: Execute the PowerShell commands above to set up and generate data.
- **Manual Data**: Spend Days 4–6 writing `data/raw/manual_data.csv` with 10,000 examples, ensuring all intents are covered.
- **Review**: Check `data/synthetic/synthetic_data_sample.csv` for quality.
- **Future**: After the dataset is ready, use `scripts/train.py` (not provided this week) to train a transformer model on `data/processed/final_data.parquet`.

If you want to add more intents (e.g., `MatrixMultiply`, `TreeTraversal`) or remove any (e.g., `GraphBFS`), let me know, and I can update `generate_data.py`. I can also help with sample manual data entries or debugging scripts. Just say the word!
