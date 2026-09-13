# GitHub website with AI auto-generation

**Use Etude_GitHub_AI.zip.** It includes the complete built website, planner,
PDF import/OCR, science materials, recordings, and an AI backend.

The previous automatic mode used PDF excerpts. This edition also includes
real AI generation using the OpenAI API: explanations, definitions, summaries,
10/20/30/50-question quizzes, answer explanations, flashcards, and text for
audio review. The existing excerpt generator remains available as a local option.

AI setup has two parts: GitHub Pages hosts the app; a Cloudflare Worker calls
OpenAI while keeping the API key private. Uploading files alone does not activate
the AI service. You need an OpenAI API account with credit and a Cloudflare account.

## 1. What to put on GitHub

Extract **Etude_GitHub_AI.zip**. Upload **everything inside** the extracted
folder to the top level of a new GitHub repository. Keep all folders intact.

| File or folder | What it contains |
| --- | --- |
| `index.html` | The app's starting page |
| `assets/` | Working app interface, quiz and automatic generators |
| `science/` | Science PDFs, pictures and recordings |
| `planner/` | To-do list and calendar |
| `vendor/` | PDF reader, text recognition, audio engine and licenses |
| `ai-backend/` | The AI server code and optional deployment configuration |
| `ai-config.json` | Optional public AI server address; never put secrets here |
| `README.md` and `AI_SETUP.md` | These setup instructions |

Use **Add file → Upload files**, drag the extracted contents into GitHub, then
commit them to `main`. Keep **index.html** at the repository's top level.
This package fits GitHub's browser upload count and individual-file size limits.

