# This plan is not ready to be executed. It's merely here for future reference. Please ignore

Remind me about utilising something like this in my own folders above code I'll be checking out from work, etc. Not everyone will appreciate my take on these, so I will probably periodically run some of these architectural rules, especially around noting entries that make it to the committed database file. I might personally want to see if any of the committed rows in there are actually required, and hit people up if I have questions. This might also form part of the code review process

- At the very least, I will probably have this form the basis of a system prompt, for each project, as I'll need to specify whether this is part of the codebase or just sitting on top of it locally

e.g. you might want to borrow this and formalise it more

```md
- docs
  - features
    - overview.md
    - <feature>
      - <feature>--<alignments>
        - <timestamp>--<description>.md
  - plans
    - research-and-development--<task-name>
      - <task-name>.md
      - pre-requisites
        - <priority-number>--<pre-requisite-task-name>
          - <pre-requisite-task-name>--<timestamp>.md
      - subtasks
        - <priority-number>--<sub-task-name>
          - <sub-task-name>--<timestamp>.md
```

If overview.md is missing, it must be made first and foremost. In line with applying commits one by one to ensure my understanding of the product and its evolution is solid, when joining a new project, I should make this overview.md file from scratch

If I'm new to a project I should probably have a written record of my own understanding, which will be somehow cross referenced to the code itself I guess. e.g. if I write down my assumptions on a project, etc. the agent can search the codebase and tell me if that understanding is incorrect, and over time, detect deviation of my understanding. This will come in handy when looking after projects for a longer period of time. As I might come back to a project a while later. Ideally, this cross referencing will be done every day, or at least applying a merge one by one and understanding what assumptions, etc. can be challenged. This way I can get a feeling for how a project has evolved, or is evolving, and then I can also update my assumptions, etc. as we go
