# SCAPLab Website

Source for the SCalable AI Platform Laboratory (SCAPLab) website, live at **https://scaplab.github.io/**.

![Builder](https://github.com/SCAPLab/scaplab.github.io/actions/workflows/builder.yml/badge.svg)

Almost all of the site's content lives in a **Google Sheets document**. Edit the sheet, trigger a rebuild, and the site updates in a couple of minutes. You only need to touch this repo to change the layout or add static files.

## How it works

```
Google Sheets (text content) ──┐
Google Drive (member/gallery photos) ──┼─▶ GitHub Actions: builder/ (Python) ─▶ docs/ ─▶ GitHub Pages
assets/ in this repo (backgrounds, research images) ──┘
```

The [Builder workflow](.github/workflows/builder.yml) runs the Python builder in [builder](builder). The builder reads the spreadsheet through the Google Sheets API, downloads Drive photos, and renders the site into a `docs/` folder **on the runner**. That folder is deployed straight to GitHub Pages; **nothing is committed back to the repository**.

The workflow runs:

- on every push to `master`,
- when you click **Run workflow** on the [Actions page](https://github.com/SCAPLab/scaplab.github.io/actions/workflows/builder.yml), or
- when you click **🚀 Update Site → Update site now** inside the spreadsheet (see [below](#one-click-update-button-in-the-spreadsheet)).

There is **no scheduled rebuild**. After you edit the sheet, trigger one of the above.

## One-time setup

### 1. GitHub repository settings

1. **Settings → Pages → Build and deployment → Source → GitHub Actions.** Do *not* use "Deploy from a branch": with that setting, GitHub renders this README as the homepage instead of the built site.
1. **Settings → Secrets and variables → Actions → New repository secret.** Add two secrets:

    | Secret | Value |
    | --- | --- |
    | `API_KEY` | Google API key (step 2) |
    | `DATA_URL` | URL of the content spreadsheet (step 3) |

### 2. Google API key

1. Create an API key in the [Google Cloud Console](https://console.cloud.google.com/apis/credentials).
1. Enable **both** the **Google Sheets API** and the **Google Drive API** for that project. Drive is needed for member and gallery photos.
1. Set **Application restrictions → None**. The builder calls the API from a server with no referrer, so a referrer-restricted key fails with `403`. You can still restrict *which APIs* the key may call (Sheets + Drive).

### 3. Content spreadsheet

1. Make a copy of the reference spreadsheet (**File → Make a copy**). This keeps every tab the builder expects, including a hidden `Members` tab.
1. Share it as **Anyone with the link → Viewer**.
1. Put its URL in the `DATA_URL` secret.
1. **Clear out data inherited from other labs** before the first build:
    - **Redirects**: every row becomes a short URL on this site (`scaplab.github.io/<path>`).
    - **Personal**: every row is another spreadsheet that gets fetched and published at `scaplab.github.io/<path>`.
    - **Links**, **Announcements**, **Publications**, **Research**, **Gallery**, **Members - …** tabs.
    - **Website** tab: in particular `root_folder`, `repo_url`, `data_url`, `map_iframe_src`.

> **Do not rename or delete these tabs:** `Website`, `Announcements`, `Members`, `Tags`, `Links`, `Pages`, `Redirects`, `Personal`. They are read in one request, and the build fails if any one of them is missing. They may be empty (keep the header row).

### 4. Google Drive photo folder

Create one Drive folder, share it as **Anyone with the link → Viewer**, and put its share link in the Website tab's `root_folder` row:

```
root/
  member/
    pi/<photo>.jpg
    student/<photo>.jpg
  gallery/
    album1/   (photos for this album)
    album2/
```

## Spreadsheet reference

Every tab uses row 1 as a header; data starts at row 2.

| Tab | Shows up as | Columns |
| --- | --- | --- |
| **Website** | Site-wide settings (see table below) | Description, **ID**, **Value** |
| **Menu** | Top navigation bar | Menu, Submenu, URL |
| **Announcements** | Home page announcements | Title, Content (Markdown), Expiration datetime (hidden after this; blank = never) |
| **Members - &lt;slug&gt;\|&lt;Title&gt;** | `/members/<slug>` | Group, Name, Email, Image, Description, Links, Degree, Year |
| **Members** | `/members` (combined page; may be left empty) | same as above |
| **Research** | `/research` | Title, Image, Content (Markdown) |
| **Publications** | `/publications`, plus "Latest Publications" on the home page | Category, Title, Authors, Booktitle, Links, Tags |
| **Tags** | Topic filter on Publications | ID, Title, Tag, Color. Row 2 column B is a switch: `off` hides topics entirely |
| **Gallery** | `/gallery` | Title, Folder (path under `root_folder`, e.g. `gallery/album1`), Description |
| **Links** | `/links` | Category, Title, Full title, URL, Search query (`{{year}}` placeholder), CFP month, Event month |
| **Pages** | Custom Markdown pages at `/<path>` | Path, Title, Content (Markdown) |
| **Redirects** | Short URLs: `/<path>` redirects to the target | Path, Target URL |
| **Personal** | Personal pages built from another spreadsheet | Path, Spreadsheet URL |

### Website tab keys

| ID | Used for |
| --- | --- |
| `title` | Lab name: home page hero, footer, browser tab |
| `instruction` | Institution name. The hero reads "`title` @ `instruction`" |
| `subtitle` | Line under the hero title |
| `logo_text` | Text logo at the top left |
| `address` | Footer and Contact page |
| `about` | *(optional)* "About" section on the home page (Markdown), shown **instead of** Latest Publications. Leave it empty to show Latest Publications. Use a blank line (Alt+Enter twice) between paragraphs |
| `map_iframe_src` | Google Maps **embed** URL on the Contact page (Maps → Share → Embed a map → the `src` value) |
| `root_folder` | Drive folder share link for member and gallery photos |
| `repo_url` | "Repository for this Website" link in the footer |
| `index_background`, `members_background`, `research_background`, `links_background`, `contact_background`, `page_background`, `gallery_background` | Header background image for each page (URL, or a repo path like `/assets/images/backgrounds/main.jpg`) |
| `analytics_id` | *(optional)* Google Analytics measurement ID, e.g. `G-XXXXXXXXXX`. Leave the row out and no analytics script is loaded |
| `favicon` | *(optional)* Browser-tab icon URL or path. Defaults to `/assets/images/logo.svg` |

Text values (`title`, `logo_text`, `subtitle`, `instruction`, `address`, `about`) support Markdown and the formatting shortcuts below.

### Navigation menu

Each row of the `Menu` tab is one entry. Leave **Submenu** empty for a plain link; fill it in to turn that **Menu** label into a dropdown (all rows sharing the same Menu value become its items).

| Menu | Submenu | URL |
| --- | --- | --- |
| Home | | / |
| Members | PI | /members/pi |
| Members | Students | /members/students |
| Research | | /research |
| Publications | | /publications |
| Gallery | | /gallery |
| Contact | | /contact |

If there is no `Menu` tab, a default menu (Home, Members, Research, Links, Contact) is used.

### Member pages

- A tab named **`Members - <Name>`** is published at **`/members/<name>`** (lower-cased, spaces become hyphens).
- To use a short URL with a longer page title, write **`Members - <slug>|<Title>`**, e.g. `Members - pi|Principal Investigator` gives `/members/pi` with the title "Principal Investigator".
- Column A is the group heading, so one page can hold several groups (e.g. *Ph.D. Student*, *M.S. Student*).
- **Image**: a path under the Drive root starting with `member/`, e.g. `/member/pi/photo.jpg`. A full URL or a repo path (`/assets/...`) also works and is used as-is.

### Research images (stored in this repo, not Drive)

Research topic images are read from **[assets/images/researches](assets/images/researches)** in this repo. Commit the image there, then put just the file name (e.g. `mytopic.jpg`) in the Research tab's Image column. A full URL or an `/assets/...` path also works. Images display at 16:10; a sample is at `/assets/images/research-example.svg`.

### Publications formatting shortcuts

The `Authors` and `Booktitle` cells (and the Website text keys) accept Markdown plus:

| Shortcut | Result |
| --- | --- |
| `[red]text[/red]` | red text |
| `[blue]text[/blue]` | blue text |
| `[b]text[/b]` or `**text**` | **bold** |
| `[i]text[/i]` or `*text*` | *italic* |
| `[u]text[/u]` | underline |

The home page's "Latest Publications" shows the first 7 rows of the **first Category** in the Publications tab (unless the Website tab's `about` is set, which replaces it).

### Line breaks

- Inside a cell, press **Alt+Enter** (Gallery and Research descriptions also accept a typed `\n`).
- A blank line starts a new paragraph.

## One-click update button in the spreadsheet

The script in [google-apps-script/update-site-button.gs](google-apps-script/update-site-button.gs) adds a **🚀 Update Site** menu that starts the Builder workflow.

1. **Create a GitHub token** at [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new):
    - *Resource owner*: `SCAPLab`
    - *Repository access*: **Only select repositories** → `scaplab.github.io`
    - *Permissions*: **Actions: Read and write** (nothing else)
1. In the spreadsheet, open **Extensions → Apps Script**, replace any existing code with the contents of `update-site-button.gs`, and save.
1. In **Project Settings → Script Properties**, add:

    | Property | Value |
    | --- | --- |
    | `GITHUB_TOKEN` | the token from step 1 |
    | `GITHUB_OWNER` | `SCAPLab` |
    | `GITHUB_REPO` | `scaplab.github.io` |

1. Reload the spreadsheet and click **🚀 Update Site → Update site now**. The first click asks you to authorize the script.

Progress is visible on the repo's [Actions tab](https://github.com/SCAPLab/scaplab.github.io/actions).

## Static files

Anything in [assets](assets) is copied to `/assets/...` on the built site. Use it for backgrounds (`assets/images/backgrounds/`), research images, the favicon, and so on.

## Building locally

```bash
pip3 install -r requirements.txt
INPUT_API_KEY=<key> INPUT_DATA_URL=<sheet-url> python3 build.py
```

The site is written to `docs/`. Serve it with `python3 -m http.server -d docs` and open http://localhost:8000.

## Troubleshooting

| Symptom | Likely cause |
| --- | --- |
| The homepage shows this README | Pages source is "Deploy from a branch". Switch it to **GitHub Actions** |
| Build fails with `API_KEY is empty` / `DATA_URL is empty` | The repository secret is missing |
| Build fails with HTTP `403` | API key has a referrer restriction, or the Sheets/Drive API is not enabled |
| Build fails with HTTP `400` | One of the required tabs is missing or renamed |
| A photo is missing; the log says `not found in Drive root folder` | Path typo, or the Drive folder isn't shared publicly |

## Acknowledgements

This site is based on an open-source research group website template built by [Jeongmin Byun](https://jmbyun.github.io/), with support from [JinYeong Bak](https://nosyu.github.io/). It also builds on [DiaSys Lab](https://github.com/DiaSysLab/diasyslab.github.io)'s extensions: Drive-backed galleries and member photos, per-group member pages, and formatting shortcuts. Many thanks to all of them.
