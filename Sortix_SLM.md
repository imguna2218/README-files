# Sortix Model Development Guide

## The Complete Guide to Building Your File Organization SLM

This guide covers everything from dataset creation to a production-ready model that runs efficiently on any computer.

---

## Phase 1: Dataset Design & Generation

### 1.1 Understanding What the Model Needs to Learn

Your Sortix model needs to transform a flat list of file paths into a logical folder hierarchy. This is essentially a **structured prediction task** – given messy input, output organized JSON.

**Example transformation:**

```
Input (messy list):
├── invoice_2025_03_15.pdf
├── DSC_001.jpg
├── vacation_beach.jpg
├── package.json
├── src/App.js
├── src/utils/helpers.js
├── node_modules/ (ignore)
├── project_notes.txt
├── tax_documents_2024.pdf
└── IMG_2022_05.jpg

Output (organized structure):
{
  "Documents": {
    "Invoices": ["invoice_2025_03_15.pdf"],
    "Tax": ["tax_documents_2024.pdf"],
    "Notes": ["project_notes.txt"]
  },
  "Photos": {
    "Vacation": ["vacation_beach.jpg"],
    "2022": ["IMG_2022_05.jpg"],
    "Unsorted": ["DSC_001.jpg"]
  },
  "Code": {
    "Project": {
      "src": {
        "root": ["App.js"],
        "utils": ["helpers.js"]
      },
      "config": ["package.json"]
    }
  }
}
```

### 1.2 Dataset Size: How Many Examples Do You Need?

Based on research into SLM fine-tuning , here's what works:

| Dataset Size | Quality | Training Time | Notes |
|--------------|---------|---------------|-------|
| **500 examples** | Baseline | 15-20 mins | Works for simple patterns |
| **2,000 examples** | Good | 45-60 mins | Covers most common cases |
| **5,000 examples** | Excellent | 2-3 hours | Handles edge cases well |
| **10,000+ examples** | Production-grade | 5-6 hours | May be overkill |

**Recommended target: 3,000-5,000 examples** – this gives you excellent coverage without excessive training time. You can always add more later.

### 1.3 Where to Get the Examples

You have three sources, and you should use ALL of them:

#### Source A: Synthetic Generation (80% of your data)
Use a larger model (Claude, Gemini, GPT) to generate examples. This is the approach validated by research .

**Step-by-step generation process:**

1. **Create seed patterns** – Write 50-100 "template" messy folder structures by hand
2. **Generate variations** – Use this prompt with a large model:

```
Generate 50 variations of messy file lists based on this template:

Template messy list:
- project_report_final_v3.docx
- image_001.jpg
- image_002.jpg
- budget_2024.xlsx
- notes.txt
- screenshot.png

For each variation:
1. Change file names (different projects, dates, numbers)
2. Mix extensions differently
3. Add 2-3 unexpected files
4. Output as JSON array of strings

Make them realistic – like a real Downloads folder or Desktop.
```

3. **Generate the ideal output** – For each messy list, ask:

```
Given this messy file list:
[list from step 2]

Generate the IDEAL organized folder structure as JSON.
Rules:
- Group related files together
- Use logical folder names (Photos, Documents, Code, etc.)
- Preserve important project structure (keep src/ structure)
- Ignore noise (node_modules, .git, .DS_Store)
- Date-based folders for photos when dates exist
- Output ONLY valid JSON, no explanations
```

#### Source B: Real-World Scraping (15% of your data)
Collect actual folder structures from:
- Your own computer (Downloads, Desktop, Documents)
- Open-source GitHub repositories (clone small projects, capture structure)
- Ask friends to share anonymized folder listings
- Public datasets of file systems (search Kaggle for "directory structure")

#### Source C: Hybrid Enhancement (5% of your data)
Take real structures and deliberately mess them up, then use as examples:
- Flatten nested folders
- Scatter related files
- Rename files to inconsistent patterns

### 1.4 Dataset Format

Create a JSONL file (each line is a complete example):

