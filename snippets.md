Of course. Here are the `curl` commands for Java, Python, C, and C++ for both the `/execute` and `/execute/batch` routes, formatted for a Linux/macOS terminal.

-----

## Clear the Cache 
```bash
# Clears the compilation artifact cache
docker exec evalx_redis redis-cli KEYS "evalx:artifact:*" | xargs -r docker exec evalx_redis redis-cli DEL

# Clears the single-execution cache
docker exec evalx_redis redis-cli KEYS "evalx:exec:*" | xargs -r docker exec evalx_redis redis-cli DEL

# Clears completed job results
docker exec evalx_redis redis-cli KEYS "result:*" | xargs -r docker exec evalx_redis redis-cli DEL
```
----
## GET request for submissions
```bash
curl http://localhost:3000/submissions/
```

## ☕ Java (java24)

### /execute

```bash
curl -X POST http://localhost:3000/execute \
-H "Content-Type: application/json" \
-d '{
    "language": "java21",
    "version": "21",
    "code": "import java.util.Scanner;\npublic class Main {\n    public static void main(String[] args) {\n        Scanner scanner = new Scanner(System.in);\n        String name = scanner.nextLine();\n  System.out.println(\"1st Print Statement !!!! \"); \n      System.out.println(\"Hello, \" + name);\n    }\n}",
    "stdin": "Batch1"
  },
'
```

### /execute/batch

```bash
curl -X POST http://localhost:3000/execute/batch \
-H "Content-Type: application/json" \
-d '[
  {
    "language": "java24",
    "version": "24",
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

```bash
curl http://localhost:3000/submissions/
```

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

```bash
curl http://localhost:3000/submissions/
```

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

```bash
curl http://localhost:3000/submissions/
```

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

-----

## ⚫ Golang (go 1.21)

### /execute

```bash
curl --location 'http://localhost:3000/execute' \
--header 'Content-Type: application/json' \
--data '{
    "language": "go",
    "version": "1.21",
    "code": "package main\n\nimport \"fmt\"\n\nfunc main() {\n    fmt.Println(\"Hello from Go!\")\n}",
    "stdin": ""
}'
```

### /execute/batch

```bash
curl --location 'http://localhost:3000/execute/batch' \
--header 'Content-Type: application/json' \
--data '[
    {
        "language": "go",
        "version": "1.21",
        "code": "package main\n\nimport (\n    \"fmt\"\n    \"io/ioutil\"\n    \"os\"\n)\n\nfunc main() {\n    input, _ := ioutil.ReadAll(os.Stdin)\n    fmt.Printf(\"Input was: %s\", string(input))\n}",
        "stdin": "First test case"
    },
    {
        "stdin": "Second test case"
    },
    {
        "stdin": "Third test case"
    },
    {
        "stdin": "Fourth test case"
    },
    {
        "stdin": "Fifth test case"
    }
]'
```
