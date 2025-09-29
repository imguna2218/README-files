Of course. Here are the `curl` commands for Java, Python, C, and C++ for both the `/execute` and `/execute/batch` routes, formatted for a Linux/macOS terminal.

-----

## Clear the Cache 
```bash
docker exec -it evalx_redis redis-cli FLUSHALL
```
----
## GET request for submissions
```bash
curl http://localhost:3000/submissions/
```

## ☕ Java (java11)

### /execute

```bash
curl -X POST http://localhost:3000/execute \
-H "Content-Type: application/json" \
-d '{
  "language": "java11",
  "version": "11",
  "code": "import java.util.Scanner;\npublic class Main {\n    public static void main(String[] args) {\n        Scanner scanner = new Scanner(System.in);\n        String name = scanner.nextLine();\n        System.out.println(\"Hello, \" + name);\n    }\n}",
  "stdin": "World"
}'
```

### /execute/batch

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
  }
]'
```

-----

## 🐍 Python (3.9)

### /execute

```bash
curl -X POST http://localhost:3000/execute \
-H "Content-Type: application/json" \
-d '{
  "language": "python",
  "version": "3.9",
  "code": "name = input()\nprint(f\"Hello, {name}\")",
  "stdin": "World"
}'
```

### /execute/batch

```bash
curl -X POST http://localhost:3000/execute/batch \
-H "Content-Type: application/json" \
-d '[
  {
    "language": "python",
    "version": "3.9",
    "code": "name = input()\nprint(f\"Hello, {name}\")",
    "stdin": "Batch1"
  },
  {
    "stdin": "Batch2"
  }
]'
```

-----

## ⚫ C (gcc 11)

### /execute

```bash
curl -X POST http://localhost:3000/execute \
-H "Content-Type: application/json" \
-d '{
  "language": "c",
  "version": "11",
  "code": "#include <stdio.h>\nint main() {\n    char name[100];\n    scanf(\"%s\", name);\n    printf(\"Hello, %s\\n\", name);\n    return 0;\n}",
  "stdin": "World"
}'
```

### /execute/batch

```bash
curl -X POST http://localhost:3000/execute/batch \
-H "Content-Type: application/json" \
-d '[
  {
    "language": "c",
    "version": "11",
    "code": "#include <stdio.h>\nint main() {\n    char name[100];\n    scanf(\"%s\", name);\n    printf(\"Hello, %s\\n\", name);\n    return 0;\n}",
    "stdin": "Batch1"
  },
  {
    "stdin": "Batch2"
  }
]'
```

-----

## ➕ C++ (gcc 11)

### /execute

```bash
curl -X POST http://localhost:3000/execute \
-H "Content-Type: application/json" \
-d '{
  "language": "cpp",
  "version": "11",
  "code": "#include <iostream>\n#include <string>\nint main() {\n    std::string name;\n    std::cin >> name;\n    std::cout << \"Hello, \" << name << std::endl;\n    return 0;\n}",
  "stdin": "World"
}'
```

### /execute/batch

```bash
curl -X POST http://localhost:3000/execute/batch \
-H "Content-Type: application/json" \
-d '[
  {
    "language": "cpp",
    "version": "11",
    "code": "#include <iostream>\n#include <string>\nint main() {\n    std::string name;\n    std::cin >> name;\n    std::cout << \"Hello, \" << name << std::endl;\n    return 0;\n}",
    "stdin": "Batch1"
  },
  {
    "stdin": "Batch22"
  },
  {
    "stdin": "Batcdweh2"
  },
  {
    "stdin": "Batchcds2"
  },
  {
    "stdin": "Batccsh2"
  },
  {
    "stdin": "Batbgch2"
  },
  {
    "stdin": "Batcrfh2"
  },
  {
    "stdin": "Batcjuh2"
  },
  {
    "stdin": "Batcesfh2"
  },
  {
    "stdin": "Basdfgtch2"
  },
  {
    "stdin": "Bandhktch2"
  },
  {
    "stdin": "Batdfmch2"
  },
  {
    "stdin": "Batsdsdfhfgjch2"
  },
  {
    "stdin": "Batsdjhch2"
  },
  {
    "stdin": "Badshsdfhatch2"
  },
  {
    "stdin": "Batasdfhch2"
  },
  {
    "stdin": "Batchafsh2"
  },
  {
    "stdin": "Baasgftch2"
  },
  {
    "stdin": "Batdfgsdfch2"
  }
]'
```