```jsonl
{"instruction": "Organize these files into a logical folder structure. Ignore node_modules, .git, and system files. Preserve important project structure.", "input": ["invoice_2025_03.pdf", "photo.jpg", "src/index.js", "README.md"], "output": {"Documents": {"Invoices": ["invoice_2025_03.pdf"]}, "Photos": ["photo.jpg"], "Project": {"src": ["index.js"], "root": ["README.md"]}}}
{"instruction": "Organize these files...", "input": ["vacation.jpg", "IMG_001.jpg", "budget.xlsx"], "output": {"Photos": {"Vacation": ["vacation.jpg"], "Unsorted": ["IMG_001.jpg"]}, "Documents": {"Finance": ["budget.xlsx"]}}}
```

### 1.5 Quality Validation

Before training, validate your dataset:
- Check for JSON parsing errors
- Ensure variety (different folder names, file types)
- Remove duplicates
- Verify no personally identifiable information

---

## Phase 2: Model Selection

### 2.1 Don't Train From Scratch

**DO NOT train from scratch.** Research shows fine-tuning an existing model is vastly more efficient . Training from scratch requires:
- Millions of examples (not thousands)
- Weeks of training on multiple GPUs
- Deep expertise in architecture design

### 2.2 Best Base Models for Sortix

Based on performance benchmarks for CPU inference , here are your options:

| Model | Parameters | RAM Usage | Speed (TPS) | Quality | Best For |
|-------|------------|-----------|-------------|---------|----------|
| **Qwen2.5-1.5B** | 1.5B | ~3.2 GB | 7-8 TPS | High | Best overall balance |
| **Llama-3.2-1B** | 1B | ~2.1 GB | 9-10 TPS | Good | Maximum speed, lower RAM |
| **Gemma-2-2B** | 2B | ~4.0 GB | 5-6 TPS | Very High | Better quality, more RAM |
| **Phi-3-mini** | 3.8B | ~4.5 GB | 3-4 TPS | Excellent | Quality focus, slower |

**Recommended: Qwen2.5-1.5B** – Excellent balance of quality, speed, and memory footprint. It achieves ~8 tokens/second on consumer CPUs with under 3.5GB RAM .

### 2.3 Quantization: The Secret to Small Footprint

Quantization reduces model size with minimal quality loss :

| Quantization | Size Reduction | RAM | Quality Loss |
|--------------|----------------|-----|--------------|
| FP16 (none) | 0% | ~3.0 GB | None |
| **Q4_K_M (recommended)** | 75% | ~0.9 GB | Minimal |
| Q2_K | 85% | ~0.5 GB | Noticeable |
| Q8 | 50% | ~1.5 GB | Negligible |

**Always use Q4_K_M quantization** – it's the sweet spot for CPU deployment.

---

## Phase 3: Fine-Tuning Process

### 3.1 Setup: Google Colab (Free GPU)

You'll use Unsloth, which makes fine-tuning 2x faster and uses 90% less memory .

