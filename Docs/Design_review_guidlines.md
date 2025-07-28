# Lightweight Design Review Guidelines (v1)

## 1. Purpose

The goal is to create a culture of **upfront lightweight design review** that:

* Encourages developers to think about architectural and future implications before committing code.
* Helps avoid significant **rework, redesign, and refactoring** later.
* Upskills junior developers in design thinking and architectural decisions.
* **Does not** slow down experimentation or become a bureaucratic burden.

⚠️ **Keep it lightweight:**
If it takes more than **20 minutes to write** and **30 minutes to review**, you’re doing it wrong.
(If serious design concerns emerge, schedule a deeper design session separately.)

---

## 2. When to Do a Design Review

Do a design review if your task meets **one or more** of the following:

✅ **New Component or Service**

* Creating a new repo or major component.
* Adding a significant new module that other parts of the system will depend on.

✅ **Architectural or Structural Change**

* Designing or significantly changing:

  * Database schema or data storage format.
  * Message/event structures or APIs (REST/GRPC/Event streams).
  * Filesystem structure used by other services.

✅ **Cross-Component Interaction**

* Introducing or modifying **integration points** (1\:N, N\:M relationships).

✅ **Introducing New Technologies or Dependencies**

* Adding a new framework, major library, or external service.

✅ **Significant Complexity or Risk**

* Work expected to take more than **\~3 days**.
* Clear **uncertainties, assumptions, or performance risks**.

### No Design Review Needed If:

* Bug fixes, small refactors, UI tweaks.
* Internal-only changes with no long-term maintainability impact.

When in doubt → **err on the side of writing one** (it’s quick).

---

## 3. Design Review Template

Keep it **½–1 page**, bullet points are fine.

### Template

#### Problem Statement

1. **Title & Summary**

   * **What is being built?** (1–2 sentences)
   * **Why are we building it?** (high-level purpose / requirements)
   * **Who are we building it for?** (who will be using this? Internal developers? Customer? Someone else?)

2. **Business Context**

   * Relation to other projects (will this thing be used / re-used in or with other projects?)
   * Relation to platform aspirations / other company software (e.g. integrated with Volt to provide financial reporting)

3. **Requirements**

   * performance (e.g. aiming for 80 percent accuracy / respond in seconds)
   * interface (e.g. REST API / proposed methods)
   * stakeholder (e.g. external accountant / internal software engineers / downstream dependencies)
   * user / product (e.g. needs to be able to search by date range / sort by size, etc.)


#### Solution Statement

4. **Interfaces**

   * What interfaces will it expose? (REST, GRPC, Event, API, STDOUT, etc.)
   * Who/what will consume them?

5. **Dependencies & Technologies**

   * What does it depend on?
   * Any **new frameworks or libraries**? Any **removed dependencies?**

6. **Technical Approach (High-Level)**

   * Tools, languages, and data flow (bullets).
   * Include a simple diagram if helpful.

7. **Justification of Technical Approach**

   * Why have you used this approach?
   * What considerations lead to this solution?

8. **Assumptions, Risks & Open Questions**

   * Uncertainties (e.g., performance, upstream dependencies [I need that feature to be working in that other codebase before I start], security).
   * Assumptions that could break the design.

---

## 4. Review Process

### Step 1 – Write

Author drafts the design doc (≤20 min).

### Step 2 – Share

* **Initial:** Create design review github issue in repository, tag reviewers & let them know via email, Slack, Watt message.
* **Design Review Issue Creation** ![UI on github to create Design Review Issue](image.png)
* **Final:** commit to repo (`/docs/design/`).

### Step 3 – Review

* **At least one reviewer** (preferably someone familiar with the affected codebase).
* **Two reviewers** for major components or new repos.

### Step 4 – Discuss

* Quick chat or short email thread (≤30 min).
* If big issues arise, schedule a deeper session.

### Step 5 – Record

* Once agreed, commit the doc into the repo (`docs/design/...`).

---

## 5. Reviewer Checklist