In **Settings → Pages**, choose **Deploy from a branch**, **main**, **/(root)**,
then **Save**. Open the published website link shown there after deployment.
For GitHub Free, use a public repository. The app and bundled class materials
will be public; PDF files you later import are not uploaded to the repository.
See [GitHub Pages setup](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

## 2. Create the AI backend

1. Sign in to Cloudflare and open **Workers & Pages → Create application**.
2. Choose **Import a repository**, connect GitHub, and select the repository
   where you uploaded this download.
3. Set the Worker project name to **etude-study-ai**. In build settings, use
   **ai-backend** as the **Root directory**, leave **Build command** blank, and
   use **npx wrangler@4.92.0 deploy** as the **Deploy command**. Deploy it. The
   ai-backend folder already contains worker.js and wrangler.jsonc.
4. Open that Worker's **Settings → Variables and Secrets**. Add the following
   as runtime settings, not the Build variables used during deployment:

| Name | Type | What you enter |
| --- | --- | --- |
| `OPENAI_API_KEY` | Secret | Your actual OpenAI API key |
| `APP_ACCESS_CODE` | Secret | A unique code you choose, at least 16 characters long |
| `ALLOWED_ORIGIN` | Text | `https://YOUR-USERNAME.github.io` with your real GitHub username |
| `OPENAI_MODEL` | Text, optional | `gpt-5-mini` |

5. Save/deploy the settings. Copy the Worker's HTTPS address, for example
   `https://etude-study-ai.YOUR-ACCOUNT.workers.dev`.

For `ALLOWED_ORIGIN`, use only the website origin: **omit the repository path
and trailing slash**. If using a custom domain, use its HTTPS origin instead.
Multiple trusted website origins can be separated by commas. Do not use `*`.

Your API key stays in the Worker's secret settings. Do not put it in GitHub,
`ai-config.json`, the app's access-code box, or a URL. The separate access code
protects your AI endpoint and must also stay out of the repository.

[Cloudflare Worker creation](https://developers.cloudflare.com/workers/get-started/dashboard/)
and [Worker secret settings](https://developers.cloudflare.com/workers/configuration/secrets/).
The included configuration preserves these dashboard variables on later deploys.

## 3. Connect it in your app

1. Open your **GitHub Pages website** and click **Add PDF**.
2. Under **Generate with**, select **AI — explanations, definitions & quiz**.
3. Enter your Worker's address in **AI server address**.
4. Enter the code you chose for `APP_ACCESS_CODE` in **AI access code**.
5. Click **Check AI connection**.
6. Choose your PDF, study language and quiz length, then **Generate with AI**.

The server address is remembered in this browser. The access code is kept only
in the open tab and must be entered again after a reload. The connection check
verifies that your Worker is configured and accepts the code; the first
generation also tests your actual OpenAI key, credit and model access.

Optional: make the AI server address appear automatically for visitors by
editing **ai-config.json** on GitHub to contain:

```json
{"serverUrl":"https://etude-study-ai.YOUR-ACCOUNT.workers.dev"}
```

Replace the example address with your actual Worker address. Put only the public
server address in that file. Visitors still need the separate access code.

## What works and what is saved

The sidebar switches between **Study room** and **To-do list**; on phones these
switches are above the content. The science quiz, study sheets, flashcards,
source PDFs, recordings, planner, and excerpt generator work without AI setup.

AI mode sends extracted document text to your Worker and OpenAI. It generates
study content from that text; it cannot inspect diagrams that OCR did not
describe. Original PDF files, study packs and progress are saved locally in
the browser. They do not sync between devices. Clearing site data removes
local saves. The server does not store packs, PDFs or access codes in a database.

AI imports support at most five PDFs, 150 total pages and 120,000 extracted
characters. Larger documents are rejected with a clear message rather than
silently shortened. Short source material can produce fewer questions. Always
check generated definitions, answers and references against the original PDFs.

AI use can incur API charges. Usage is billed to the configured OpenAI API
account. The app does not include an API key or prepaid credit. Use your
provider account's usage controls; the app's access code is intended for your
own use or trusted users, not an unrestricted public AI service.

## If something does not work

- **Only words/code:** open the website link in **Settings → Pages**, not the
  repository's Code page. Refresh with Ctrl+Shift+R after an update.
- **Could not connect / website not allowed:** check the Worker address,
  deployment and `ALLOWED_ORIGIN`. For GitHub Pages this is the username's
  `github.io` origin, without `/repository-name/`.
- **Enter your AI access code:** use `APP_ACCESS_CODE`, not `OPENAI_API_KEY`.
- **Finish AI setup:** all three required Worker settings must exist and the
  access code must be at least 16 characters long.
- **Provider usage/credit/rate limit:** check the OpenAI API account's available
  credit and limits, then retry. GitHub Pages cannot resolve an API billing issue.
- **Response cut short:** choose fewer questions or split the PDF into chapters.
- **Original PDFs or recordings missing:** upload all the downloaded folders.

## Editable source and checks

**Etude_GitHub_AI_Source.zip** contains the complete editable source and all
original assets. With Node.js 22.13 or newer:

```sh
npm run install:ci
npm run build:pages
node --experimental-strip-types scripts/check-pages.mjs
node --experimental-strip-types scripts/check-ai.mjs
node scripts/package-github-ai.mjs
```

The last command creates **github-ai-export**. Upload everything inside that
folder to rebuild the ready-to-upload edition after editing the source.

The full source uses **ai-backend/entry.mjs** and **worker.mjs**. The ready-to-upload
website ZIP includes a bundled **worker.js** so its Git deployment runs without building the frontend again. The original ChatGPT-hosted server edition remains
in the source project and has not been republished by this GitHub export.

The app build, browser-storage checks and AI client/backend integration checks
passed here. AI provider responses were simulated for testing; no paid OpenAI
request was made. Your actual AI account connection must be completed and tested
with your own key after setup.

Technical references: [OpenAI structured outputs](https://developers.openai.com/api/docs/guides/structured-outputs),
[GPT-5 Mini](https://developers.openai.com/api/docs/models/gpt-5-mini),
[OpenAI production guidance](https://developers.openai.com/api/docs/guides/production-best-practices).
