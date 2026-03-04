# YAML Notes & Cheat Sheet for DevOps 


---

## 1. What is YAML
- YAML = **YAML Ain't Markup Language**
- Human-readable data serialization format
- Widely used for configuration and pipelines
- Key characteristics:
  - Indentation-sensitive (no tabs!)
  - Supports key-value pairs, lists, nested objects, and multi-line strings
  - Optional comments with `#`

---

## 2. Key Concepts

### Key-Value Pairs
- Basic building block of YAML

```yaml
name: MyApp
role: Developer
experience_years: 5
learning: true
```

**Tips:**
- Use spaces, not tabs
- Boolean: `true` / `false`
- Strings with spaces may be quoted: `"like this"` or `'like this'`

### Lists

Two ways to write lists:

**Block Style**

```yaml
tools:
  - Git
  - Terraform
  - Ansible
```

**Inline Style**

```yaml
hobbies: [reading, coding, hiking]
```

> Block style is preferred for readability; inline style is good for short lists.

### Nested Objects / Maps

Hierarchical data with indentation

```yaml
server:
  name: web-server
  ip: 192.168.1.10
  port: 8080

database:
  host: localhost
  name: mydb
  credentials:
    user: admin
    password: secret
```

**Notes:**
- Use consistent spaces (2 per level)
- Tabs break YAML

### Multi-line Strings

**Literal style `|`** – preserves newlines

```yaml
script: |
  echo "Starting server..."
  ./run_server.sh
```

**Folded style `>`** – folds lines into a single line (newlines become spaces)

```yaml
message: >
  This is a long message
  that will be folded into one line
```

- `|` → when formatting matters (scripts, commands)
- `>` → for long paragraphs or folded messages

### Comments

```yaml
# This is a comment
tools:
  - Git  # version control
```

---

## 3. Quick Visual Reference Table

| Concept           | Example                                 |
|------------------|-----------------------------------------|
| Key-Value         | `name: web-server`                     |
| Boolean           | `enabled: true`                        |
| List (block)      | `tools:\n  - Git\n  - Terraform`       |
| List (inline)     | `hobbies: [reading, coding]`           |
| Nested objects    | `server: {name: web, ip: 10.0.0.1}`    |
| Multi-line string | `script: |`                             |
| Folded string     | `message: >\n  One line`               |
| Comment           | `# This is a comment`                  |

---

## 4. Best Practices

- Use spaces, not tabs
- Indent consistently (2 spaces per level recommended)
- Validate YAML files with tools like `yamllint` or online validators
- Choose formatting for readability:
  - Block lists for long items
  - Inline lists for short items
  - `|` for scripts / exact formatting
  - `>` for folded text or long messages
- Keep YAML files clean, consistent, and easy to read