When reviewing, ask:

1. **Purpose & Scope** – Is the problem well understood? Does the design fit the requirements?
2. **Relationships** – Have cross-component impacts been considered?
3. **Interfaces** – Are the interfaces clear, minimal, and aligned with existing conventions?
4. **Dependencies** – Any unnecessary frameworks or tech bloat introduced?
5. **Future Flexibility** – Does the design allow reasonable future changes without huge refactors?
6. **Risks & Assumptions** – Are key uncertainties identified?

---

## 6. Examples

### Example 1 – FLAIR's file based caching mechanism

# Problem Statement

## Summary

Cache for REST API server, scripts should be run though external process (agent running on framework or forked bash process) and store data produced by each script in the cache.

## Business Context

This is intended to be integrated with the agent framework when it is ready to be used, for the scripts in flair scripts to be run by an agent in the cloud by the agent framework and the output to be transferred somehow into an output location when the agent completes.

The intention is to use a similar framework with the agent framework in MAXAD and Neptune.

## Requirements

- Be able to handle structured / semi-structured data, like that from an API.
- Be able to handle variable data which doesn't adhere to a strict schema like that produced by an LLM
- Be able to be easily written to by distributed agent running in the cloud 
- Be able to handle missing data / no-data being returned, be able to handle agent encountering an error in execution, efficiently handle json data nativly without having to transform it
- Be able to wok out path / route to data from script and inputs e.g. company number

# Solution Statement

## Interfaces

The REST API should consume the data in the cache and return it to the client-side, if the data is not available in the cache, the API should trigger the script that produces the data to run and store the data in a predefined cache location.

## Dependencies & Technologies

Removed Prisma (database manager) and SQL (database platform). 

## Technical Approach

Store script output as files in a directory filepath based on:
- script being run
- input arguments

With the metadata about the request to run the script in a file with the .meta extension (just this file present means the script result is pending), the resultant json data in a .json extension file and any errors encountered returned in a .error extension file.

e.g. for script at `company/financial_data/getFinancialData` which has input `{ company_number: 01234567}` we'd store the metadata at `output_cache/company/financial_data/getFinancialData/company_number_01234567.meta`, creating this when the script is called. When the data is produced it is saved at  `output_cache/company/financial_data/getFinancialData/company_number_01234567.json`, if an error is encountered the error is saved at `output_cache/company/financial_data/getFinancialData/company_number_01234567.error`.

The code that handles the storing of the script output only needs to know the parent directory of the cache and can calculate the filepath to store the output at from just the script path and inputs.

## Justification of technical approach

Directory structure of `cache_dir/<scripts_path>/<Input_params>.ext` was chosen because:

• Cache management is easier - When a script breaks or needs updates, you can invalidate/fix all outputs for that data type in one place, rather than hunting through hundreds of company folders

• Development workflow is better - When debugging the financial data script, you want to see all financial outputs together to spot patterns and errors, not dig through individual company directories

• File system performance - Fewer directories with more files performs better than thousands of sparse company directories with just a few files each

• API structure alignment - Your cache structure mirrors your API endpoints (/api/company/financial-data → /company/financial_data/), making the codebase intuitive

• Cross-company analysis is trivial - Operations like "find all companies missing financial data" or "update all confirmation statement formats" become simple directory scans instead of complex traversals

• Ready for future agent framework - Each agent can own its cache directory cleanly, rather than needing to coordinate access to shared company directories

## Assumptions / Risks / Open Questions

- Assume we don't have to query, searching over multiple companies, or where we do need to we could do this using an existing third-party API, as searching over files in directories is slow
- Assume if we want to query over multiple companies properties we can project the file system onto a database
- Assume that we can achieve easy synchronising of the filesystem using yjs through the volt
- Assume we will change the approach or develop a better solution if we need to scale to 100s or 1000s of users as we will hit performance issues.
- Assume that we will be replacing forked bash processing with agents running in agent framework in the near future (few weeks or so)
- Dependant on agent framework easily handing writing to files at shared file location