**Colab setup:**
1. Go to [Google Colab](https://colab.research.google.com)
2. Select Runtime → Change runtime type → T4 GPU (free)
3. Install Unsloth:

```python
# Cell 1: Install
!pip install unsloth
!pip install torch torchvision torchaudio xformers --index-url https://download.pytorch.org/whl/cu121
```

### 3.2 Load and Quantize Base Model

```python
# Cell 2: Load model
from unsloth import FastLanguageModel
import torch

model_name = "unsloth/Qwen2.5-1.5B-bnb-4bit"

model, tokenizer = FastLanguageModel.from_pretrained(
    name=model_name,
    max_seq_length=2048,  # Enough for 100+ files
    dtype=torch.float16,
    load_in_4bit=True,  # Quantized immediately
)
```

### 3.3 Add LoRA Adapters

LoRA (Low-Rank Adaptation) is what makes fine-tuning efficient. Instead of updating all 1.5B parameters, you update a small subset .

```python
# Cell 3: Add LoRA
model = FastLanguageModel.get_peft_model(
    model,
    r=16,  # Rank – higher = more capacity, 16 is good
    lora_alpha=16,
    lora_dropout=0,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    use_rslora=True,
)
```

### 3.4 Load and Format Dataset

```python
# Cell 4: Load your dataset
from datasets import load_dataset

# Upload your JSONL file to Colab first
dataset = load_dataset('json', data_files='sortix_dataset.jsonl', split='train')

# Format for training
def format_example(example):
    """Convert to chat format"""
    text = f"""<|im_start|>system
You are Sortix, an AI that organizes files into logical folder structures. Output only valid JSON.<|im_end|>
<|im_start|>user
Organize these files: {example['input']}<|im_end|>
<|im_start|>assistant
{example['output']}<|im_end|>"""
    return tokenizer(text, truncation=True, max_length=2048)

dataset = dataset.map(format_example)
```

### 3.5 Hyperparameters (Critical Choices)

Based on research , use these settings:

```python
# Cell 5: Training arguments
from transformers import TrainingArguments
from trl import SFTTrainer

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        warmup_steps=10,
        num_train_epochs=3,  # 3 epochs is plenty
        learning_rate=2e-4,
        fp16=True,
        logging_steps=10,
        output_dir="sortix_output",
        optim="adamw_8bit",
        weight_decay=0.01,
        lr_scheduler_type="cosine",
    ),
)
```

**Why these values:**
- **Epochs = 3**: More than this risks overfitting on 3-5k examples
- **Learning rate = 2e-4**: Standard for fine-tuning
- **Batch size = 2**: Fits in T4 memory with 1.5B model
- **Gradient accumulation = 4**: Simulates batch size 8

### 3.6 Train

```python
# Cell 6: Train
trainer.train()

# Save LoRA adapters (small file)
model.save_pretrained("sortix_lora")
tokenizer.save_pretrained("sortix_lora")
```

### 3.7 Convert to GGUF (for Ollama)

```python
# Cell 7: Merge and export
model = model.merge_and_unload()  # Merge LoRA weights

# Save full model
model.save_pretrained("sortix_full")
tokenizer.save_pretrained("sortix_full")

# Convert to GGUF (download conversion script)
!git clone https://github.com/ggerganov/llama.cpp
!python llama.cpp/convert.py sortix_full --outfile sortix_q4_k_m.gguf --outtype q4_k_m
```

Download the resulting `sortix_q4_k_m.gguf` file (about 900MB).

---

## Phase 4: Testing Your Model

### 4.1 Local Testing with Ollama

Create a `Modelfile`:

```
FROM ./sortix_q4_k_m.gguf

TEMPLATE """<|im_start|>system
You are Sortix, an AI that organizes files into logical folder structures. Output only valid JSON.<|im_end|>
<|im_start|>user
{{ .Prompt }}<|im_end|>
<|im_start|>assistant"""

PARAMETER temperature 0.1  # Low temp for consistent output
PARAMETER top_p 0.9
PARAMETER stop "<|im_end|>"
```

Install and test:
```bash
ollama create sortix -f Modelfile
ollama run sortix "Organize these files: invoice.pdf, photo.jpg, script.py"
```

### 4.2 Test Suite

Create a test set of 100 examples NOT used in training. Test for:

1. **JSON validity** – Does output parse?
2. **Structure quality** – Are folders logical?
3. **Noise handling** – Does it ignore node_modules?
4. **Edge cases** – Empty folders, duplicate names, special characters

```python
# Example test script
test_cases = [
    {"input": ["a.pdf", "b.pdf", "c.jpg"], "expected_folders": ["Documents", "Photos"]},
    {"input": ["src/index.js", "package.json"], "expected_preserves": ["src"]},
]

for case in test_cases:
    response = ollama.chat(model='sortix', messages=[{
        'role': 'user', 
        'content': f"Organize: {case['input']}"
    }])
    # Validate response
```

---

## Phase 5: Integration with Sortix (Rust App)

### 5.1 Model Integration Points

Your Rust app will:

1. **Scan directory** → Get flat list of file paths (relative to root)
2. **Filter** → Remove noise (node_modules, .git, hidden files)
3. **Call model** → Send list to Ollama
4. **Parse JSON** → Get proposed structure
5. **Present to user** → Show tree for editing
6. **Execute moves** → Apply approved changes

### 5.2 Prompt Engineering

The exact prompt format:

```
<|im_start|>system
You are Sortix, an AI file organizer. Analyze the list of files and output a JSON folder structure.

Rules:
- Group similar files together
- Use standard folder names (Documents, Photos, Code, etc.)
- Preserve important paths (keep src/components structure)
- Ignore: node_modules, .git, .DS_Store, Thumbs.db
- For photos with dates, create year/month folders
- Output ONLY the JSON, no explanations
<|im_end|>
<|im_start|>user
Organize these files:
[
  "invoice_2025_03.pdf",
  "vacation_beach.jpg",
  "src/App.js",
  "src/utils/helpers.js",
  "README.md",
  "node_modules/react/index.js",
  "IMG_20220101.jpg"
]
<|im_end|>
<|im_start|>assistant
```

### 5.3 User Edit Feature

The model should output JSON that your UI can render as an editable tree. When user modifies, send the new structure back to model for validation? No – let users override; model is suggestion only.

### 5.4 Commands Generation

**Don't have model generate commands.** Commands are trivial to generate in Rust once you have source → target mapping. Model just provides the target structure.

Example mapping:
```rust
// Model output JSON → commands
for (old_path, new_path) in mapping {
    if old_path != new_path {
        commands.push(format!("mv {} {}", old_path, new_path));
    }
}
```

---

## Phase 6: Performance Optimization

### 6.1 Model Size Targets

| Metric | Target | Why |
|--------|--------|-----|
| RAM usage | < 1.5 GB | Runs on 4GB RAM machines |
| Model file | < 1 GB | Quick download |
| Inference | < 2 seconds | Good UX for 100 files |
| Tokens/sec | > 5 | Feels responsive |

Qwen2.5-1.5B Q4_K_M hits all these .

### 6.2 Caching Strategy

Cache model responses for identical folder structures:
- Hash the file list (sorted) → store in SQLite
- If same list seen before, use cached structure
- Saves repeated API calls

### 6.3 Batch Processing

For huge folders (10,000+ files):
- Split into chunks of 500 files
- Process each chunk separately
- Merge results with conflict resolution
- Shows progress to user

---

## Phase 7: Iteration & Improvement

### 7.1 Collect Feedback Loop

After release, add optional anonymized feedback:

```rust
// If user edits the AI suggestion, log the change
struct Feedback {
    original_list: Vec<String>,
    ai_suggestion: JsonValue,
    user_edit: JsonValue,
    // No personal data
}
```

Periodically use this to create v2 dataset.

### 7.2 When to Retrain

Retrain when:
- You have 1000+ new examples from user edits
- Users complain about specific patterns
- New file types become common

### 7.3 A/B Testing

Test different base models:
- Qwen2.5-1.5B vs Llama-3.2-1B
- Compare speed and quality
- Let users choose "Fast" vs "Smart" mode

---

## Timeline (Realistic)

| Phase | Duration | Output |
|-------|----------|--------|
| Dataset generation | 3-5 days | 3,000-5,000 examples |
| Fine-tuning | 1 day | Trained LoRA adapters |
| Testing | 1 day | Validation results |
| GGUF conversion | 2 hours | Production model |
| Integration | Weekends | Working Sortix |

**Total: 1-2 weeks** for a polished model.

---

## Summary: Your Action Plan

1. **Week 1:** Generate 3,000 synthetic examples using Claude/Gemini
2. **Week 2:** Fine-tune Qwen2.5-1.5B on Colab, convert to GGUF
3. **Week 3:** Test thoroughly, iterate on dataset
4. **Week 4:** Integrate with Rust app, release beta

The model will be:
- ✅ 100% local, private
- ✅ < 1.5 GB RAM
- ✅ Fast on any computer
- ✅ Understands semantics, not just extensions
- ✅ User-editable suggestions

This is **absolutely doable** and will make Sortix genuinely unique in the market.

Want me to dive deeper into any specific phase? The dataset generation is usually the biggest hurdle – I can provide exact prompts and scripts for that if needed.
