```bash
curl -X POST http://localhost:3000/execute/batch \
-H "Content-Type: application/json" \
-d '[
  {
    "language": "java11",
    "version": "11",
    "code": "import java.util.Scanner;\npublic class Main {\n    public static void main(String[] args) {\n        Scanner scanner = new Scanner(System.in);\n        String name = scanner.nextLine();\n        System.out.println(\"Hello, \" + name);\n    }\n}",
    "stdin": "Batch1"
  },
  {
    "stdin": "Batch2"
  },
  {
    "stdin": "Batch3"
  },
  {
    "stdin": "Batch4"
  },
  {
    "stdin": "Batch5"
  },
  {
    "stdin": "Batch6"
  },
  {
    "stdin": "Batch7"
  },
  {
    "stdin": "Batch8"
  },
  {
    "stdin": "Batch9"
  },
  {
    "stdin": "Batch10"
  },
  {
    "stdin": "Batch11"
  },
  {
    "stdin": "Batch12"
  },
  {
    "stdin": "Batch13"
  },
  {
    "stdin": "Batch14"
  },
  {
    "stdin": "Batch15"
  },
  {
    "stdin": "Batch16"
  },
  {
    "stdin": "Batch17"
  },
  {
    "stdin": "Batch18"
  },
  {
    "stdin": "Batch19"
  },
  {
    "stdin": "Batch20"
  }
]'
```
