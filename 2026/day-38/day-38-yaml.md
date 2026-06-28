# Day 38 – YAML Basics

## Task

Before writing a single CI/CD pipeline, we need to get comfortable with **YAML** — the language every pipeline is written in.

We will:

- Understand YAML syntax and rules
- Write YAML files by hand
- Validate them

---

## Key-Value Pairs

Create `person.yaml` that describes yourself with:

```bash
About:
  name: Prem
  role: Fullstack Engineer with Devops Experience
  Experience: "1-2 year"
  learning: true
```

---

## Lists

Add to `person.yaml`:

```bash
tools:
    - Docker
    - Gitlab
    - GitHub Actions
    - Linux
    - Jira
hobbies: [Travelling , Dancing , Cinematography , Photography]
```


### *Two ways to write a list in yaml is :*

- Block Style
```bash
fruits:
  - Apple
  - Orange
  - Banana
```

- Flow Style
```bash
fruits: [Apple, Orange, Banana]
```

---

## Nested Objects

Create `server.yaml` that describes a server:

- `server` with nested keys: `name`, `ip`, `port`
- `database` with nested keys: `host`, `name`, `credentials` (nested further: `user`, `password`)

```bash
server:
  name: My-Server
  ip: http://127.0.0.1
  port: 80

database:
  host: mongo
  name: My-Db
  credentials:
    - user: mishra0703
    - password: hello321@@
```

---

## Multi-line Strings

In `server.yaml`, add a `startup_script` field using:

1. The `|` block style (preserves newlines)
2. The `>` fold style (folds into one line)

```bash
startup_script: |  
  echo "Starting my server..."
  systemctl start mongo
  echo "Mongo started , launching app"
  npm start

description: >
  This is a very long sentence
  that we split across multiple lines
  for better readability.
```

- `|` → Every line break kept, each command stays on its own line. Use `|` (block style or literal) when newlines matter , Ex: Actual shell scripts, Multi-line commands, Code blocks, anything we want to run line-by-line 

- `>` → All the line breaks collapsed into spaces, one long string. Use `>` (fold style or folded) when we're writing prose that's just wrapped for readability in the file but should read as one continuous string — descriptions, comments, long messages, documentation fields. 

---

## Spot the Difference

What's wrong with the second one ?

```yaml
# Block 1 - correct
name: devops
tools:
  - docker
  - kubernetes
```

```yaml
# Block 2 - broken
name: devops
tools:
  - docker
    - kubernetes        # Indentation is wrong 
  - kubernetes          # Fixed
```

