---
{"dg-publish":true,"permalink":"/cards/concepts/digital-oceans-writing-guidelines/","tags":["concept"]}
---

[[input\|input]]

- Never use the word "easy" or similar words as it can cause frustrations when they encounter an issue - it is the equivalent of saying "you should know this already"
- Provide all the details necessary to understand and trust the article.
- Practical approach - at the end of article reader should have a usable environment.
- Use second person such as "You will" and use motivational outcomes such as instead of "You wil learn how to install apache" try "In this tutorial, you will install Apache".
- When making a **title** - think carefully about what the reader will accomplish by following your tutorial

## Note Structure 

#### Procedural 
---
Which walk the reader through accomplishing a task step-by-step

- Title (Level 1 heading)
- Introduction (Level 3 heading)
- Prerequisites (Level 2 heading)
- Step 1 — Doing the First Thing (Level 2 heading)
- Step 2 — Doing the Next Thing (Level 2 heading)
- …
- Step n — Doing the Last Thing (Level 2 heading)
- Conclusion (Level 2 heading)
#### Conceptual
---
- Title (Level 1 heading)
- Introduction (Level 3 heading)
- Prerequisites (optional) (Level 2 heading)
- Subtopic 1 (Level 2 heading)
- Subtopic 2 (Level 2 heading)
- …
- Subtopic n (Level 2 heading)
- Conclusion (Level 2 heading)

#### Small Task or quick solutions 
---
- Title (Level 1 heading)
- Introduction paragraph
- Prerequisites (optional) (Level 2 heading)
- Article body
- Conclusion paragraph

https://github.com/do-community/do-article-templates/blob/main/conceptual_tutorial_template.md.txt

## Introduction 

- **What is the tutorial about?** What software is involved and what does each component do (briefly)?
- **Why should the reader learn this topic?** What are the benefits of using this particular software in this configuration? What are some practical reasons why the reader should follow this tutorial?
- **What will the reader do or create in this tutorial?** Are they setting up a server and then testing it? Are they building an app and deploying it? Be specific, as this provides the motivation readers need and gets them excited about the topic.
- **What will the reader have accomplished when they’re done?** What new skills will they have? What will they be able to do that they couldn’t do before?
### Pre-Reqs 
---
- Spell out exactly what the reader should have or should do
- Each point should have a link to be much more **precise**.
- Bullet points pre-reqs

#### And of course what is a tutorial without the Conclusion 
---
- What the reader have accomplished
- What can the reader do next
## Formatting 

**Bold text** should be used for:

- Visible GUI text
- Hostnames and usernames, like **wordpress-1** or **sammy**
- Term lists
- Emphasis when changing context for a command, like switching to a new server or user

_Italics_ should only be used when introducing technical terms

In-line code formatting should be used for:

- Command names, like `unzip`
- Package names, like `mysql-server`
- Optional commands
- File names and paths, like `~/.ssh/authorized_keys`
- Example URLs, like `http://==your_domain==`
- Ports, like `:3000`
- Key presses, which should be in ALL CAPS, like `ENTER`. If keys need to be pressed simultaneously, use a plus symbol (**+**), such as `CTRL+C`.

Code blocks should be used for:

- Commands the reader needs to execute to complete the tutorial
- Files and scripts
- Terminal output
- Interactive dialogues that are in text



