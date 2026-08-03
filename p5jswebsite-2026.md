## The Issue
- [Kendall] - What #432 asks for, in plain English
- [Jaden] - A downloadable reference ZIP matters because it makes p5.JS easily accessible for users who don't have reliable/consistent wifi or aren't able to access wifi at that time.
- [Des] - Key files in the codebase, and what each does… for ex: src/scripts/builders/reference.ts, astro.config.mjs, src/globals/p5-version.ts, release-workflow-v2.yml
- [Xavier] - Supporting artifacts, if relevant: error messages, console output, screenshots
 
## Documentation Pipeline Overview
- [NIJEL C] - Astro, GitHub Actions/Workflows, any other pieces, and each one’s role
- [Amari] - A system diagram: Docstrings → data.json → website build → dist/ → offline artifact, spanning both repos. Mark the three investigation tracks on it and number them so they are easy to refer back to
- [Amario] - Briefly narrate one specific documentation change end to end: a docstring edit appearing in a rebuilt reference page

## Investigation Tracks (three subsections / one per team)
### _Where_ in the build process do we build the offline reference? (Team: JAA)
- [Amario] - Restate each team’s scope of technical investigation, referring back to the previous diagram
- [Name] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [Jaden] - Current status of each investigation (do you think it’s a dead end? or is it a promising direction to explore future?)
- [Name] - Code snippets with plain-English explanation, where relevant
### _How_ do we build the offline reference? (Team: TeamFive)
- [NIJEL C] - Restate each team’s scope of technical investigation, referring back to the previous diagram
- [Xavier] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [NIJEL C] - Current status of each investigation (do you think it’s a dead end? or is it a promising direction to explore future?)
- [Des] - Code snippets with plain-English explanation, where relevant
### _What files_ should be in the offline reference? (Team: RawRattlers)
- [Kam] - Restate each team’s scope of technical investigation, referring back to the previous diagram
- [Kendall] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [Mariah] - Current status of each investigation (do you think it’s a dead end? or is it a promising direction to explore future?)
- [Kendall] - Code snippets with plain-English explanation, where relevant
 
## Findings and recommendations
- [NIJEL C] - Summary of options considered, with tradeoffs
- [Mariah] - Anything that we directly verified/tested, like experimental scripts, the size of the generated reference files at present
- [Kam will need help with this] - What we recommend next and why
  - Good place to mention what would be a reasonable size for the .zip
  - Also a good place to share thoughts on including all images, search functionality, language support, etc.
- [Name] - Suggested three sub-issues for Kit to add, informed by our research
