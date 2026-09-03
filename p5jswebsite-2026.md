## The Issue
[Issue #432 ](https://github.com/processing/p5.js-website/issues/432), within the [p5.js-website](https://github.com/processing/p5.js-website) repo is asking for the implementation of a fully functional, updated downloadable reference ZIP of p5.JS that can be available for offline use. 
A downloadable reference ZIP matters because it makes p5.JS easily accessible for users who don't have reliable/consistent wifi or aren't able to access wifi at that time. For example, someone wanting to access p5.js on an airplane, or during a power outage.
- release-workflow-v2.yml: This GitHub Actions workflow automates the p5.js release process. It   builds the library, runs tests, updates the website's generated reference files, and           prepares  the project for deployment. It is relevant to the offline reference because it       shows   where an  offline reference generation or packaging step could be added in the         future as part    of the     automated release process.

- src/scripts/builders/reference.ts: This script builds the website's reference documentation. It goes through the p5.js reference data, converts each class, method, property, and constant into MDX, fixes links, organizes the pages into the correct folders, and saves them under src/content/reference/en/. It is important for the offline reference because it creates the documentation content that Astro later turns into HTML pages.
- `npm run build:reference` — build completed but threw this error twice: `Error modifying absolute path in preprocessor: Error: ENOENT: no such file or directory, open '.../p5.sound.js/docs/preprocessor.js'`
- `npm run build:search` — no errors, but several locale folders are missing: `localeDir src/content/events/es does not exist. Skipping...` (same pattern for people/tutorials/libraries across es, hi, ko, zh-Hans)

Note: Despite these warnings and preprocessor errors, the build process completed successfully. 

## Documentation Pipeline Overview
- The pipeline begins with the p5.js source documentation, which is processed into structured files such as data.json and MDX reference pages. Astro then uses those generated files to build the static website, while GitHub Actions automates testing, builds, and deployment to keep the process consistent.
- Diagram for New p5.js Release Process:
```mermaid
flowchart LR
        A["New p5.js 2.x tag is released"]
        A --> B["Release process begins"]
        B --> C["Check that everything works:<br>-run tests<br>-run build"]
        C --> D["Prepare release files:<br>-gather the files from the build process<br>-create a zip archive of the built p5.js files"]
        D --> E["Release p5.js:<br>-create GitHub release<br>-publish package to NPM"]
        E --> F{"Is this a prerelease?"}
        F -- Yes --> G["Publish under beta tag in NPM"]
        G --> I["Done"]
        F -- No --> H["Clone the p5.js website"]
        H --> J["Update p5.js website:<br>-rebuild website version and documentation pages<br>-commit updated website files<br>-push update to website repository"]
        J --> K["Done"]
```
Let's say we wanted to change the documentation of the function `box()`. The end to end process of making a change in the offline reference page goes as follows: find the .js file in the p5.js repository that contains the data you want to change, in our case, the filepath `src/webgl/3d_primitives.js`. After you make your change, if you have npm installed, run `npm run docs` to update the `data.json` file that will be read by the p5.js-website repository. Commit the changes before you move over to the website repository, and with the repositories in the same location, run `npm run custom:dev ../p5.js#main` to connect the two. To generate the local reference, use `npm run build` and open the local host link it gives you when it's finished. This should pull up an offline version of the reference page, and from there, you just navigate to `box()` to check if your change went through.

Additional testing helped us better understand the documentation pipeline. We confirmed that the p5.js website does not render the reference directly from data.json. Running npm run docs in the p5.js repository updates the reference data, which is then processed by the p5.js-website repository into MDX reference pages. Astro uses those generated MDX files to build the final website. We verified this process by making a test documentation change in p5.js and tracing it through the pipeline until the change appeared on the locally generated reference website.

## Investigation Tracks
To break Issue #432 into manageable parts, we divided the research into
three investigation tracks. Each team focused on a different question:
where in the build process the offline reference should be created,
how it should be generated and packaged, and what files should be included.
Together, these investigations helped us explore the problem from the build,
packaging, and content perspectives.
### _Where_ in the build process do we build the offline reference?
-  The scope of this technical investigation is in the different workflows, actions, and artifacts that are used to generate different aspects of the project. We mainly used the *New p5.js 2.x release* workflow based in the *release-workflow-v2.yml* file that creates the version release notes, along with different tutorials about YAML files, as a reference and guide for our tests.  
-  We investigated multiple options for where the offline reference could be generated during the p5.js release process. We explored three options in particular. We explored a cross-repository workflow concept using repository_dispatch, where we would include a new step in the **release-workflow-v2.yml** file that would then trigger a new workflow in the p5.js-website repo that would then generate the offline reference. We then explored only editing the **release-workflow-v2.yml** file by including a new step that would either upload the offline reference as a workflow artifact or a release asset. 
- Using repository dispatch as the creation of the zip file is a dead end as it requires a PAT. Another dead end we reached was publishing it as a workflow artifact because it expires after a while. Workflow dispatch can be explored further in the creation of the ZIP. For publishing, two routes could still be explored: publishing it as a release asset, and publishing it to its own storage space( e.g.,cloudflare).
- Initial Experiment done by our team was using a 'dummy' folder of tutorial images from the public/images/tutorials/ directory and creating a workflow that zips the file as a workflow artifact. The workflow navigated to the directory using the working-depository keyword and the file was then zipped using command line zip command. The zip file was then uploaded as a workflow artifact using the upload-artifact GitHub action. 
- Code Snippet:
  Plan is to implement the new step for zipping the offline reference after the website is built in the **release-workflow-v2.yml** file. 
  ```yaml
  - name: Updated website files
        if: ${{ steps.semver.outputs.is-prerelease != 'true' }}
        run: |
          cd website
          npm install
          npm run build:p5-version
          npm run build:contributor-docs
          npm run build:contributors
          npm run build:reference
          npm run build:search

         ##(Add new step here)##

  ```
  
### _How_ do we build the offline reference?
- Our investigation focused on the packaging stage of the documentation pipeline. Rather than changing how documentation is generated, we researched how the completed reference files could be packaged into a downloadable offline artifact while remaining separate from the existing build process.

- We researched how the offline ZIP currently works and found that generated site files can break when opened directly without a server because of link and search issues.

- We created a Python extraction script using Beautiful Soup that reads the HTML reference pages generated by Astro inside `dist/reference/` and searches each page for `<main id="main-content">` to isolate the main documentation content.

- The extracted documentation is saved as new HTML files inside a separate `extracted-reference/` folder, leaving the original Astro build unchanged. During testing, the script successfully processed **899 reference pages with 0 pages skipped**.

- The extraction process also copied supporting asset folders such as `_astro`, `assets`, `fonts`, `images`, and `search-indices` so the files needed by the reference could be included for later testing.

- The extracted reference was designed to move through sanitization and validation before reaching the packaging stage. The packaging prototype was designed to verify the expected documentation, generate a manifest of the package contents, produce build/validation reports, measure artifact size, and create a ZIP archive.

- This resulted in our proposed workflow:

  **`Astro Build → Beautiful Soup Extraction → Sanitization/Validation → Packaging → Offline ZIP`**

- While the individual stages were successfully prototyped, additional integration and testing are still needed to create a fully functional offline reference, particularly around offline links and navigation.

Further offline testing showed that some links that work normally when the reference is served as a website do not work when the generated files are opened directly from a local folder. The generated site uses web-style routes, but an offline ZIP needs those routes to correctly resolve to local files such as HTML pages. This caused some pages and links to break when testing the reference completely offline and became one of the reasons we investigated alternative packaging approaches instead of simply ZIPing the generated Astro output as-is.
  
  Code snippets:  
  Reads an HTML page generated by Astro and loads it into Beautiful Soup so the page can be searched and modified.
  ```python
  html = source_file.read_text(encoding="utf-8")
  soup = BeautifulSoup(html, "html.parser")
  ```
  This part extracts documentation into a separate folder without changing the original build. It is then passed to the next stages for sanitization and packaging.
  ```python
  output_file.write_text(extracted_html, encoding="utf-8")
  ```

We also investigated Service Workers as another possible offline approach. Service Workers can cache website files so previously loaded resources can remain available without an internet connection. However, this approach is better suited for adding offline capabilities to a website than creating the fully downloadable, self-contained ZIP requested by Issue #432. Because of that difference, we did not consider Service Workers the strongest option for the final offline reference package.

### _What files_ should be in the offline reference?
- Our approach to researching solutins for issue 432 was to review the oldest working version of the zipped, offline-downloadable reference file to help decide what needs to be in the offline reference doc. We spent time picking apart and experimenting with the old reference page, figuring out what made it work, which files were vital to the page's functionality, what was expendable, and why it had to be zipped.
- We focused on reverse-engineering the older offline reference ZIP, determining which files are essential to the functionality of the website. Testing which files are essential by removing assets and analyzing how search, examples, styling and navigation behave offline. Our team experimented with wget (a command‑line tool that downloads entire websites for offline use) as an alternative to Astro, compared file structures and sizes, while investigating link behavior and MDX portability. Our ongoing work includes prototyping a minimum offline reference.

Current status of each investigation
- What we found so far shows that getting the reference to work offline is definitely possible, but the wget version still has a few issues. @kameron-ctrl was able to open it and run the examples offline, but search still sends you back to the online reference. He also tested compressing small.mp4 from 383,631 bytes to 120,792 bytes and beat.mp3 from 254,118 bytes to 95,339 bytes. Since removing search barely changed the file size, we think it makes more sense to focus on the bigger media files instead of taking away a useful feature than possibly breaking something else. Also found the ffmpeg could be used to compress all the assets and save 90% of the space it takes up. 

#### FFmpeg Media Compression Testing

- To further explore the media compression mentioned above, we used FFmpeg to test five different compression methods across 42 assets. The original assets totaled approximately 6.40 MB, allowing us to compare how different formats affected both file size and functionality.

- WebP reduced the tested assets to approximately 1.42 MB, a 78% reduction, while still being suitable for the image assets we tested. Some video codecs achieved greater size reductions, but caused problems with PNG transparency.

- These results showed that media compression has potential for reducing the size of the offline reference, but the format used matters. Further testing would be needed to make sure compression does not negatively affect transparency, compatibility, or how assets are referenced by the offline pages.

Code Snippets :  
```python 
wget --mirror --page-requisites --adjust-extension --convert-links --no-parent --execute robots=off --user-agent="Mozilla/5.0" http://p5js.org/reference/
```
This command downloads the full reference site—including all required assets—so our team can compare Astro’s build output with a fully scraped offline version
 
## Findings and recommendations

We considered integrating packaging directly into the documentation build process or keeping it as a separate step. We recommend a separate packaging stage because it is easier to maintain, test, and update without affecting the existing documentation generation pipeline, though it does require a prepared set of files before packaging can begin.

## Testing and verified findings:
- Testing of the archived offline reference showed that reference pages, examples, and search could function without an internet connection, although some features behaved differently depending on the version of the offline reference being tested.
- Removing search required removing five files but only reduced the reference size by approximately 2.8 MB. Because search is useful and removing it produced a relatively small reduction, larger media assets were identified as a better target for reducing the ZIP size.
- Media compression was tested using five different methods with FFmpeg. Across 42 tested assets, the original files totaled approximately 6.40 MB. WebP produced the best overall result in this testing, reducing the files to approximately 1.42 MB, or about a 78% reduction.
- Compression still introduces tradeoffs involving file formats, transparency, and compatibility, so these need to be considered before selecting a final compression method.

## A few possible paths forward:
  - modify the release workflow in p5.js to use wget on the reference section of the generated website
  - modify the release workflow in p5.js to use beautiful soup on the reference section of the generated website
  - modify the release workflow in p5.js to zip a folder (any folder) -> then: test automated upload of zip to an external storage website like cloudflare (is it possible?)
  - maybe: compress entire assets folder with FFMPEG (can this be done with a script? how could filenames be preserved for assets to load without modifying the html?)
  - maybe: implement bleach library for HTML sanitization (used to sanitize links in offline reference)

## Recommendations and next steps:
- Prioritize maintaining a functional offline reference instead of removing useful features solely to reduce file size.
- Keep essential assets and working internal links. External links that require internet access can remain but should be clearly identified as online links.
- Search does not appear to be a major source of file size, so media compression should be prioritized before removing search.
- Examples are useful to users but increase the ZIP size, so their inclusion should be evaluated while developing the minimum offline reference.
- Finalize the list of essential files and prototype a minimum offline reference using only those files.
- Continue comparing generation and packaging methods, including Astro and wget, to determine the most stable long-term approach.
- Determine where the completed ZIP should be hosted and how it can be automatically published as part of the release process.
- Rather than selecting an arbitrary ZIP size limit, determine a reasonable target based on the smallest package that preserves the essential functionality of the offline reference.

Downloadable PDF of our research slides with additional information on our Issue #432 research: [PDF](https://export-download.canva.com/JUhGY/DAHRWRJUhGY/901/0-2333377588284187085.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAQYCGKMUH5AO7UJ26%2F20260817%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20260817T131431Z&X-Amz-Expires=9011&X-Amz-Signature=fa2294edfd1533daab5e354623775725cd3cc091f22e9105c775d1541d58f0f3&X-Amz-SignedHeaders=host%3Bx-amz-expected-bucket-owner&response-content-disposition=attachment%3B%20filename%2A%3DUTF-8%27%27CATALYST.pdf&response-expires=Mon%2C%2017%20Aug%202026%2015%3A44%3A42%20GMT)
