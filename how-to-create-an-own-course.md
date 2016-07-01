---
head:
  title: "How to create your own php course on phping.me"
---

# How To Create your own Course

## 1. Create an index.md

The entry point for a course is an ```index.md``` file. Each task in a course is wrapped into a text block that starts with a h2 heading.

### Global Meta
The ```index.md``` has also a global meta block, that defines global meta information about the course. The meta block have to be defined in a YAML structure and put within ```---``` lines, just like this:
```
<pre>
---
  head:
    title: "Some weired course name"
---
</pre>
```
After the global meta block, we just start to write a markdown text.

### Course Structure
There are 3 parts that makes a course complete:
- Introduction
- Tasks
- Conclusion

Introduction, each of the Tasks and Conclusion have to start with a h2 header element. Just like:
<pre>
## Introduction

## Task 1

## Task 2

## Task 3

## Conclusion
</pre>

### Task Structure
Each Task has to have a provided **Solution** block and **task+data** blocks.

#### Solution
Solution block starts with h3 and have to be the very last content of task.
<pre>
...
## Task 1
### Task 1 Header 1
### Task 1 Header 2
### Solution
## Task 2
...
</pre>

#### task+data
To specify task default code and matching data you have to prepare a block like this of type **task+data**:
<pre>
```task+data
default code
---
yaml structure
---

```
</pre>

Matching information have to be defined in YAML structure. There are 3 types of matching:
- match_input
- match_output
- match_file

The structure of a specific matching type is following:
```yaml
<matching_type>
  match:
    - RegExp1
    - RegExp2
    - ...
    - RegExpN
```

Matching type defines whether the task is passed or not. It passing in case each RegExp matches.

## 2. Define composer.json

To provide your course's php backend dependencies, and a course information as well, you have to define a **composer.json** file.
It have to look like this:
```json
{
    "name": "phpingme-courses/<course-name>",
    "description": "Course description",
    "license": "<License>",
    "authors": [
        {
            "name": "Firstname Lastname",
            "email": "email@email.com"
        }
    ],
    "homepage": "http://phping.me/courses/<course-name>",
    "keywords": ["keyword1", "keyword2", "keyword3", "keyword4"],
    "require": {
      "php": ">=7.0",
      "<any vendor>/<you using in a course>": "^<version>"
    }
}
```


## 3. Make a pull request

When you done, make a Pull Request to this repo on branch **drafts**
