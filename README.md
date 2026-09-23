<div align="center">

<img src="icons/icon128.png" alt="" width="96" height="96">

# TTD Smart Fill

**Save pilgrim details once. Fill TTD booking forms in one click.**

A Chrome extension that stores pilgrim profiles on your own device and fills them into the pilgrim-details form on the Tirumala Tirupati Devasthanams booking portal. You still review the form, enter the CAPTCHA/OTP and submit it yourself.

![Manifest V3](https://img.shields.io/badge/Manifest-V3-663399)
![Chrome 105+](https://img.shields.io/badge/Chrome-105%2B-663399)
![No dependencies](https://img.shields.io/badge/dependencies-none-7c067e)
![No network requests](https://img.shields.io/badge/network%20requests-none-7c067e)

</div>

> [!IMPORTANT]
> This is an independent project. It is **not affiliated with or endorsed by Tirumala Tirupati Devasthanams (TTD)**. It only types into form fields for you. It does not book tickets, guarantee availability, skip queues, or bypass CAPTCHA, OTP or payment.

<p align="center">
  <img src="docs/popup.png" alt="Popup with three saved profiles and a Fill 4 pilgrims button" width="300">
  &nbsp;&nbsp;
  <img src="docs/options.png" alt="Profile editor showing pilgrim name, age, gender, photo ID and contact fields" width="520">
</p>

## Features

- **Profiles for every group.** Family, parents, friends: up to 6 pilgrims each, with name, age, gender, photo-ID type and number.
- **One-click fill.** Works with text boxes, dropdowns, Angular Material selects, autocomplete fields and radio buttons.
- **Fills fields in the right order.** It picks the ID type before the ID number and the State before the City, and waits for fields that load or unlock late.
- **Clear feedback.** Filled fields are outlined, and a card on the page lists anything you still need to fill by hand. It also warns you if the form has fewer pilgrim rows than your profile.
- **Checks your data before saving.** Aadhaar, PAN, passport, voter ID, mobile and PIN code formats are validated as you type. Pasted numbers like `+91 98765 43210` are cleaned up.
- **Backup.** Export all profiles to a JSON file, and import them on another computer (merge or replace).
- **Matches the TTD portal's look**, with its purple palette, rounded cards and Open Sans font.

<p align="center">
  <img src="docs/fill-result.png" alt="A test form after filling: fields outlined in purple and a Filled 12 fields card" width="640">
  <br><sub>Result card after filling the included test form.</sub>
</p>

## What it will never do

| | |
|---|---|
| 🚫 Submit forms | It never clicks Submit, Pay, Proceed, Confirm or Verify. |
| 🚫 Touch CAPTCHA or OTP | Fields named or labelled CAPTCHA, OTP or password are skipped. |
| 🚫 Send your data anywhere | There are no servers and no analytics. The extension makes no network requests. |
| 🚫 Run on other sites | It only has access to `ttdevasthanams.ap.gov.in`. |

## Install

The extension isn't on the Chrome Web Store yet. To install it from source:

1. Download this repository (**Code → Download ZIP**, then unzip it) or clone it:
   ```bash
   git clone https://github.com/<your-username>/ttd-smart-fill.git
   ```
2. Open `chrome://extensions` and turn on **Developer mode** (top right).
3. Click **Load unpacked** and select the `ttd-smart-fill` folder.
4. Pin the extension from the puzzle-piece menu so the icon is always visible.

## How to use

1. Click the extension icon, then **Add pilgrim profile** (or the ⚙ button). Enter your pilgrims and click **Save profile**.
2. Sign in on [ttdevasthanams.ap.gov.in](https://ttdevasthanams.ap.gov.in/), pick your darshan/seva and the number of persons, and go on to the **pilgrim-details** step.
3. Click the extension icon, choose a profile and click **Fill N pilgrims**.
4. Check every field, enter the CAPTCHA/OTP, and submit the booking yourself.

> [!TIP]
> If the popup says **"Couldn't match: …"**, fill those fields by hand. Then see [Tuning for the live site](#tuning-for-the-live-site) to fix the matching permanently.

## Privacy

Your pilgrim details stay in `chrome.storage.local` on your computer. They are never uploaded, synced or shared. Removing the extension deletes them. The full policy is in [PRIVACY.md](PRIVACY.md).

| Permission | Why |
|---|---|
| `storage` | Save your profiles on this device |
| `activeTab`, `scripting` | Fill the TTD tab you're on, only when you click Fill |
| `https://ttdevasthanams.ap.gov.in/*` | The only site the extension runs on |

## Tuning for the live site

The fill engine finds each field by reading its `name`, `formcontrolname`, `id`, `placeholder` and `aria-label`. It splits these into words (`idProofNumber` becomes "id proof number") and falls back to the visible label text. The rules live in [`content/fieldMap.js`](content/fieldMap.js).

If TTD changes its form and a field stops matching:

1. On the pilgrim-details page, open the popup and click **Scan fields**. This copies a JSON list of every field the extension can see.
2. Add an exact CSS selector for the missing field to its `selectors` array in `content/fieldMap.js`:
   ```js
   idNumber: { selectors: ['input[formcontrolname="proofNumber"]'], /* ... */ },
   ```
3. Click reload on the extension card in `chrome://extensions`, then refresh the TTD tab.

Pull requests with updated selectors are welcome.

## Development

Everything is plain HTML, CSS and JavaScript. There is no build step and nothing to install.

```
ttd-smart-fill/
├── manifest.json          # MV3 manifest (TTD host only)
├── lib/storage.js         # chrome.storage.local wrapper: profiles, import/export
├── content/
│   ├── fieldMap.js        # field-matching rules, value aliases, forbidden fields
│   └── content.js         # fill engine and on-page result card
├── popup/                 # profile picker and Fill button
├── options/               # profile editor, validation, backup
├── icons/
├── docs/                  # README screenshots, publishing checklist
└── test/                  # mock forms and a UI preview harness (not shipped)
```

### Running the tests

Serve the folder with any static server:

```bash
python -m http.server 8765
```

| Page | What it checks |
|---|---|
| `localhost:8765/test/mock-ttd.html` | Basic fill: text inputs, a native select, a Material-style dropdown. The CAPTCHA must stay empty and the form must not be submitted. |
| `localhost:8765/test/mock-edge.html` | Hard cases: "Female" listed before "Male", an ID number that unlocks late, a City list that loads after State, a field matched by its second attribute, an OTP box, and more pilgrims than form rows. |
| `localhost:8765/test/harness.html?page=popup` | The popup with sample data, without installing the extension. |
| `localhost:8765/test/harness.html?page=options` | The profile editor with sample data. |

On the mock pages, click **Run fill** and read the log. The harness takes extra options: `&ttd=0` (not on the TTD site), `&seed=0` (empty state), `&reset=1` (reload sample profiles), `&bare=1` (no frame) and `&open=1` (open the first profile).

### Updating the screenshots

With the test server running:

```powershell
$chrome = "C:\Program Files\Google\Chrome\Application\chrome.exe"
& $chrome --headless=new --hide-scrollbars --force-device-scale-factor=2 --window-size=360,600 --virtual-time-budget=6000 --screenshot=docs/popup.png "http://localhost:8765/test/harness.html?page=popup&bare=1&reset=1"
& $chrome --headless=new --hide-scrollbars --force-device-scale-factor=2 --window-size=1280,800 --virtual-time-budget=6000 --screenshot=docs/options.png "http://localhost:8765/test/harness.html?page=options&open=1&reset=1"
& $chrome --headless=new --hide-scrollbars --force-device-scale-factor=2 --window-size=900,440 --virtual-time-budget=6000 --screenshot=docs/fill-result.png "http://localhost:8765/test/mock-ttd.html?autorun=1"
```

### Packaging a release

```powershell
Compress-Archive -Path manifest.json,lib,content,popup,options,icons -DestinationPath ttd-smart-fill.zip -Force
```

Bump `version` in `manifest.json` first. The Chrome Web Store listing checklist is in [docs/PUBLISHING.md](docs/PUBLISHING.md).

## Known limitations

- The field rules were built against mock forms. The live pilgrim-details page needs a TTD login, so exact selectors still need to be captured from it (see [Tuning](#tuning-for-the-live-site)).
- Only the top-level page is filled, not forms inside iframes.
- Up to 6 pilgrims per profile.

## Contributing

Issues and pull requests are welcome, especially updated field selectors when the TTD site changes. Please run both mock pages before opening a PR, and keep the rules above: no auto-submit, no CAPTCHA/OTP handling, no network requests.

## License

No license has been chosen yet. Until a `LICENSE` file is added, all rights are reserved by the author.
