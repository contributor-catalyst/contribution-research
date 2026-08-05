## The Issue
[Issue #432 ](https://github.com/processing/p5.js-website/issues/432), within the [p5.js-website](https://github.com/processing/p5.js-website) repo is asking for the implementation of a fully functional, updated downloadable reference ZIP of p5.JS that can be available for offline use. 
A downloadable reference ZIP matters because it makes p5.JS easily accessible for users who don't have reliable/consistent wifi or aren't able to access wifi at that time. For example, someone wanting to access p5.js on an airplane, or during a power outage.
- release-workflow-v2.yml: This GitHub Actions workflow automates the p5.js release process. It   builds the library, runs tests, updates the website's generated reference files, and           prepares  the project for deployment. It is relevant to the offline reference because it       shows   where an  offline reference generation or packaging step could be added in the         future as part    of the     automated release process.

- src/scripts/builders/reference.ts: This script builds the website's reference documentation. It goes through the p5.js reference data, converts each class, method, property, and constant into MDX, fixes links, organizes the pages into the correct folders, and saves them under src/content/reference/en/. It is important for the offline reference because it creates the documentation content that Astro later turns into HTML pages.

- [Xavier] - Supporting artifacts, if relevant: error messages, console output, screenshots
- `npm run build:reference` — build completed but threw this error twice: `Error modifying absolute path in preprocessor: Error: ENOENT: no such file or directory, open '.../p5.sound.js/docs/preprocessor.js'`
- `npm run build:search` — no errors, but several locale folders are missing: `localeDir src/content/events/es does not exist. Skipping...` (same pattern for people/tutorials/libraries across es, hi, ko, zh-Hans)

Note: Despite these warnings and preprocessor errors, the build process completed successfully. 

## Documentation Pipeline Overview
- [NIJEL C] - The pipeline begins with the p5.js source documentation, which is processed into structured files such as data.json and MDX reference pages. Astro then uses those generated files to build the static website, while GitHub Actions automates testing, builds, and deployment to keep the process consistent.
- [Amari] - A system diagram: Docstrings → data.json → website build → dist/ → offline artifact, spanning both repos. Mark the three investigation tracks on it and number them so they are easy to refer back to
Let's say we wanted to change the documentation of the function `box()`. The end to end process of making a change in the offline reference page goes as follows: find the .js file in the p5.js repository that contains the data you want to change, in our case, the filepath `src/webgl/3d_primitives.js`. After you make your change, if you have npm installed, run `npm run docs` to update the `data.json` file that will be read by the p5.js-website repository. Commit the changes before you move over to the website repository, and with the repositories in the same location, run `npm run custom:dev ../p5.js#main` to connect the two. To generate the local reference, use `npm run build` and open the local host link it gives you when it's finished. This should pull up an offline version of the reference page, and from there, you just navigate to `box()` to check if your change went through.

## Investigation Tracks (three subsections / one per team)
### _Where_ in the build process do we build the offline reference? (Team: JAA)
JAA's scope of technical investigation is in the different workflows, actions, and artifacts that are used to generate different aspects of the project. We mainly used the *New p5.js 2.x release* workflow based in the *release-workflow-v2.yml* file that creates the version release notes, along with different tutorials about YAML files, as a reference and guide for our tests.  
We investigated multiple options for where the offline reference could be generated during the p5.js release process. We explored three options in particular. We explored a cross-repository workflow concept using repository_dispatch, where we would include a new step in the **release-workflow-v2.yml** file that would then trigger a new workflow in the p5.js-website repo that would then generate the offline reference. We then explored only editing the **release-workflow-v2.yml** file by including a new step that would either upload the offline reference as a workflow artifact or a release asset. 
- Using repository dispatch as the creation of the zip file is a dead end as it requires a PAT. Another dead end we reached was publishing it as a workflow artifact because it expires after a while. Workflow dispatch can be explored further in the creation of the ZIP. For publishing, two routes could still be explored: publishing it as a release asset, and publishing it to its own storage space( e.g.,cloudflare). 
- [Name] - Code snippets with plain-English explanation, where relevant
### _How_ do we build the offline reference? (Team: TeamFive)
- [NIJEL C] - Our investigation focused on the packaging stage of the documentation pipeline. Rather than changing how the documentation is generated, we researched how the completed reference files could be packaged into a downloadable offline artifact while remaining separate from the existing build process.

- Researched how the offline ZIP file currently works and where its process can be improved.
- Found that current site files break when opened directly without a server due to link and search issues.
- Created a Python script using Beautiful Soup to extract only the main reference pages and necessary assets.
- Researched using sanitization tools (like bleach) to ensure extracted files remain safe and secure.
- Built a packaging pipeline that tests the files, keeps the ZIP under 15 MB, and generates the final download.
- Proved that improving the ZIP process as a separate step after the build is the best approach for the team.

- Our investigation has been promising so far. We successfully developed and tested a working packaging prototype, with the remaining work centered on final team decisions such as directory structure, naming conventions, and integration with the other investigation tracks.
- Code snippets:  
  Reads an HTML page generated by Astro and loads it into Beautiful Soup so the page can be searched and modified.
  ```python
  html = source_file.read_text(encoding="utf-8")
  soup = BeautifulSoup(html, "html.parser")
  ```
  This part extracts documentation into a separate folder without changing the original build. It is then passed to the next stages for sanitization and packaging.
  ```python
  output_file.write_text(extracted_html, encoding="utf-8")
  ```
### _What files_ should be in the offline reference? (Team: RawRattlers)
- [Kam] - Our approach to researching solutins for issue 432 was to review the oldest working version of the zipped, offline-downloadable reference file to help decide what needs to be in the offline reference doc. We spent time picking apart and experimenting with the old reference page, figuring out what made it work, which files were vital to the page's functionality, what was expendable, and why it had to be zipped.
We focused on reverse-engineering the older offline reference ZIP, determining which files are essential to the functionality of the website. Testing which files are essential by removing assets and analyzing how search, examples, styling and navigation behave offline. Our team experimented with wget (a command‑line tool that downloads entire websites for offline use) as an alternative to Astro, compared file structures and sizes, while investigating link behavior and MDX portability. Our ongoing work includes prototyping a minimum offline reference.
- [Mariah] - Current status of each investigation (do you think it’s a dead end? or is it a promising direction to explore future?)
What we found so far shows that getting the reference to work offline is definitely possible, but the wget version still has a few issues. @kameron-ctrl was able to open it and run the examples offline, but search still sends you back to the online reference. He also tested compressing small.mp4 from 383,631 bytes to 120,792 bytes and beat.mp3 from 254,118 bytes to 95,339 bytes. Since removing search barely changed the file size, we think it makes more sense to focus on the bigger media files instead of taking away a useful feature than possibly breaking something else. Also found the ffmpeg could be used to compress all the assets and save 90% of the space it takes up. 

- [Kendall] - Code snippets with plain-English explanation, where relevant
 
## Findings and recommendations
- [NIJEL C] - We considered integrating packaging directly into the documentation build process or keeping it as a separate step. We recommend a separate packaging stage because it is easier to maintain, test, and update without affecting the existing documentation generation pipeline, though it does require a prepared set of files before packaging can begin.
- [Mariah] - Anything that we directly verified/tested, like experimental scripts, the size of the generated reference files at present
- [Kam will need help with this] - What we recommend next and why
  - Good place to mention what would be a reasonable size for the .zip
  - For a smaller downloadable zip we must discuss what all we want in the zip. The search feature is expendable especiallly since the filter feature already exists. Also the assets is taking up a great
    amount of storage and could be compressed (ffmpeg works well as a compression tool if you turn photo into single frame video) or could be turned into a version with images and without images. 
- A few possible paths forward:
  - modify the release workflow in p5.js to use wget on the reference section of the generated website
  - modify the release workflow in p5.js to use beautiful soup on the reference section of the generated website
  - modify the release workflow in p5.js to zip a folder (any folder) -> then: test automated upload of zip to an external storage website like cloudflare (is it possible?)
  - maybe: compress entire assets folder with FFMPEG (can this be done with a script? how could filenames be preserved for assets to load without modifying the html?)
  - maybe: implement bleach library for HTML sanitization (used to sanitize links in offline reference)
