---
{"dg-publish":true,"permalink":"/cards/dots/rules/"}
---

~ [[input\|input]]

**A dedicated note section that gives an overview of how I write and structure my notes.**

If the note covers more than 1 technical mechanism → it must be split.
#### Note Structure
---
`+simmering` (unpublished) folder - notes that is unfinished, cannot be organized yet, or I want to look out in the future.

`cards` (published) folder - holds all topics that I have been studying - each unique topic gets its own dedicated folder i.e cards/active-directory/.

- `cards/topic` has its own image container so I can easily copy and paste the folder to move.

`map-of-contents` (published) folder - holds all notes available or created per topic, i.e every notes cards/active-directory folder will be listed in active-directory.md (note)

`x` (published) folder - holds utilities such as templates, excalidraw, and images.

`z_archive` - notes that I dont want to delete but will never be used.


#### How do I note take?

I follow the [[cards/concepts/Digital Oceans Writing Guidelines\|Digital Oceans Writing Guidelines]] template -- basically creating a note where its both a blog and documentation type note.

For red and blue teaming:

There is a specific template for each of them: [[x/templates/A - Techniques Blue Team v1\|A - Techniques Blue Team v1]] and [[x/templates/A - Techniques  Red Team v1\|A - Techniques  Red Team v1]], the template's purpose is to atomicize each investigation or exploitation steps so readers can view/skim easily with the parent note template is [[x/templates/A  - MOC 2 - TOPIC specific\|A  - MOC 2 - TOPIC specific]].

Here is a sample notes for both red and blue teaming, displaying aliases is **MANDATORY**.

- [[x/Suspicious PowerShell Parent\|DETECT - Endpoint Windows Suspicious PowerShell Parent]]
- [[x/WMI remote exec\|TECH - T1047 - WMI - Remote Process Creation]]

The [[x/templates/A  - MOC 2 - TOPIC specific\|A  - MOC 2 - TOPIC specific]] will not simply hold a list of related notes but instead will hold links of related notes i.e investigation and technique steps with more details.

#### Template Techniques Details
---

- Both start with **Summary** + **How it works** for instant context.
- Both have a middle “hands-on core” (`Steps` for red, `Detection Workflow` for blue).
- Both finish with “Response” and “Related”.
- In the metadata or properties of each technique template it will contain both the following:
	- `mitre_tactic` to link the PARENT tactic.
	- `mitre_technique` the specific technique ID.
	- So in search we can easily filter for the same techniques.

Additionally, I use a [[notes\|AI prompt]] to summarize my notes into key sections under the 'Key Topics' header found on every note - if needed.

**Where do uncategorized but finished notes go but is still connected to my learnings?**

To the `cards/dots` directory, why name `dots`?

"Well when we look at the Obsidian graph view, you can see your notes as tiny dots and these dots can have arms, meaning they are connected with other notes (or dots in this case), the same thing notes found in this directory, they still have a place in this ideaverse they are just waiting so for now they are just dots."