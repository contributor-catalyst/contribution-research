## The Issue
[Issue #432 ](https://github.com/processing/p5.js-website/issues/432), within the [p5.js-website](https://github.com/processing/p5.js-website) repo is asking for the implementation of a fully functional, updated downloadable reference ZIP of p5.JS that can be available for offline use. 
A downloadable reference ZIP matters because it makes p5.JS easily accessible for users who don't have reliable/consistent wifi or aren't able to access wifi at that time. For example, someone wanting to access p5.js on an airplane, or during a power outage.
- [Des] - Key files in the codebase, and what each does… for ex: src/scripts/builders/reference.ts, astro.config.mjs, src/globals/p5-version.ts, release-workflow-v2.yml
- [Xavier] - Supporting artifacts, if relevant: error messages, console output, screenshots
 
## Documentation Pipeline Overview
- [NIJEL C] - The pipeline begins with the p5.js source documentation, which is processed into structured files such as data.json and MDX reference pages. Astro then uses those generated files to build the static website, while GitHub Actions automates testing, builds, and deployment to keep the process consistent.
- [Amari] - A system diagram: Docstrings → data.json → website build → dist/ → offline artifact, spanning both repos. Mark the three investigation tracks on it and number them so they are easy to refer back to
- [Amario] - Briefly narrate one specific documentation change end to end: a docstring edit appearing in a rebuilt reference page

## Investigation Tracks (three subsections / one per team)
### _Where_ in the build process do we build the offline reference? (Team: JAA)
- [Amario] - Restate each team’s scope of technical investigation, referring back to the previous diagram
- [Name] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [Jaden] - Using repository dispatch as the creation of the zip file is a dead end as it requires a PAT. Another dead end we reached was publishing it as a workflow artifact because it expires after a while. Workflow dispatch can be explored further in the creation of the ZIP. For publishing, two routes could still be explored: publishing it as a release asset, and publishing it to its own storage space( e.g.,cloudflare). 
- [Name] - Code snippets with plain-English explanation, where relevant
### _How_ do we build the offline reference? (Team: TeamFive)
- [NIJEL C] - Our investigation focused on the packaging stage of the documentation pipeline. Rather than changing how the documentation is generated, we researched how the completed reference files could be packaged into a downloadable offline artifact while remaining separate from the existing build process.
- [Xavier] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [NIJEL C] - Our investigation has been promising so far. We successfully developed and tested a working packaging prototype, with the remaining work centered on final team decisions such as directory structure, naming conventions, and integration with the other investigation tracks.

- [Des] - Code snippets with plain-English explanation, where relevant
### _What files_ should be in the offline reference? (Team: RawRattlers)
- [Kam] - Our approach to researching solutions for issue 432 was to review the oldest working version of the zipped, offline-downloadable reference file to help decide what needs to be in the offline reference doc. We spent time picking apart and experimenting with the old reference page, figuring out what made it work, which files were vital to the page's functionality, what was expendable, and why it had to be zipped.
- [Kendall] - List all self-directed areas of investigation per team: this section is the point! It's the record of our extensive research and testing
- [Mariah] - Current status of each investigation (do you think it’s a dead end? or is it a promising direction to explore future?)
- [Kendall] - Code snippets with plain-English explanation, where relevant
 
## Findings and recommendations
- [NIJEL C] - We considered integrating packaging directly into the documentation build process or keeping it as a separate step. We recommend a separate packaging stage because it is easier to maintain, test, and update without affecting the existing documentation generation pipeline, though it does require a prepared set of files before packaging can begin.
- [Mariah] - Anything that we directly verified/tested, like experimental scripts, the size of the generated reference files at present
- [Kam will need help with this] - What we recommend next and why
  - Good place to mention what would be a reasonable size for the .zip
  - Also a good place to share thoughts on including all images, search functionality, language support, etc.
- [Name] - Suggested three sub-issues for Kit to add, informed by our research
